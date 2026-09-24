---
title: c语言的常见错误一
date: 2026-09-19 12:19:20
tags:
---

易错点：
1.scanf 忘记 &

int age;
scanf("%d", age);   // 错：age 是值，不是地址
scanf("%d", &age);  // 对

但是
char name[20]; scanf("%s", name); 不用 &（字符数组名本身就是地址，所以不用 &）

2.
if (x = 5) { }        // 错：赋值，不是比较
if (5 == x) { }       // 对：常量放左边，写错编译器会报错

3
switch (n) {
    case 1: printf("1");   // 忘了 break，会继续执行 case 2
    case 2: printf("2");
}


4.数组存放的元素个数不能为零
  元素类型要相同  

5.
int i = -1;
unsigned int u = 1;
if (i < u) { }        // 结果可能是 false：i 被转成无符号大数


6.
#define SQUARE(x) x * x
SQUARE(1 + 2)         // 展开为 1 + 2 * 1 + 2 = 5，不是 9
#define SQUARE(x) ((x) * (x))   // 正确


7.局部数组生命周期
char *f() {
    char buf[10];
    return buf;       // 错：返回局部数组，函数结束就失效
}

8.字符串常量与字符数组
char *s = "hello";
s[0] = 'H';           // 错：字符串常量不可修改
char s[] = "hello";   // 对：数组可修改

9.sizeof对数组
int a[10];
void f(int a[]) {
    sizeof(a);        // 这里是指针大小，不是数组总大小
}

10.二维数组其实并不是直觉上的表格形式，而是连续储存的

原因：通过监听内存地址的窗口我们可以发现每个元素之间的字节差距都是4（这里以int数组为例）
直观展示：[1,2,3,4,5] [2,3,4,5,6,7]

而不是： [1,2,3,4,5]
        [2,3,4,5,6,7]


11.
if (fabs(a - b) < 1e-9) { }   // 浮点数不要直接用 == 比较


12.
char name[20];
scanf("%s", name);    // 对，字符数组名本身就是地址