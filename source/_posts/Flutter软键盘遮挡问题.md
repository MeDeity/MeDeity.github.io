---
title: 'Flutter软键盘遮挡问题'
date: 2020-03-31 18:18:11
categories: 
  - 前端-Flutter
tags:
  - 技巧
---
在Column布局中,如果存在输入框,在输入框获取焦点导致软键盘弹起时,经常会遇到控件被遮挡的问题.

这里提供两种解决思路:
1. resizeToAvoidBottomInset
通过将属性设置为true,在弹出软键盘的情况下,视图会发生重绘,以避免被软键盘遮挡.当然利用这个规则
,如果设置成false,则可以实现取消某些widget由于软键盘的弹出导致浮动的效果.
//TODO 此处应用图片以及相应的代码实现

2.将输入框包裹在可滚动的Widget中(例如ScrollView等),当弹出软键盘时,系统会将涉及到的控件进行滚动以避免被软键盘遮挡.
//TODO 此处应用图片以及相应的代码实现

showModalBottomSheet显示的内容中有输入框时,内容被弹出的软键盘所遮挡
在[Issue#18564](https://github.com/flutter/flutter/issues/18564)给出了详细的解决方案
```
showModalBottomSheet(
    isScrollControlled: true,
    builder: (BuildContext context) {
    return SingleChildScrollView(
        child:Container(
            padding:
            EdgeInsets.only(bottom: MediaQuery.of(context).viewInsets.bottom),
            child: AnyChild(),
        )
    );
});
```
处理后的效果图

<img src="/images/Flutter/hiddenByKeyBoard.jpg" width=50% height=50% align=center/>
