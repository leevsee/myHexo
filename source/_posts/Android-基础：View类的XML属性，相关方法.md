---
title: Android 基础：View类的XML属性，相关方法
date: 2016-10-10 12:19:03
tags:
---

记录起来，以后用到就好找
<br/>


XML属性|相关方法|说明 
---|---|---
android:alpha| setAlpha(float) | 设置该组件的透明度
android:background | setBackgroundResource<br/>(int) | 设置该组件的背景颜色
android:clickable | setClickable(boolean) | 设置该组件是否可以激发单击事件
android:contentDesc<br/>ription | setContentDescription<br/>(CharSequence) | 设置该组件的主要描述信息
android:drawingCach<br/>eQuality | setDrawingCacheQuality<br/>(iint) | 设置该组件所使用的绘制缓存的质量
android:elevation | setElevation(float) | 设置改组件“浮”起来的高度，通过设置该属性可让改组件呈现3D效果
android:fadeScroll<br/>bars | setScrollbarFading<br/>Enabled(boolean) | 当不使用该组件的滚动条时，是否淡出显示滚动条
android:fadingEdge | setVerticalFadingEdge<br/>Enabled(boolean) | 设置滚动该组件时组件边界是否使用淡出效果
android:fadingEdge<br/>Length | getVerticalFading<br/>EdgeLength() | 设置淡出边界的长度
android:focusable | setFocusable(boolean) | 设置组件是否可以得到焦点
android:focusableIn<br/>TouchMode | setFocusableInTouchMode<br/>(boolean) | 设置该组件在触摸模式下是否可以得到焦点
android:id | setId(int) | 设置该组件的唯一标识。在Java 代码中可以通过 findViewById 来获取它
android:isScrollCo<br/>ntainer | setScrollContainer(boolean) | 设置该组件是否作为可滚动容器使用
android:keepScreenOn | setKeepScreenOn(boolean) | 设置该组件是否会强制手机屏幕一直打开
android:longClickable | setLongClickable(boolean) | 设置该组件是否可以响应长单击事件
android:minHeight | setMinimumHeight(int) | 设置该组件的最小高度
android:minWidth | setMinimumWidtht(int) | 设置该组件的最小宽度
android:nextFocusDown | SetNextFocusDownId(int) |  设置焦点在该组件上，且按向下键时获得焦点的组件 ID
android:nextFocusLeft | setNextFocusLeftId(int) | 设置焦点在该组件上，且按向左键时获得焦点的组件 ID
android:nextFocusRig<br/>ht | setNextFocusRightId(int) | 设置焦点在该组件上，且按向右键时获得焦点的组件 ID
android:nextFocusUp | setNextFocusUpId(int) | 设置焦点在该组件上，且按向上键时获得焦点的组件 ID
android:onClick |  | 为该组件的单击事件绑定监听器
android:padding | setPadding(int,int,int,int) | 在组件的四边设置填充区域
android:paddingLeft | setPadding(int,int,int,int) | 在组件的左边设置填充区域
android:paddingTop | setPadding(int,int,int,int) | 在组件的上边设置填充区域
android:paddingRight | setPadding(int,int,int,int) | 在组件的右边设置填充区域
android:paddingBottom | setPadding(int,int,int,int) | 在组件的下边设置填充区域
android:rotation | setRotation(float) | 设置该组件旋转的角度
android:rotationX | setRotationX(float) | 设置该组件绕X 轴旋转的角度
android:rotationY | setRotationY(float) | 设置该组件绕Y 轴旋转的角度
android:saveEnabled | setSaveEnabled(boolean) | 如果设置为false ，那当该组件被冻结时不会保存它的状态
android:scaleX | setScaleX(float) | 设置该组件在水平方向的缩放比
android:scaleY | setScaleY(float) | 设置该组件在垂直方向的缩放比
android:scrollX |   | 该组件初始化后的水平滚动偏移
android:scrollY |  | 该组件初始化后的垂直滚动偏移
android:scrollbarAl<br/>waysDrawHorizontalTrack |  | 设置该组件是否总是显示水平滚动条的轨迹
android:scrollbarAl<br/>waysDrawVerticalTrack |  | 设置该组件是否总是显示垂直滚动条的轨迹
android:scrollbarDe<br/>faultDelayBeforeFade | setScrollbarDefault<br/>DelayBeforeFade(int) | 设置滚动条在淡出隐藏之前延迟多少毫秒
android:scrollbarFa<br/>deDuration | setScrollbarFade<br/>Duration(int) | 设置滚动条淡出隐藏过程需要多少秒
android:scrollbarSize | setScrollbarSize(int) | 设置垂直滚动条的宽度和水平滚动条的高度
android:scrollbarStyle | setScrollbarStyle(int) | 设置滚动条的风格和位置。该属性支持如下属性：insideOverlay<br/>insideInset<br/>OutsideOverlay<br/>OutsideInset
android:scrollbarThu<br/>mbHorizontal |  | 设置该组件的水平滚动条的滑块对应的 Drawable 对象
android:scrollbarThum<br/>bVertical |  | 设置该组件的垂直滚动条的滑块对应的 Drawable 对象
android:scrollbarTrac<br/>kHorizontal |  | 设置该组件的水平滚动条的轨道对应的 Drawable 对象
android:scrollbarTrac<br/>kVertical |  | 设置该组件的垂直滚动条的轨道对应的 Drawable 对象
android:scrollbars |  | 定义该组件滚动时显示几个滚动条，该属性支持如下属性值。<br/>none ：不显示滚动条<br/>horizont ：显示水平滚动条<br/>vertical ：显示垂直滚动条
android:soundEffects<br/>Enabled | setSoundEffects<br/>Enabled(boolean) | 设置该组件被单击时是否使用音效
android:tag |  | 为该组件设置一个字符串类型的tag 值。接下来可通过 View 的 getTag() 获取该字符串，或通过 findViewWithTag() 查找该组件
android:transformPivotX | setPivotX(float) | 设置该组件旋转时旋转中心的 X 坐标
android:transformPivotY | setPivotY(float) | 设置该组件旋转时旋转中心的 Y 坐标
android:translationX | setTranslationX(float) | 设置该组件在X 方向上位移
android:translationY | setTranslationY(float) | 设置该组件在Y 方向上位移
android:visibility | setVisibility(int) | 设置该组件是否可见