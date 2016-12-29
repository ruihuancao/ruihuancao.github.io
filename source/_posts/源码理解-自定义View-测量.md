---
title: 自定义View-测量
date: 2015-10-18 12:08:59
categories:
- 开发
- Android
tags:
- View
---
要了解自定义View首先要搞清楚，AndroidView的体系,这个很重要,大局观，理清条例。UI中最常用的就2个：控件和布局。所有的控件直接或间接继承自View。布局则继承自ViewGroup，Viewgroup继承自View。到最后只要关注这两个类View和ViewGroup的实现就可以。其他的控件，布局都是对父类扩展。我们关注2个问题，View的事件和View的绘制。
<!--more-->

## 前置
Activity通常被认为是一个页面，从Activity到View，从源码可以看到：Activity->PhoneWindow->DecorView
DecorView就是我们看到的整个界面的父容器,继承自FrameLayout.

## View绘制
View绘制流程：measure ->layout->draw
方法：onMeasure ->onLayout->onDraw
源码在ViewRootImpl的performTraversals中
### 自定义View实现
首先按照常用创建自定义View的方法，即：继承View类，大致如下：
```
public class CustomView extends View {

    private Paint paint ;
    float size ;
    public CustomView(Context context) {
        this(context, null);
    }

    public CustomView(Context context, AttributeSet attrs) {
        super(context, attrs);

        // 获取自定义属性
        TypedArray a = context.obtainStyledAttributes(attrs, R.styleable.CustomView);
        size = a.getDimension(R.styleable.CustomView_defaultSize, 0);
        a.recycle();

        init();
    }

    private void init(){
    }
}
```
### 测量
View类中的onMeasure()
```
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}

// 有背景，获取最小高度，没有背景，取背景和最小高度的最大值
protected int getSuggestedMinimumHeight() {
    return (mBackground == null) ? mMinHeight : max(mMinHeight, mBackground.getMinimumHeight());
}
protected int getSuggestedMinimumWidth() {
    return (mBackground == null) ? mMinWidth : max(mMinWidth, mBackground.getMinimumWidth());
}
// size 默认大小，measureSpec 测量单位
// 测量模式是一个32位值，前两位测量模式，后30位测量大小
public static int getDefaultSize(int size, int measureSpec) {
        int result = size;
        // measureSpec 来自于父容器
        // 获取测量模式
        int specMode = MeasureSpec.getMode(measureSpec);
        // 获取测量大小
        int specSize = MeasureSpec.getSize(measureSpec);

        switch (specMode) {
        case MeasureSpec.UNSPECIFIED:
            result = size;
            break;
        case MeasureSpec.AT_MOST:
        case MeasureSpec.EXACTLY:
            result = specSize;
            break;
        }
        return result;
    }
```
MeasureSpec在的产生在父容器中,默认实现，查看ViewGroup
```
protected void measureChildWithMargins(View child,
            int parentWidthMeasureSpec, int widthUsed,
            int parentHeightMeasureSpec, int heightUsed) {
        final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                        + widthUsed, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                        + heightUsed, lp.height);

        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }

    public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
            int specMode = MeasureSpec.getMode(spec);
            int specSize = MeasureSpec.getSize(spec);

            int size = Math.max(0, specSize - padding);

            int resultSize = 0;
            int resultMode = 0;

            switch (specMode) {
            // Parent has imposed an exact size on us
            case MeasureSpec.EXACTLY:
                if (childDimension >= 0) {
                    resultSize = childDimension;
                    resultMode = MeasureSpec.EXACTLY;
                } else if (childDimension == LayoutParams.MATCH_PARENT) {
                    // Child wants to be our size. So be it.
                    resultSize = size;
                    resultMode = MeasureSpec.EXACTLY;
                } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                    // Child wants to determine its own size. It can't be
                    // bigger than us.
                    resultSize = size;
                    resultMode = MeasureSpec.AT_MOST;
                }
                break;

            // Parent has imposed a maximum size on us
            case MeasureSpec.AT_MOST:
                if (childDimension >= 0) {
                    // Child wants a specific size... so be it
                    resultSize = childDimension;
                    resultMode = MeasureSpec.EXACTLY;
                } else if (childDimension == LayoutParams.MATCH_PARENT) {
                    // Child wants to be our size, but our size is not fixed.
                    // Constrain child to not be bigger than us.
                    resultSize = size;
                    resultMode = MeasureSpec.AT_MOST;
                } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                    // Child wants to determine its own size. It can't be
                    // bigger than us.
                    resultSize = size;
                    resultMode = MeasureSpec.AT_MOST;
                }
                break;

            // Parent asked to see how big we want to be
            case MeasureSpec.UNSPECIFIED:
                if (childDimension >= 0) {
                    // Child wants a specific size... let him have it
                    resultSize = childDimension;
                    resultMode = MeasureSpec.EXACTLY;
                } else if (childDimension == LayoutParams.MATCH_PARENT) {
                    // Child wants to be our size... find out how big it should
                    // be
                    resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                    resultMode = MeasureSpec.UNSPECIFIED;
                } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                    // Child wants to determine its own size.... find out how
                    // big it should be
                    resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                    resultMode = MeasureSpec.UNSPECIFIED;
                }
                break;
            }
            //noinspection ResourceType
            return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
        }

```

可以看出，View测量的大小取决与父容器的测量模式和自身的布局参数。View测量本身大小, 直接完成。ViewGroup则需要遍历去调用所有 child 的measure 方法，各个 child 再递归去执行这个流程
