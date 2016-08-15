title: Android自动换行ViewGroup
date: 2015-08-08 15:36:13
tags: [Android]
---
在Android开发过程中经常会碰到需要让某一些控件，比如：Textview作为标签，而这些标签还要能够自适应屏幕，即要求能够超过一行的宽度时自动换到下一行。Android本身是没有这个控件的。实现起来也不是很难。下面描述一下如何实现这个viewGroup。
首先，我们需要知道控件内部的子view的三个属性：**横向间距**、**纵向间距**、**子view方向(orientation)**。需要先设置控件属性。代码如下：

    <declare-styleable name="FlowLayout">
        <attr name="horizontalSpacing" format="dimension" />
        <attr name="verticalSpacing" format="dimension" />
        <attr name="orientation" format="enum">
            <enum name="horizontal" value="0" />
            <enum name="vertical" value="1" />
        </attr>
    </declare-styleable>
<!--more-->
在构造方法中获取到这些属性，留着之后使用：
		
	TypedArray a = context.obtainStyledAttributes(attributeSet, R.styleable.FlowLayout);
    horizontalSpacing = a.getDimensionPixelSize(R.styleable.FlowLayout_horizontalSpacing, 0);
    verticalSpacing = a.getDimensionPixelSize(R.styleable.FlowLayout_verticalSpacing, 0);
    orientation = a.getInteger(R.styleable.FlowLayout_orientation, HORIZONTAL);

之后要使用到**onMeasure()**方法给控件设置宽高大小。

	int sizeWidth = MeasureSpec.getSize(widthMeasureSpec) - this.getPaddingRight() - this.getPaddingLeft();
    int sizeHeight = MeasureSpec.getSize(heightMeasureSpec) - this.getPaddingTop() - this.getPaddingBottom();
	//分别获取到横向和竖向的specMode
    int modeWidth = MeasureSpec.getMode(widthMeasureSpec);
    int modeHeight = MeasureSpec.getMode(heightMeasureSpec);

因为控件本身可以横向放置也可以纵向放置，所以需要对横纵方向的mode都做处理。如果是横向放置就要使用横向的mode，反之亦然。

	if (orientation == HORIZONTAL) {
        size = sizeWidth;
        mode = modeWidth;
    } else {
        size = sizeHeight;
        mode = modeHeight;
    }

其中**size**表示整个控件的大小。

之后便是给子view进行位置的确定。也是需要先获取所有的子view并且调用子view的measure方法设置子view的实际宽高。

	final View child = getChildAt(i);
	if (child.getVisibility() == GONE) {
    	continue;
	}
	LayoutParams lp = (LayoutParams) child.getLayoutParams();
	child.measure(
       getChildMeasureSpec(widthMeasureSpec, this.getPaddingLeft() + this.getPaddingRight(), lp.width),
       getChildMeasureSpec(heightMeasureSpec, this.getPaddingTop() + this.getPaddingBottom(), lp.height)
	);

此时就需要使用到之前属性设置的子view横向和纵向的间距以及子view的宽高

	int hSpacing = this.getHorizontalSpacing(lp);
    int vSpacing = this.getVerticalSpacing(lp);
    int childWidth = child.getMeasuredWidth();
	int childHeight = child.getMeasuredHeight();

此时在循环中要进行计算，如果当前子view的边界再加上边距和下一个子view的宽或者高超出单行的宽度值时则需要进行换行。

    int childLength;
    int childThickness;
    int spacingLength;
    int spacingThickness;

    if (orientation == HORIZONTAL) {
        childLength = childWidth;
        childThickness = childHeight;
        spacingLength = hSpacing;
        spacingThickness = vSpacing;
    } else {
        childLength = childHeight;
        childThickness = childWidth;
        spacingLength = vSpacing;
        spacingThickness = hSpacing;
    }

    lineLength = lineLengthWithSpacing + childLength;
    lineLengthWithSpacing = lineLength + spacingLength;

    boolean newLine = lp.newLine || (mode != MeasureSpec.UNSPECIFIED && lineLength > size);
    if (newLine) {
        prevLinePosition = prevLinePosition + lineThicknessWithSpacing;

        lineThickness = childThickness;
        lineLength = childLength;
        lineThicknessWithSpacing = childThickness + spacingThickness;
        lineLengthWithSpacing = lineLength + spacingLength;
    }

    lineThicknessWithSpacing = Math.max(lineThicknessWithSpacing, childThickness + spacingThickness);
    lineThickness = Math.max(lineThickness, childThickness);

之后便是给自定义view设置起始位置

    int posX;
    int posY;
    if (orientation == HORIZONTAL) {
        posX = getPaddingLeft() + lineLength - childLength;
        posY = getPaddingTop() + prevLinePosition;
    } else {
        posX = getPaddingLeft() + prevLinePosition;
        posY = getPaddingTop() + lineLength - childHeight;
    }
    lp.setPosition(posX, posY);

此时已经可以给控件和子view设置好宽高和所要放置的相应位置。紧接着需要调用**onLayout()**方法，此时如果需要让控件左对齐显示则直接使用之前设置好的位置便可以，代码如下

    final int count = getChildCount();
    for (int i = 0; i < count; i++) {
        View child = getChildAt(i);
        LayoutParams lp = (LayoutParams) child.getLayoutParams();
        child.layout(lp.x, lp.y, lp.x + child.getMeasuredWidth(), lp.y + child.getMeasuredHeight());
    }

而如果当前使用到的标签需要居中显示，则需要对每行的控件获取到并且判断当前行左对齐放置时距右边的边距。此时将右边距除以二，再将当前行第一个子view的左起位置替换则可以达到Group中的子view居中效果。

此时则需要两次循环，一次获取单行的居中时距左边边距，一次进行控件的摆放，代码如下：

    final int count = getChildCount();
    int tmpY = 0;
    int tmpX = 0;
    for (int i = 0; i < count; i++) {
        View child = getChildAt(i);
        LayoutParams lp = (LayoutParams) child.getLayoutParams();
        if (lp.y == tmpY) {
            tmpX = lp.x + child.getMeasuredWidth();
        } else {
            tmpY = lp.y;
            int temp = (ScreenUtils.getScreenWidth() - tmpX) / 2 - 5;
            DebugLog.d("temp:" + temp);
            startX.add(temp);
        }
        if (i == count - 1) {
            tmpX = lp.x + child.getMeasuredWidth();
            int temp = (ScreenUtils.getScreenWidth() - tmpX) / 2;
            startX.add(temp);
        }
    }

    int position = 0;
    int tmpY2 = 0;
    for (int i = 0; i < count; i++) {
        View child = getChildAt(i);
        LayoutParams lp = (LayoutParams) child.getLayoutParams();
        if (lp.y == 0) position = 0;
        if (tmpY2 < lp.y) {
            position++;
            tmpY2 = lp.y;
        }
        child.layout(startX.get(position) + lp.x, lp.y, startX.get(position) + lp.x + child.getMeasuredWidth(),
                lp.y + child.getMeasuredHeight());
    }

这样便完成了对自定义换行viewGroup的实现。