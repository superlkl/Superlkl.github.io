# 绘制ViewGroup
**要求：绘制一个ViewGroup装下三个控件**

布局文件
```
<com.harvic.simplelayout.MyLinLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="#ff00ff"
    tools:context=".MainActivity">
 
    <TextView android:text="第一个VIEW"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
 
    <TextView android:text="第二个VIEW"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
 
    <TextView android:text="第三个VIEW"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
 
</com.harvic.simplelayout.MyLinLayout>
```
**1、测量**
```
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    int measureWidth = MeasureSpec.getSize(widthMeasureSpec);
    int measureHeight = MeasureSpec.getSize(heightMeasureSpec);
    int measureWidthMode = MeasureSpec.getMode(widthMeasureSpec);
    int measureHeightMode = MeasureSpec.getMode(heightMeasureSpec);
 
    int height = 0;
    int width = 0;
    int count = getChildCount();
    for (int i=0;i<count;i++) {
		//测量子控件
        View child = getChildAt(i);
        measureChild(child, widthMeasureSpec, heightMeasureSpec);
		//获得子控件的高度和宽度
        int childHeight = child.getMeasuredHeight();
        int childWidth = child.getMeasuredWidth();
		//得到最大宽度，并且累加高度
        height += childHeight;
        width = Math.max(childWidth, width);
    }
 
    setMeasuredDimension((measureWidthMode == MeasureSpec.EXACTLY) ? measureWidth: width, (measureHeightMode == MeasureSpec.EXACTLY) ? measureHeight: height);
}
```
**2.布局**
```
protected void onLayout(boolean changed, int l, int t, int r, int b) {
    int top = 0;
    int count = getChildCount();
    for (int i=0;i<count;i++) {
 
        View child = getChildAt(i);
 
        int childHeight = child.getMeasuredHeight();
        int childWidth = child.getMeasuredWidth();
 
        child.layout(0, top, childWidth, top + childHeight);
        top += childHeight;
    }
}
```
**container什么时候被布局？**
```
public final void layout(int l, int t, int r, int b) {  
    boolean changed = setFrame(l, t, r, b); //设置每个视图位于父视图的坐标轴  
    if (changed || (mPrivateFlags & LAYOUT_REQUIRED) == LAYOUT_REQUIRED) {  
        if (ViewDebug.TRACE_HIERARCHY) {  
            ViewDebug.trace(this, ViewDebug.HierarchyTraceType.ON_LAYOUT);  
        }  
  
        onLayout(changed, l, t, r, b);//回调onLayout函数 ，设置每个子视图的布局  
        mPrivateFlags &= ~LAYOUT_REQUIRED;  
    }  
    mPrivateFlags &= ~FORCE_LAYOUT;  
 ```
 **如果子控件有自己的属性如magin怎么办？**
 ```
 <TextView android:text="第一个VIEW"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="10dp"
        android:background="#ff0000"/>
```
**重写一个返回参数的函数**
```@Override
protected LayoutParams generateLayoutParams(LayoutParams p) {
    return new MarginLayoutParams(p);
}
 
@Override
public LayoutParams generateLayoutParams(AttributeSet attrs) {
    return new MarginLayoutParams(getContext(), attrs);
}
 
@Override
protected LayoutParams generateDefaultLayoutParams() {
    return new MarginLayoutParams(LayoutParams.MATCH_PARENT,
            LayoutParams.MATCH_PARENT);
}
```
**修改测量函数**
```
MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
int childHeight = child.getMeasuredHeight()+lp.topMargin+lp.bottomMargin;
int childWidth = child.getMeasuredWidth()+lp.leftMargin+lp.rightMargin;
```
**修改布局函数**
```
@Override
protected void onLayout(boolean changed, int l, int t, int r, int b) {
    int top = 0;
    int count = getChildCount();
    for (int i=0;i<count;i++) {
 
        View child = getChildAt(i);
 
        MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
        int childHeight = child.getMeasuredHeight()+lp.topMargin+lp.bottomMargin;
        int childWidth = child.getMeasuredWidth()+lp.leftMargin+lp.rightMargin;
 
        child.layout(0, top, childWidth, top + childHeight);
        top += childHeight;
    }
}
```

# 绘制流式布局
假设有十个子控件在一个container里面
```
<style name="text_flag_01">
    <item name="android:layout_width">wrap_content</item>
    <item name="android:layout_height">wrap_content</item>
    <item name="android:layout_margin">4dp</item>
    <item name="android:background">@drawable/flag_01</item>
    <item name="android:textColor">#ffffff</item>
</style>
```
截取其中一个控件
```
<TextView
            style="@style/text_flag_01"
            android:background="@drawable/flag_03"
            android:text="IT工程师"
            android:textColor="#43BBE7" />
```
计算流式布局需要的空间
```
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
   super.onMeasure(widthMeasureSpec, heightMeasureSpec);
   int measureWidth = MeasureSpec.getSize(widthMeasureSpec);
   int measureHeight = MeasureSpec.getSize(heightMeasureSpec);
   int measureWidthMode = MeasureSpec.getMode(widthMeasureSpec);
   int measureHeightMode = MeasureSpec.getMode(heightMeasureSpec);
 
 
   int lineWidth = 0;
   int lineHeight = 0;
   int height = 0;
   int width = 0;
   int count = getChildCount();
   for (int i=0;i<count;i++){
       View child = getChildAt(i);
       measureChild(child,widthMeasureSpec,heightMeasureSpec);
       
       MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
       int childWidth = child.getMeasuredWidth() + lp.leftMargin +lp.rightMargin;
       int childHeight = child.getMeasuredHeight() + lp.topMargin + lp.bottomMargin;
 
       if (lineWidth + childWidth > measureWidth){
           //需要换行
           width = Math.max(lineWidth,width);
           height += lineHeight;
           //因为由于盛不下当前控件，而将此控件调到下一行，所以将此控件的高度和宽度初始化给lineHeight、lineWidth
           lineHeight = childHeight;
           lineWidth = childWidth;
       }else{
           // 否则累加值lineWidth,lineHeight取最大高度
           lineHeight = Math.max(lineHeight,childHeight);
           lineWidth += childWidth;
       }
 
       //最后一行是不会超出width范围的，所以要单独处理
       if (i == count -1){
           height += lineHeight;
           width = Math.max(width,lineWidth);
       }
 
   }
   //当属性是MeasureSpec.EXACTLY时，那么它的高度就是确定的，
   // 只有当是wrap_content时，根据内部控件的大小来确定它的大小时，大小是不确定的，属性是AT_MOST,此时，就需要我们自己计算它的应当的大小，并设置进去
   setMeasuredDimension((measureWidthMode == MeasureSpec.EXACTLY) ? measureWidth
           : width, (measureHeightMode == MeasureSpec.EXACTLY) ? measureHeight
           : height);
}
```
布局子项
```
protected void onLayout(boolean changed, int l, int t, int r, int b) {
   int count = getChildCount();
   int lineWidth = 0;
   int lineHeight = 0;
   int top=0,left=0;
   for (int i=0; i<count;i++){
       View child = getChildAt(i);
       MarginLayoutParams lp = (MarginLayoutParams) child
               .getLayoutParams();
       int childWidth = child.getMeasuredWidth()+lp.leftMargin+lp.rightMargin;
       int childHeight = child.getMeasuredHeight()+lp.topMargin+lp.bottomMargin;
 
       if (childWidth + lineWidth >getMeasuredWidth()){
           //如果换行,当前控件将跑到下一行，从最左边开始，所以left就是0，而top则需要加上上一行的行高，才是这个控件的top点;
           top += lineHeight;
           left = 0;
           //同样，重新初始化lineHeight和lineWidth
           lineHeight = childHeight;
           lineWidth = childWidth;
       }else{
           lineHeight = Math.max(lineHeight,childHeight);
           lineWidth += childWidth;
       }
       //计算childView的left,top,right,bottom
       int lc = left + lp.leftMargin;
       int tc = top + lp.topMargin;
       int rc =lc + child.getMeasuredWidth();
       int bc = tc + child.getMeasuredHeight();
       child.layout(lc, tc, rc, bc);
       //将left置为下一子控件的起始点
       left+=childWidth;
   }
}
```
# 绘制瀑布流布局
