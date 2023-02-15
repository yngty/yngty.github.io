---
title: Qt 之 焦点处理
date: 2023-02-15 17:08:09
tags:
- 焦点
categories:
- Qt
---

# 焦点事件

```c++
void focusInEvent(QFocusEvent *event) override;
void focusOutEvent(QFocusEvent *event) override;
```
# 焦点策略

### enum Qt::FocusPolicy


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

`Tab` 次序。

```c++
static void setTabOrder(QWidget *, QWidget *);
```
# 焦点代理

```c++
void setFocusProxy(QWidget *);
```