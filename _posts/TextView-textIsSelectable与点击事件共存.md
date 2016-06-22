title: TextView textIsSelectable与点击事件共存
date: 2016-06-22 10:14:47
tags:
- Android
- GestureDetectorCompat
- 手势
---
#问题所在
## 如果一个TextView设置了textIsSelectable属性来实现文字可选复制，这时候设置其点击事件（OnClickListener）会出现第二次点击才响应事件；
#解决
## 采用OnTouch代替
# 实现
##
```bash
 //TextView 设置了TextIsSelectable属性实现选择复制,导致点击事件在点两下才响应/
    //采用onTouch拦截
    public void setSelectableTextViewClick(final IntentData intentData, TextView textView) {
        final GestureDetectorCompat gestureDetector = new GestureDetectorCompat(mActivity, new SimpleOnGestureListener());
        gestureDetector.setOnDoubleTapListener(new GestureDetector.OnDoubleTapListener() {
            @Override
            public boolean onSingleTapConfirmed(MotionEvent e) {
                GoodDetailActivity.start(intentData);//点击
                return false;
            }

            @Override
            public boolean onDoubleTap(MotionEvent e) {
                return false;
            }

            @Override
            public boolean onDoubleTapEvent(MotionEvent e) {
                return false;
            }
        });

        textView.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View v, MotionEvent event) {
                gestureDetector.onTouchEvent(event);
                return false;
            }
        });

    }

```
