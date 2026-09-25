---
title: deepseek建议与实践
date: 2026-09-20 12:26:20
tags:
---
1.一个当前会话内容查询功能（好像最近已经被官方实现了）（等等deepseek官方那里有bug）
搜索算法
用的是精确子串 + Token 覆盖的混合策略，适合"用户查找"场景：精确匹配得 100 分，Token 全命中再加 30 分。这意味着你搜"之前那个报错"和搜"报错"都能命中同一条消息，但前者如果恰好是原文子串，会排在最前面。不需要向量检索——关键词匹配在这个场景下足够快、足够准。

中文分词
用 bigram（双字组合）做中文分词。比如"搜索插件"会被拆成 搜、索、插、件、搜索、索插、插件。这样即使用户输入的关键词和原文不完全一致，也能通过 bigram 重叠匹配到。代码里用的是正则 [\u4e00-\u9fff\u3400-\u4dbf] 提取 CJK 字符，覆盖了基本汉字和扩展 A 区。

虚拟滚动处理
提供了 loadAllMessages() 函数，但默认没有自动调用，因为它会强制滚动页面。如果你觉得搜索结果不全，可以在搜索前手动调用：

js
// 在控制台手动触发
loadAllMessages().then(() => console.log('加载完成，可以搜索了'));
更好的做法是在面板里加一个"加载全部消息"按钮，但考虑到首次使用体验，默认不触发滚动。

增量索引
MutationObserver 监听 document.body 的子节点变化，防抖 800ms 后调用 incrementalIndex()。这个函数只添加 ID 不在索引中的新消息，不会重建整个索引，性能开销很小。

会话切换
通过监听 location.href 变化来检测会话切换。DeepSeek 是 SPA，切换会话不会刷新页面，所以需要监听 URL 变化。检测到变化后等 1 秒重建索引。

(function () {
  'use strict';

  // ==================== 配置 ====================
  const CONFIG = {
    // 消息选择器 —— 优先用 data 属性，如果不匹配则回退到 class 策略
    MESSAGE_SELECTOR: '[data-message-author-role]',
    FALLBACK_SELECTOR: 'div[class*="message"]',
    // 搜索面板最大结果显示数
    MAX_RESULTS: 30,
    // 搜索防抖延迟（毫秒）
    DEBOUNCE_MS: 150,
  };

  // ==================== 状态 ====================
  const state = {
    messages: [],        // { id, role, text, el }
    index: new Map(),    // id -> message 对象
    panelOpen: false,
    currentQuery: '',
    resultElements: [],  // 搜索结果对应的 DOM 元素
    currentResultIndex: -1,
  };

  // ==================== 工具函数 ====================

  /** 简单字符串 hash，用于生成稳定 ID */
  function simpleHash(str) {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      const char = str.charCodeAt(i);
      hash = (hash << 5) - hash + char;
      hash |= 0;
    }
    return Math.abs(hash).toString(36);
  }

  /** 从 URL 提取会话 ID */
  function getSessionId() {
    const parts = location.pathname.split('/').filter(Boolean);
    return parts[parts.length - 1] || 'default';
  }

  /** 中文分词：bigram + 英文单词 */
  function tokenize(text) {
    const tokens = [];
    const lower = text.toLowerCase();

    // 英文单词和数字
    const wordMatches = lower.match(/[a-z0-9_]+/g);
    if (wordMatches) tokens.push(...wordMatches);

    // 中文字符 bigram
    const cjkRegex = /[\u4e00-\u9fff\u3400-\u4dbf]/g;
    const cjkChars = lower.match(cjkRegex);
    if (cjkChars) {
      tokens.push(...cjkChars);
      for (let i = 0; i < cjkChars.length - 1; i++) {
        tokens.push(cjkChars[i] + cjkChars[i + 1]);
      }
    }

    return tokens;
  }

  /** 检测消息角色 */
  function detectRole(el) {
    // 优先用 data 属性
    const role = el.dataset.messageAuthorRole;
    if (role === 'user' || role === 'assistant') return role;

    // 回退：根据 class 或文本特征判断
    const className = el.className || '';
    if (className.includes('user') || className.includes('User')) return 'user';
    if (className.includes('assistant') || className.includes('Assistant')) return 'assistant';

    // 最后回退：用户消息通常较短
    return el.innerText.length < 300 ? 'user' : 'assistant';
  }

  // ==================== 消息提取 ====================

  /** 提取页面中所有已加载的消息 */
  function extractMessages() {
    let elements = [...document.querySelectorAll(CONFIG.MESSAGE_SELECTOR)];

    // 如果主选择器没有匹配到，尝试回退
    if (elements.length === 0) {
      elements = [...document.querySelectorAll(CONFIG.FALLBACK_SELECTOR)]
        .filter(el => {
          const rect = el.getBoundingClientRect();
          return rect.width > 300 && el.innerText.trim().length > 5;
        });
    }

    // 去重：如果父元素和子元素都匹配，只保留最深的
    const deduped = elements.filter(el => {
      return !elements.some(other => other !== el && other.contains(el));
    });

    // 按 DOM 顺序排列
    deduped.sort((a, b) => {
      const pos = a.compareDocumentPosition(b);
      if (pos & Node.DOCUMENT_POSITION_FOLLOWING) return -1;
      if (pos & Node.DOCUMENT_POSITION_PRECEDING) return 1;
      return 0;
    });

    return deduped.map((el, i) => {
      const text = el.innerText.trim();
      const id = `msg-${getSessionId()}-${i}-${simpleHash(text.slice(0, 100))}`;
      return {
        id,
        role: detectRole(el),
        text,
        el,
      };
    });
  }

  /** 重建索引 */
  function rebuildIndex() {
    state.messages = extractMessages();
    state.index.clear();
    state.messages.forEach(msg => state.index.set(msg.id, msg));
    updateResultCount();
  }

  /** 增量索引：只添加新消息 */
  function incrementalIndex() {
    const current = extractMessages();
    let added = 0;

    current.forEach(msg => {
      if (!state.index.has(msg.id)) {
        state.messages.push(msg);
        state.index.set(msg.id, msg);
        added++;
      }
    });

    if (added > 0) {
      // 重新排序，保持 DOM 顺序
      state.messages.sort((a, b) => {
        const pos = a.el.compareDocumentPosition(b.el);
        if (pos & Node.DOCUMENT_POSITION_FOLLOWING) return -1;
        if (pos & Node.DOCUMENT_POSITION_PRECEDING) return 1;
        return 0;
      });
    }
  }

  // ==================== 搜索逻辑 ====================

  /** 执行搜索 */
  function search(query) {
    if (!query || query.trim().length < 1) {
      renderResults([]);
      return;
    }

    const queryTokens = tokenize(query);
    if (queryTokens.length === 0) {
      renderResults([]);
      return;
    }

    const queryLower = query.toLowerCase();

    // 对每条消息计算得分
    const scored = state.messages.map(msg => {
      const textLower = msg.text.toLowerCase();
      const textTokens = tokenize(msg.text);
      const tokenSet = new Set(textTokens);

      let score = 0;

      // 1. 精确子串匹配（最高权重）
      if (textLower.includes(queryLower)) {
        score += 100;
        // 如果在开头附近匹配，额外加分
        const idx = textLower.indexOf(queryLower);
        if (idx < 50) score += 20;
      }

      // 2. Token 匹配
      let matchedTokens = 0;
      for (const token of queryTokens) {
        if (tokenSet.has(token)) {
          matchedTokens++;
          score += 10;
        }
      }

      // 3. Token 覆盖率奖励
      if (queryTokens.length > 1) {
        const coverage = matchedTokens / queryTokens.length;
        if (coverage === 1) score += 30; // 全部命中
        else if (coverage >= 0.5) score += 10;
      }

      // 4. 角色过滤奖励（用户提问通常更需要被搜索到）
      if (msg.role === 'user' && score > 0) score += 5;

      return { msg, score };
    });

    // 过滤掉得分为 0 的，排序取 top
    const results = scored
      .filter(item => item.score > 0)
      .sort((a, b) => b.score - a.score)
      .slice(0, CONFIG.MAX_RESULTS)
      .map(item => item.msg);

    renderResults(results, query);
  }

  // ==================== UI 构建 ====================

  /** 创建搜索面板 DOM */
  function createPanel() {
    const panel = document.createElement('div');
    panel.id = 'ds-search-panel';
    panel.innerHTML = `
      <div class="ds-search-header">
        <input id="ds-search-input" type="text" placeholder="搜索当前会话..." autocomplete="off" />
        <span id="ds-search-count"></span>
        <button id="ds-search-close" title="关闭">✕</button>
      </div>
      <div id="ds-search-results"></div>
      <div class="ds-search-footer">
        <span>Enter 跳转到下一条 · Shift+Enter 上一条 · Esc 关闭</span>
      </div>
    `;

    // 注入样式
    const style = document.createElement('style');
    style.textContent = `
      #ds-search-panel {
        position: fixed;
        top: 60px;
        right: 20px;
        width: 420px;
        max-height: 70vh;
        background: var(--ds-bg-color, #1e1e2e);
        color: var(--ds-text-color, #cdd6f4);
        border: 1px solid rgba(128, 128, 128, 0.3);
        border-radius: 12px;
        box-shadow: 0 8px 32px rgba(0, 0, 0, 0.4);
        z-index: 99999;
        display: none;
        flex-direction: column;
        font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
        font-size: 14px;
        overflow: hidden;
        backdrop-filter: blur(12px);
      }
      #ds-search-panel.open { display: flex; }

      .ds-search-header {
        display: flex;
        align-items: center;
        padding: 10px 12px;
        border-bottom: 1px solid rgba(128, 128, 128, 0.2);
        gap: 8px;
      }
      #ds-search-input {
        flex: 1;
        background: rgba(128, 128, 128, 0.15);
        border: 1px solid rgba(128, 128, 128, 0.2);
        border-radius: 6px;
        padding: 8px 10px;
        color: inherit;
        font-size: 14px;
        outline: none;
      }
      #ds-search-input:focus {
        border-color: #7c3aed;
        box-shadow: 0 0 0 2px rgba(124, 58, 237, 0.2);
      }
      #ds-search-count {
        font-size: 12px;
        opacity: 0.6;
        white-space: nowrap;
      }
      #ds-search-close {
        background: none;
        border: none;
        color: inherit;
        cursor: pointer;
        font-size: 16px;
        opacity: 0.6;
        padding: 4px;
      }
      #ds-search-close:hover { opacity: 1; }

      #ds-search-results {
        overflow-y: auto;
        flex: 1;
        padding: 4px;
      }

      .ds-result-item {
        padding: 8px 10px;
        border-radius: 8px;
        cursor: pointer;
        margin-bottom: 2px;
        transition: background 0.15s;
      }
      .ds-result-item:hover,
      .ds-result-item.active {
        background: rgba(124, 58, 237, 0.15);
      }
      .ds-result-role {
        font-size: 11px;
        font-weight: 600;
        opacity: 0.5;
        margin-bottom: 3px;
        text-transform: uppercase;
        letter-spacing: 0.5px;
      }
      .ds-result-snippet {
        font-size: 13px;
        line-height: 1.5;
        opacity: 0.85;
        display: -webkit-box;
        -webkit-line-clamp: 3;
        -webkit-box-orient: vertical;
        overflow: hidden;
      }
      .ds-result-snippet mark {
        background: rgba(124, 58, 237, 0.4);
        color: inherit;
        border-radius: 2px;
        padding: 0 2px;
      }
      .ds-result-empty {
        padding: 20px;
        text-align: center;
        opacity: 0.5;
        font-size: 13px;
      }

      .ds-search-footer {
        padding: 6px 12px;
        border-top: 1px solid rgba(128, 128, 128, 0.2);
        font-size: 11px;
        opacity: 0.4;
        text-align: center;
      }

      /* 消息高亮动画 */
      .ds-search-hit {
        animation: ds-hit-flash 1.5s ease-out !important;
        outline: 2px solid #7c3aed !important;
        outline-offset: 2px;
        border-radius: 8px;
      }
      @keyframes ds-hit-flash {
        0% { outline-color: #7c3aed; outline-width: 4px; }
        100% { outline-color: transparent; outline-width: 2px; }
      }

      /* 悬浮按钮 */
      #ds-search-fab {
        position: fixed;
        bottom: 100px;
        right: 24px;
        width: 40px;
        height: 40px;
        border-radius: 50%;
        background: #7c3aed;
        color: #fff;
        border: none;
        cursor: pointer;
        font-size: 18px;
        z-index: 99998;
        display: flex;
        align-items: center;
        justify-content: center;
        box-shadow: 0 4px 12px rgba(124, 58, 237, 0.4);
        transition: transform 0.15s, box-shadow 0.15s;
      }
      #ds-search-fab:hover {
        transform: scale(1.1);
        box-shadow: 0 6px 16px rgba(124, 58, 237, 0.5);
      }
    `;

    document.head.appendChild(style);
    document.body.appendChild(panel);

    // 悬浮按钮
    const fab = document.createElement('button');
    fab.id = 'ds-search-fab';
    fab.title = '搜索会话 (Alt+K)';
    fab.textContent = '🔍';
    document.body.appendChild(fab);

    // ==================== 事件绑定 ====================

    const input = panel.querySelector('#ds-search-input');
    const resultsEl = panel.querySelector('#ds-search-results');
    const closeBtn = panel.querySelector('#ds-search-close');

    // 输入防抖
    let debounceTimer;
    input.addEventListener('input', () => {
      clearTimeout(debounceTimer);
      debounceTimer = setTimeout(() => {
        state.currentQuery = input.value;
        search(input.value);
      }, CONFIG.DEBOUNCE_MS);
    });

    // 键盘导航
    input.addEventListener('keydown', (e) => {
      if (e.key === 'Enter') {
        e.preventDefault();
        if (e.shiftKey) navigateResult(-1);
        else navigateResult(1);
      } else if (e.key === 'Escape') {
        closePanel();
      }
    });

    // 关闭按钮
    closeBtn.addEventListener('click', closePanel);
    fab.addEventListener('click', togglePanel);

    // 全局快捷键 Alt+K
    document.addEventListener('keydown', (e) => {
      if (e.altKey && e.key === 'k') {
        e.preventDefault();
        togglePanel();
      }
      if (e.key === 'Escape' && state.panelOpen) {
        closePanel();
      }
    });

    // 点击面板外部关闭（可选，注释掉以免误关）
    // document.addEventListener('click', (e) => {
    //   if (state.panelOpen && !panel.contains(e.target) && e.target !== fab) {
    //     closePanel();
    //   }
    // });
  }

  /** 打开/关闭面板 */
  function togglePanel() {
    const panel = document.getElementById('ds-search-panel');
    if (!panel) return;

    if (state.panelOpen) {
      closePanel();
    } else {
      panel.classList.add('open');
      state.panelOpen = true;
      const input = document.getElementById('ds-search-input');
      input.value = '';
      input.focus();
      // 打开时重建索引，确保包含最新消息
      rebuildIndex();
      renderResults([]);
    }
  }

  function closePanel() {
    const panel = document.getElementById('ds-search-panel');
    if (panel) panel.classList.remove('open');
    state.panelOpen = false;
    state.resultElements = [];
    state.currentResultIndex = -1;
  }

  /** 渲染搜索结果 */
  function renderResults(results, query) {
    const resultsEl = document.getElementById('ds-search-results');
    if (!resultsEl) return;

    state.resultElements = results.map(r => r.el);
    state.currentResultIndex = -1;

    if (results.length === 0) {
      resultsEl.innerHTML = query
        ? '<div class="ds-result-empty">没有找到匹配的消息</div>'
        : '<div class="ds-result-empty">输入关键词开始搜索</div>';
      updateResultCount();
      return;
    }

    resultsEl.innerHTML = results.map((msg, i) => {
      // 生成高亮片段
      let snippet = msg.text.slice(0, 200);
      if (query) {
        const escaped = query.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
        snippet = snippet.replace(
          new RegExp(`(${escaped})`, 'gi'),
          '<mark>$1</mark>'
        );
      }
      const roleLabel = msg.role === 'user' ? '你' : 'AI';

      return `
        <div class="ds-result-item" data-index="${i}">
          <div class="ds-result-role">${roleLabel}</div>
          <div class="ds-result-snippet">${snippet}</div>
        </div>
      `;
    }).join('');

    // 绑定点击事件
    resultsEl.querySelectorAll('.ds-result-item').forEach(item => {
      item.addEventListener('click', () => {
        const idx = parseInt(item.dataset.index, 10);
        jumpToResult(idx);
      });
    });

    updateResultCount();
  }

  /** 更新结果计数显示 */
  function updateResultCount() {
    const countEl = document.getElementById('ds-search-count');
    if (!countEl) return;
    const total = state.resultElements.length;
    if (state.currentResultIndex >= 0 && total > 0) {
      countEl.textContent = `${state.currentResultIndex + 1}/${total}`;
    } else {
      countEl.textContent = total > 0 ? `${total} 条` : '';
    }
  }

  /** 导航到上一条/下一条结果 */
  function navigateResult(direction) {
    const total = state.resultElements.length;
    if (total === 0) return;

    state.currentResultIndex += direction;
    if (state.currentResultIndex < 0) state.currentResultIndex = total - 1;
    if (state.currentResultIndex >= total) state.currentResultIndex = 0;

    // 更新 UI 选中态
    document.querySelectorAll('.ds-result-item').forEach((item, i) => {
      item.classList.toggle('active', i === state.currentResultIndex);
    });

    // 滚动到对应消息
    const el = state.resultElements[state.currentResultIndex];
    if (el) {
      el.scrollIntoView({ behavior: 'smooth', block: 'center' });
      el.classList.add('ds-search-hit');
      setTimeout(() => el.classList.remove('ds-search-hit'), 1600);
    }

    updateResultCount();
  }

  /** 跳转到指定索引的结果 */
  function jumpToResult(index) {
    state.currentResultIndex = index;

    document.querySelectorAll('.ds-result-item').forEach((item, i) => {
      item.classList.toggle('active', i === index);
    });

    const el = state.resultElements[index];
    if (el) {
      el.scrollIntoView({ behavior: 'smooth', block: 'center' });
      el.classList.add('ds-search-hit');
      setTimeout(() => el.classList.remove('ds-search-hit'), 1600);
    }

    updateResultCount();
  }

  // ==================== 虚拟滚动处理 ====================

  /**
   * 尝试加载更早的历史消息。
   * DeepSeek 使用虚拟滚动，未加载到 DOM 的消息无法搜索。
   * 策略：滚动到顶部触发加载，等待后重建索引。
   */
  function loadAllMessages() {
    return new Promise((resolve) => {
      const scrollContainer = findScrollContainer();
      if (!scrollContainer) {
        resolve();
        return;
      }

      const originalScrollTop = scrollContainer.scrollTop;
      let attempts = 0;
      const maxAttempts = 10;

      function scrollUpAndWait() {
        scrollContainer.scrollTop = 0;
        attempts++;

        // 等待一会儿看 DOM 是否变化
        setTimeout(() => {
          const currentCount = extractMessages().length;
          const prevCount = state.messages.length;

          if (currentCount > prevCount && attempts < maxAttempts) {
            // 有新消息加载，继续
            rebuildIndex();
            scrollUpAndWait();
          } else {
            // 没有更多了，恢复滚动位置
            scrollContainer.scrollTop = originalScrollTop;
            rebuildIndex();
            resolve();
          }
        }, 500);
      }

      scrollUpAndWait();
    });
  }

  /** 找到滚动容器 */
  function findScrollContainer() {
    // 尝试常见的滚动容器选择器
    const candidates = [
      '[data-conversation-scroll]',
      '.chat-container',
      'main',
      '[class*="scroll"]',
    ];

    for (const sel of candidates) {
      const el = document.querySelector(sel);
      if (el && el.scrollHeight > el.clientHeight) return el;
    }

    // 回退：找 scrollHeight 最大的元素
    let best = null;
    let bestScrollHeight = 0;
    document.querySelectorAll('div').forEach(el => {
      if (el.scrollHeight > el.clientHeight && el.scrollHeight > bestScrollHeight) {
        best = el;
        bestScrollHeight = el.scrollHeight;
      }
    });
    return best;
  }

  // ==================== 初始化 ====================

  function init() {
    // 等待页面加载完成
    const checkReady = setInterval(() => {
      const msgs = document.querySelectorAll(CONFIG.MESSAGE_SELECTOR);
      if (msgs.length > 0 || document.querySelector('#chat-input')) {
        clearInterval(checkReady);

        // 创建 UI
        createPanel();

        // 首次索引
        rebuildIndex();

        // 监听 DOM 变化，增量索引
        const observer = new MutationObserver(() => {
          // 防抖，避免频繁重建
          clearTimeout(observer._timer);
          observer._timer = setTimeout(() => {
            incrementalIndex();
          }, 800);
        });
        observer.observe(document.body, { childList: true, subtree: true });

        // 监听 URL 变化（切换会话）
        let lastUrl = location.href;
        new MutationObserver(() => {
          if (location.href !== lastUrl) {
            lastUrl = location.href;
            setTimeout(() => {
              rebuildIndex();
              if (state.panelOpen) renderResults([]);
            }, 1000);
          }
        }).observe(document.body, { childList: true, subtree: true });

        console.log('[DeepSeek搜索] 已加载，按 Alt+K 打开搜索面板');
      }
    }, 1000);

    // 10 秒后放弃等待
    setTimeout(() => clearInterval(checkReady), 10000);
  }

  // 启动
  if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', init);
  } else {
    init();
  }
})();

2.一个壁纸类插件
