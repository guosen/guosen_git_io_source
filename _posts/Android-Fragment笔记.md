title: Android Fragment笔记
date: 2016-03-20 10:50:32
tags:
- Andoid
- Fragment
---
# 关于Fragment
## Fragment 释义为片段;界面的一部分 

# 与Activity之间的关联 
## 依附于宿主Activity，同样拥有生命周期
onAttach、onCreate、onCreateView、onStart、onResume、onPause、onStop、onDestroyView、onDestroy
#构造方式
##xml配置
```bash
<FrameLayout
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <fragment
        android:tag="tag"
        android:name="com.asha.fragmentdemo.MyDialogFragment"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</FrameLayout>
```
