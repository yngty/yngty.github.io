---
title: Qt 之 焦点处理
date: 2023-02-15 17:08:09
tags:
- 焦点
categories:
- Qt
---

# 焦点事件

当焦点从一个 `widget` 移动到另一个 `widget` 时，会触发 `QFocusEvent` 事件，这个事件会被发送给原焦点窗口和当前焦点窗口，原焦点窗口执行 `focusOutEvent()` ，新焦点窗口执行 `focusInEvent()`。 相关函数如下：

```c++
void focusInEvent(QFocusEvent *event) override;
void focusOutEvent(QFocusEvent *event) override;
```
# 焦点策略

只有**可获取焦点**的窗口，才有机会成为焦点窗口。比如`QWidget` 默认策略是 `Qt::NoFocus` 所以 QWidget 默认不获取焦点。Qt提供了如下接口：
```c++
void QWidget::setFocusPolicy(Qt::FocusPolicy policy);
```

## enum Qt::FocusPolicy

| Constant | Value | Description | 
| :---        |    :---  |          :--- |
| Qt::TabFocus |	0x1	| the widget accepts focus by tabbing.| 
| Qt::ClickFocus | 	0x2	| the widget accepts focus by clicking.| 
| Qt::StrongFocus | 	TabFocus &#124; ClickFocus &#124; 0x8	|  the widget accepts focus by both tabbing and clicking. On macOS this will also be indicate that the widget accepts tab focus when in 'Text/List focus mode'.| 
| Qt::WheelFocus | 	StrongFocus &#124; 0x4	| like Qt::StrongFocus plus the widget accepts focus by using the mouse wheel.| 
| Qt::NoFocus | 0	| the widget does not accept focus.| 

# 焦点原因

### Qt::FocusReason

| Constant | Value | Description | 
| :---        |    :---  |          :--- |
| Qt::MouseFocusReason| 	0	| A mouse action occurred.
| Qt::TabFocusReason| 	1	| The Tab key was pressed.
| Qt::BacktabFocusReason| 	2	| A Backtab occurred. The input for this may include the Shift or Control keys; e.g. Shift+Tab.
| Qt::ActiveWindowFocusReason| 	3	| The window system made this window either active or inactive.
| Qt::PopupFocusReason| 	4	| The application opened/closed a pop-up that grabbed/released the keyboard focus.| 
| Qt::ShortcutFocusReason| 	5	| The user typed a label's buddy shortcut| 
| Qt::MenuBarFocusReason| 	6	| The menu bar took focus.| 
| Qt::OtherFocusReason| 	7	| Another reason, usually application-specific.| 
# 焦点信号

`QApplication` 的信号。
```c++
void focusChanged(QWidget *old, QWidget *now);
```
# 焦点次序

相关接口如下： 
```c++
// 返回此部件焦点链中的下一个部件
QWidget* QWidget::nextInFocusChain() const; 
// 返回此部件焦点链中的前一个部件
QWidget* QWidget::previousInFocusChain() const;
// 将焦点顺序中的部件 second 放置在部件 first 之后
static void setTabOrder(QWidget *, QWidget *);
```

通过按 `Tab` 或者 `Shift+Tab`，可以实现焦点在各个窗口之间循环移动。

- 点击 `Tab` 键，焦点向后查找，直至找到第一个 `FocusPolicy` 大于等于 `TabFocus` 的窗口，并设置该窗口为焦点窗口;

- 点击 `Shift+Tab` ，焦点向前查找，直至找到第一个 `FocusPolicy`  大于等于 `TabFocus` 的窗口，并设置该窗口为焦点窗口;

默认情况下, 先加入的 `QWidget` 焦点顺序越靠前。可以通过 `setTabOrder` 调整顺序.

比如，若默认的焦点链顺序为 `a-b-c-d` ，则：
```
setTabOrder(d,c); //改变后焦点链的顺序为 a-b-d-c
setTabOrder(b,a); //改变后焦点链的顺序为 b-a-d-c
```

# 焦点切换

相关接口如下： 

```c++
// 等同于focusNextPrevChild(true)
bool QWidget::focusNextChild();
// 等同于focusNextPrevChild(false)
bool QWidget::focusPreviousChild();
// next==true：设置焦点链中下个`FocusPolicy`为`TabFocus`的窗口为焦点窗口
// next==false：设置焦点链中前一个`FocusPolicy`为`TabFocus`的窗口为焦点窗口
bool QWidget::focusNextPrevChild(bool next);
// 设置当前窗口为焦点窗口
void QWidget::setFocus(Qt::FocusReason reason);
// 取消焦点窗口
void QWidget::clearFocus();
```

# 焦点代理

代为接收焦点事件。比如，`Widget A` 是 `Widget B` 的焦点代理，则当 `B` 获得焦点时，实际获得并处理焦点的是 `A`。相关接口如下： 

```c++
//返回该窗口的焦点代理
QWidget* QWidget::focusProxy() const; 
//设置该窗口的焦点代理为w
void QWidget::setFocusProxy(QWidget* w);
```