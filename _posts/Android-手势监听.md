title: Android 手势监听
date: 2017-01-04 15:26:45
tags:
---
#ScaleGestureDetector
缩放事件，
```java   @Override
    public boolean onTouchEvent(MotionEvent event) {
        boolean scaleEvent = mScaleGestureDetector.onTouchEvent(event);//缩放事件
        boolean gestureEvent = mGestureDetector.onTouchEvent(event);//处理一般事件
        //如果没有焦点的状态应该交给父类,缩放背景。
        //二维码被选中时缩放就缩放二维码 背景不改变。
        return !isFocusState()?false:scaleEvent || gestureEvent;
    }
```
对于手势监听，只有当onTouch 返回为True的情况下，才会执行，（也就是说event.getPonterCount()返回才会大于1）

#关于这个类的用法:
```java mScaleGestureDetector = new ScaleGestureDetector(getContext(), new           ScaleGestureDetector.SimpleOnScaleGestureListener() {
            @Override
            public boolean onScale(ScaleGestureDetector detector) {
                if (isFocusState()) {
                    float scaleFactor = mScaleFactor * detector.getScaleFactor();
                    float distance = calculateDistance(mImageCenterCorners[0], mImageCenterCorners[1], detector.getFocusX(), detector.getFocusY());
                    float scale = distance / mOldDistance;
                    float[] currentCorners = Arrays.copyOf(mCurrentImageCorners, mCurrentImageCorners.length);
                    mTempMatrix.reset();
                    mTempMatrix.postScale(scaleFactor, scaleFactor, mImageCenterCorners[0], mImageCenterCorners[1]);
                    mTempMatrix.mapPoints(currentCorners); //限制缩放不能小于原始尺寸四分之一//
                    float currentSize = calculateDistance(currentCorners[0], currentCorners[1], currentCorners[2], currentCorners[3]);
                    if (currentSize >= mMinWidth) {
                        mCurrentImageCorners = currentCorners;
                        mOldScale = scale;
                    }
                    invalidate();
                    return true;
                }
                return false;
            }
        });
```
#GestureDetector
一般手势监听（包括单击、滑动、触摸）

#关于这类的用法
```java  mGestureDetector = new GestureDetector(getContext(), new GestureDetector.SimpleOnGestureListener() {
            @Override
            public boolean onDown(MotionEvent event) {
                float x = event.getX();
                float y = event.getY();
                mTouchDownX = x;
                mTouchDownY = y;
                if (isFocusState()) {
                    if (mBtnEditDrawable != null && mBtnEditDrawable.getBounds().contains((int) x, (int) y)) {
                        mTouchBtnType = TOUCH_BTN_EDIT;
                        return true;
                    } else if (mBtnRotationDrawable != null && mBtnRotationDrawable.getBounds().contains((int) x, (int) y)) {
                        mTouchBtnType = TOUCH_BTN_CONTROLL;
                        mOldDistance = calculateDistance(mImageCenterCorners[0], mImageCenterCorners[1], x, y);
                        mOldScale = 1;
                        return true;
                    } else if (mBtnDelDrawable != null && mBtnDelDrawable.getBounds().contains((int) x, (int) y)) {
                        mTouchBtnType = TOUCH_BTN_DELETE;
                        return true;
                    }
                }
                mTouchBtnType = TOUCH_NO_BTN;
                return false;
            }
            @Override
            public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY) {
                //放大//拖动
                if (mTouchBtnType == StickerView.TOUCH_BTN_CONTROLL) {
                    Log.d(TAG, "processScale");
                    processScale(e2.getX(), e2.getY());
                    invalidate();
                    return true;
                }
                return ((MarkLayout) getParent()).processScrollEvent(e1, e2, distanceX, distanceY);
            }
            @Override
            public boolean onSingleTapUp(MotionEvent event) {
                //触摸
                //删除
                if (mTouchBtnType == StickerView.TOUCH_BTN_DELETE
                        && (mTouchDownX == event.getX() && mTouchDownY == event.getY())) {
                    delete();
                    return true;
                }
                //编辑
                if (mTouchBtnType == TOUCH_BTN_EDIT
                        && (mTouchDownX == event.getX() && mTouchDownY == event.getY())) {
                    editTDCode();
                    return true;
                }
                return ((MarkLayout) getParent()).processSingleTapUpEvent(event);
            }
        });
    }
```











