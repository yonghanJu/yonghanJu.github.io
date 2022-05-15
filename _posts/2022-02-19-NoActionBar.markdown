---
layout: post
title:  NoActionBar
date:   2022-02-19 16:07:00 +0900
categories:   Android
---

## NoActionBar

***

<Br>

### 액션바를 없에는 방법

<br>

1. res-values-themes 에 스타일 추가하기

```xml
<resources xmlns:tools="http://schemas.android.com/tools">
    <!-- Base application theme. -->
    <style name="Theme.NumberPicker" parent="Theme.MaterialComponents.DayNight.DarkActionBar">
        <!-- Primary brand color. -->
        <item name="colorPrimary">@color/purple_500</item>
        <item name="colorPrimaryVariant">@color/purple_700</item>
        <item name="colorOnPrimary">@color/white</item>
        <!-- Secondary brand color. -->
        <item name="colorSecondary">@color/teal_200</item>
        <item name="colorSecondaryVariant">@color/teal_700</item>
        <item name="colorOnSecondary">@color/black</item>
        <!-- Status bar color. -->
        <item name="android:statusBarColor" tools:targetApi="l">?attr/colorPrimaryVariant</item>
        <!-- Customize your theme here. -->
    </style>

    <!--새롭게 추가된 스타일 "NoActionBar"-->
    <style name="Theme.NumberPicker.NoActionBar" parent="Theme.MaterialComponents.DayNight.NoActionBar"></style>
</resources>
```

<br><br>

2. Manifesr Acticity Theme에 위 스타일 추가하기

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.jyh.numberpicker">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.NumberPicker">

        <!-- 새롭게 추가된 Theme 스타일-->
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:theme="@style/Theme.NumberPicker.NoActionBar">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

<br>

# 배경 색 바꾸기

```xml
<!-- res/values/themes.xml 스타일 안에 item 추가-->
<item name="android:windowBackground">@color/name</item>
```

<br>

# 상단 바 색 바꾸기

```xml
<!-- res/values/themes.xml 스타일 안에 item 추가-->
<item name="android:statusBarColor">@color/name</item>

<!--만약 밝은 색 또는 transparent(#00000000) 지정시 자동으로 statusBar 아이콘 색을 어둡게 변경 -->
<item name="android:windowLightStatusBar">true</item>
```

<br>