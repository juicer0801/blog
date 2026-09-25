---
title: 健脑房第一节————ai记忆的基础实现
date: 2026-09-15 21:20:52
tags:
---

ai的使用逐渐普及，与工业革命机器代替劳动而出现的健身房一般，ai代替思考的当下我们应该有一个健脑房
我的blog将会科普许多数字工作者当下该有的常识



本期将介绍ai记忆的保存问题（基础）

为什么你调用的ai记不住你说的话？


如下是ds的流式输出代码
答案其实很朴素：大模型 API 默认是无状态的。
你看到的所有“记忆”，几乎都不是模型自己记住了，而是代码在每一轮请求里，把历史消息重新塞回去。这个过程，像滚雪球。


在网页版聊天框里，AI 好像有连续记忆。
但在代码层面，一次 AI 调用更像一个函数：

text
response = model(messages)
你传进去什么，它就基于什么回答。
除非你再次把历史传进去，否则它不知道上一轮发生了什么。

所以，所谓“AI记忆”，在大多数 API 场景里其实是：

text
messages = messages + [用户新消息, AI回复]
下一次请求 = 把整个 messages 再发给模型


假设最开始只有一个系统提示：
messages = [
    {"role": "system", "content": "你是一名严谨的技术助理。"}
]
用户问：“什么是向量数据库？”


代码把用户消息追加进去：
messages.append({"role": "user", "content": "什么是向量数据库？"})
调用模型，得到回答。


然后代码又把回答追加进去：
messages.append({"role": "assistant", "content": answer})
下一轮，用户再问：“它和传统数据库有什么区别？”


此时发给模型的 messages 已经包含：

系统提示

第一轮用户问题

第一轮 AI 回答

第二轮用户问题

模型看到的不是“它”，而是完整的前情提要。
代码替模型记住了上下文。

一个最小实现大概是这样：
from openai import OpenAI

client = OpenAI()

messages = [
    {"role": "system", "content": "你是一名严谨的技术助理。"}
]

def chat(user_input):
    messages.append({"role": "user", "content": user_input})

    resp = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=messages,
    )

    answer = resp.choices[0].message.content
    messages.append({"role": "assistant", "content": answer})
    return answer

messages 数组就是 AI 的短期记忆。
每一次 append，都是在滚雪球。

三、雪球越滚越大，问题也会出现
滚雪球式记忆简单有效，但它不是无限的。

1. 上下文窗口有限
模型一次能看的 token 有上限。
雪球滚太大，早期内容会被截断，或者直接报错。

2. 成本和延迟上升
每轮都把历史重新发一遍。
历史越长，token 越多，费用越高，响应也可能越慢。

3. 噪声会越来越多
旧对话里可能有错误、闲聊、过期信息。
它们会继续影响新回答，像雪球里混进了泥沙。

4. 隐私风险
历史消息会反复上传。
如果里面有手机号、密钥、客户数据，风险会被放大。

如何完善？

四、工程化记忆

AI记忆不是模型能力，而是上下文工程。

常见做法有几类：

1. 截断：只保留最近几轮
python
MAX_TURNS = 10

def trim(messages):
    system = messages[0]
    recent = messages[-MAX_TURNS:]
    return [system] + recent


2. 摘要：把旧雪球压成一句话
python
def trim_with_summary(messages):
    if len(messages) <= 12:
        return messages

    system = messages[0]
    old = messages[1:-10]
    recent = messages[-10:]

    summary = summarize(old)  # 调模型或规则生成摘要

    return [
        system,
        {"role": "system", "content": f"历史摘要：{summary}"},
        *recent
    ]
这样既保留长期线索，又控制 token。

3. 检索：长期记忆放仓库，按需取用
把知识、文档、历史记录存进向量数据库。
每次只检索最相关的几段，塞进当前提示。

def build_messages(user_input):
    docs = vector_db.search(user_input, top_k=5)
    context = "\n".join(docs)

    return [
        {"role": "system", "content": "根据以下资料回答：\n" + context},
        {"role": "user", "content": user_input}
    ]
这相当于：
雪球不把所有东西都粘上，而是需要哪块，就去仓库取哪块。

4. 结构化记忆：状态存数据库，不靠自然语言
用户偏好、任务进度、订单状态，最好存成字段：

json
{
  "user_id": "u123",
  "preferred_language": "中文",
  "current_task": "写博客",
  "last_topic": "AI记忆"
}

我个人其实更认同一个结合体：
ai会记录并搭建个人的知识库，实现定制的检索手段

