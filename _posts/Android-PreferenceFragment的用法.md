title: Android PreferenceFragment的用法
date: 2016-03-09 12:07:07
tags:
- Android
- UI
- Preference
---
# 简介
## 使用来定义App 关于 界面的一个Fragment，将条目定义在xml中，可直接跳转Activity
# 用法
## xml定义

```bash
<PreferenceCategory android:title="就看天气">
        <Preference
            android:key="introduction"
            android:title="应用介绍"/>
        <Preference
            android:key="current_version"
            android:title="当前版本"/>
        <Preference
            android:key="check_version"
            android:title="检查更新"/>
    </PreferenceCategory>
```
---

```bash
public void onCreate(Bundle savedInstanceState){
	 addPreferencesFromResource(R.xml.about);
     mIntroduction = findPreference(INTRODUCTION);
     mIntroduction.setOnPreferenceClickListener(this);
}
```
---
按钮点击事件
```bash
 @Override public boolean onPreferenceClick(Preference preference) {
    if(mShare == preference){

    }
}
```
