title: Android自定义控件包含两个TextView
date: 2016-03-23 13:02:14
tags:
- Android
- 自定义控件
---
#前言
## 为了解决美芽个人页面布局类似显示 “粉丝 12”，颜色不一，间距确定而开发的；
# 设定属性
## 在attr.XML中设定如下属性
```bash
 <declare-styleable name="MultiTextViewItem">
       <attr name="mtv_textColor_left" format="color"/>
       <attr name="mtv_textColor_right" format="color"/>
       <attr name="mtv_textSize" format="dimension|reference"/>
       <attr name="mtv_tv_margin" format="dimension"/>
       <attr name="mtv_text_left" format="string|reference"/>
       <attr name="mtv_text_right" format="string|reference"/>
   </declare-styleable>
```

#新建类继承于LinearLayout;
##
```bash
/**
 * Created by shouwang on 16/3/23.
 * 个人页面
 * 用来显示关注 12 粉丝 4
 * 便于调整间距 来与设计稿相符
 */
public class MultiTextViewItem extends LinearLayout {
    TextView tvLeft;
    TextView tvRight;
    String textLeft;
    String textRight;


    public MultiTextViewItem(Context context) {
        super(context);
    }

    public MultiTextViewItem(Context context, AttributeSet attrs) {
        super(context, attrs);
        LayoutInflater inflater = LayoutInflater.from(context);
        inflater.inflate(R.layout.view_multitle_textview, this, true);
        findViews();
        setupByAttrs(context,attrs);

    }

    public MultiTextViewItem(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    public MultiTextViewItem(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context, attrs, defStyleAttr, defStyleRes);
    }

    void findViews() {
        tvLeft = (TextView) findViewById(R.id.tvLeft);
        tvRight = (TextView) findViewById(R.id.tvRight);
    }

    void setupByAttrs(Context context, AttributeSet attrs) {
        TypedArray typedArray = context.getTheme().obtainStyledAttributes(attrs, R.styleable.MultiTextViewItem, 0, 0);
        String text_left = typedArray.getString(R.styleable.MultiTextViewItem_mtv_text_left);
        String text_right = typedArray.getString(R.styleable.MultiTextViewItem_mtv_text_right);
        int color_left = typedArray.getInt(R.styleable.MultiTextViewItem_mtv_textColor_left, -1);
        int color_right = typedArray.getInt(R.styleable.MultiTextViewItem_mtv_textColor_right, -1);
        int tv_margin = typedArray.getDimensionPixelSize(R.styleable.MultiTextViewItem_mtv_tv_margin, -1);
        float text_size=typedArray.getDimension(R.styleable.MultiTextViewItem_mtv_textSize,12);
        LinearLayout.LayoutParams lp = (LinearLayout.LayoutParams) tvRight.getLayoutParams();
        lp.setMargins(tv_margin, 0, 0, 0);
        tvRight.setLayoutParams(lp);

        tvLeft.setText(text_left);
        tvRight.setText(text_right);
        tvLeft.setTextColor(color_left);
        tvRight.setTextColor(color_right);
        //==字体单位
        tvLeft.setTextSize(TypedValue.COMPLEX_UNIT_PX,text_size);
        tvRight.setTextSize(TypedValue.COMPLEX_UNIT_PX,text_size);
        setOrientation(HORIZONTAL);
        typedArray.recycle();
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    }

    public void setTextLeft(String text){
        textLeft=text;
        tvLeft.setText(text);
    }
    public void setTvRight(String text){
        textRight=text;
        tvRight.setText(text);
    }
```

#新建布局
##
```bash
<?xml version="1.0" encoding="utf-8"?>
<merge xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="horizontal"
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              >
<TextView
    android:id="@+id/tvLeft"
    android:gravity="center"
    android:layout_height="wrap_content"
    android:layout_width="wrap_content"
    android:textColor="@color/text_light_black_light"
    android:textSize="11sp"

    android:text="关注"
    />
    <TextView
        android:id="@+id/tvRight"
        android:gravity="center"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="32259"
        android:textSize="11sp"
        android:textColor="@color/text_black_lighter"/>
</merge>
```
#运用控件
## xml中引入
```bash
 <cn.app.meiya.aa.widget.MultiTextViewItem
                    android:id="@+id/tvFollowCount"
                    android:layout_height="wrap_content"
                    android:layout_width="wrap_content"
                    android:includeFontPadding="false"
                    android:paddingBottom="12dp"
                    android:paddingLeft="10dp"
                    android:paddingRight="10dp"
                    android:paddingTop="12dp"
                    app:mtv_textSize="13sp"
                    app:mtv_text_left="关注"
                    app:mtv_text_right="322"
                    app:mtv_textColor_right="@color/text_black_lighter"
                    app:mtv_textColor_left="@color/text_light_black_light"
                    app:mtv_tv_margin="7dp">

                </cn.app.meiya.aa.widget.MultiTextViewItem>
```
#关于抽象布  http://blog.csdn.net/xyz_lmn/article/details/14524567
# 关于自定义属性
## 参考 http://blog.csdn.net/android_tutor/article/details/5508615
