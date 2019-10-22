
 * 设计一个container
 * 实现内部控件自动换行。即里面的控件能够根据长度来判断当前行是否容得下它，进而决定是否转到下一行显示
 * 也叫做流式布局（FlowLayout）
 * 绘制流程分为三步：测量、布局、绘制
 * 分别对应：onMeasure()、onLayout()、onDraw()
 * 其中，他们三个的作用分别如下：
 *  1.onMeasure()：测量自己的大小，为正式布局提供建议。（注意，只是建议，至于用不用，要看onLayout）;
 *  2.onLayout():使用layout()函数对所有子控件布局；
 *  3.onDraw():根据布局的位置绘图；
 *  onMeasure()是用来测量当前控件大小的，给onLayout（）提供数值参考

**必须重写的构造函数**
``` 
public class MyLinLayout extends ViewGroup {
  public MyLinLayout(Context context) {
        super(context);
    }
    public MyLinLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
    public MyLinLayout(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }
 ```
 **测量**
 ```
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        /*
        首先是从父类传过来的建议宽度和高度值：widthMeasureSpec、heightMeasureSpec
        widthMeasureSpec和heightMeasureSpec各自都有它对应的模式，模式的由来分别来自于XML定义
         */
        int measureWidth = MeasureSpec.getSize(widthMeasureSpec);
        int measureHeight = MeasureSpec.getSize(heightMeasureSpec);
        //从他们里面利用MeasureSpec提取宽高值和对应的模式
        int measureWidthMode = MeasureSpec.getMode(widthMeasureSpec);
        int measureHeightMode = MeasureSpec.getMode(heightMeasureSpec);

        //接下来就是通过测量它所有的子控件来决定它所占位置的大小
        int height = 0;
        int width = 0;
        //得到子控件的数量
        int count = getChildCount();
        for (int i=0;i<count;i++) {

            //测量子控件
            View child = getChildAt(i);
            measureChild(child, widthMeasureSpec, heightMeasureSpec);
            //测量之后获得高度和宽度
            MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
            int childHeight = child.getMeasuredHeight()+lp.topMargin+lp.bottomMargin;
            int childWidth = child.getMeasuredWidth()+lp.leftMargin+lp.rightMargin;
            //得到最佳宽度并累加高度
            width = Math.max(childWidth, width);
            height += childHeight;
        }
        //最后设置高度和宽度
        // wrap_content-> MeasureSpec.AT_MOST(完全); match_parent -> MeasureSpec.EXACTLY(至多)
        setMeasuredDimension((measureWidthMode == MeasureSpec.EXACTLY) ? measureWidth: width,
                (measureHeightMode == MeasureSpec.EXACTLY) ? measureHeight: height);
    }
    ```
    **布局**
    ```
       @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        /*
        首先是从父类传过来的建议宽度和高度值：widthMeasureSpec、heightMeasureSpec
        widthMeasureSpec和heightMeasureSpec各自都有它对应的模式，模式的由来分别来自于XML定义
         */
        int measureWidth = MeasureSpec.getSize(widthMeasureSpec);
        int measureHeight = MeasureSpec.getSize(heightMeasureSpec);
        //从他们里面利用MeasureSpec提取宽高值和对应的模式
        int measureWidthMode = MeasureSpec.getMode(widthMeasureSpec);
        int measureHeightMode = MeasureSpec.getMode(heightMeasureSpec);

        //接下来就是通过测量它所有的子控件来决定它所占位置的大小
        int height = 0;
        int width = 0;
        //得到子控件的数量
        int count = getChildCount();
        for (int i=0;i<count;i++) {

            //测量子控件
            View child = getChildAt(i);
            measureChild(child, widthMeasureSpec, heightMeasureSpec);
            //测量之后获得高度和宽度
            MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
            int childHeight = child.getMeasuredHeight()+lp.topMargin+lp.bottomMargin;
            int childWidth = child.getMeasuredWidth()+lp.leftMargin+lp.rightMargin;
            //得到最佳宽度并累加高度
            width = Math.max(childWidth, width);
            height += childHeight;
        }
        //最后设置高度和宽度
        // wrap_content-> MeasureSpec.AT_MOST(完全); match_parent -> MeasureSpec.EXACTLY(至多)
        setMeasuredDimension((measureWidthMode == MeasureSpec.EXACTLY) ? measureWidth: width,
                (measureHeightMode == MeasureSpec.EXACTLY) ? measureHeight: height);
    }
    ```
    **container自己什么时候被布局**
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
**怎么获取margin参数**
```
<TextView android:text="第一个VIEW"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="10dp"
        android:background="#ff0000"/>
```
**重写**
```

@Override
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
**在测量函数里添加**
```

MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
int childHeight = child.getMeasuredHeight()+lp.topMargin+lp.bottomMargin;
int childWidth = child.getMeasuredWidth()+lp.leftMargin+lp.rightMargin;
```
**修改布局函数**
```
 MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
        int childHeight = child.getMeasuredHeight()+lp.topMargin+lp.bottomMargin;
        int childWidth = child.getMeasuredWidth()+lp.leftMargin+lp.rightMargin;
 
        child.layout(0, top, childWidth, top + childHeight);
        top += childHeight;
```


   
