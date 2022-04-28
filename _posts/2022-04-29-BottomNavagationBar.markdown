---
layout: post
title:  Bottom Navigation Menu
date:   2022-04-29 01:00:00 +0900
categories:   Codelab
---

# res/menu/bottom_navigation_menu.xml 생성

메뉴에 아이탬 3개 만들기, 아이콘은 벡터 이미지 사용

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/home"
        android:title="@string/home"
        android:icon="@drawable/ic_baseline_home_24"/>
    <item android:id="@+id/chatList"
        android:title="@string/chatting"
        android:icon="@drawable/ic_baseline_chat_24"/>
    <item android:id="@+id/myPage"
        android:title="@string/myInfo"
        android:icon="@drawable/ic_baseline_person_24"/>
</menu>
```

<br>

# res/drawable/selector_menu_color.xml 생성

바텀 네비게이션 메뉴가 눌렸을 때 아이콘 색상 지정

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:color = "@color/black" android:state_checked="true"/>
    <item android:color = "@color/gray_cc" android:state_checked="false"/>
    <!-- gray_cc = #cccccc -->
</selector>
```

<BR>

# 바텀 네비게이션 뷰 추가

selector_menu_color.xml 파일을 iconColor로 지정해주자

rippleColor 는 null 값으로 지정, 텍스트는 검정으로 지정

```xml
    <com.google.android.material.bottomnavigation.BottomNavigationView
        android:id="@+id/bottomNavigationView"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        app:itemIconTint="@drawable/selector_menu_color"
        app:itemRippleColor="@null"
        app:itemTextColor="@color/black"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:menu="@menu/bottom_navigation_menu" />
```

<img src="/public/img/bottomNavagationImg.png"  width="300" height="800">