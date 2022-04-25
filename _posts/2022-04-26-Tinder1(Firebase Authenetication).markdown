---
layout: post
title:  Tinder Clone Coding Part 1(Firebase Authenetication)
date:   2022-04-26 00:35:00 +0900
categories:   Codelab
---

# Component

이번 실습에서 진행할 내용

* __Firebase Authenetication (SNS로 로그인하기)__

* Firebase Realtime Database (서버 데이터 베이스)

* CardStackView(카드 스택 뷰 구현, Opensource)

<br>

# Create Firebase Project, Authenetication, 이메일 사용 설저 ON

<img src="/public/img/snsauth1.png"  width="400" height="300">

<img src="/public/img/snsauth0.png"  width="500" height="200">

 라이브러리를 포함시키기 위해서 아래와 같은 코드들을 추가해야한다.

 Android Studio Arctic Fox 버전 이후로는 조금 달라진 코드 모습을 볼 수 있다.

 ```kotlin
 // setting.gradle
 // 모드를 변경해야함
 dependencyResolutionManagement {
    // FAIL_ON_PROJECT_REPOS -> PREFER_PROJECT
    repositoriesMode.set(RepositoriesMode.PREFER_PROJECT)
    repositories {
        google()
        mavenCentral()
    }
}
 ```

 ```kotlin
 // project level - build.gradle
 buildscript {
    dependencies {
        classpath 'com.google.gms:google-services:4.3.10'
    }
}
 ```

 ```kotlin
plugins {
    // ...
    id 'com.google.gms.google-services'
}

dependencies {
    // 파이어베이스
    implementation platform('com.google.firebase:firebase-bom:29.3.1')

    //파이어 베이스 Auth
    implementation 'com.google.firebase:firebase-auth-ktx'
}
 ```

 <Br>

 # Realtime Database 지역 미국, 테스트 버전으로 생성

 # 데이터 베이스 생성 후 google-service.json 파일을 최신화 버전으로 업데이트 해주기

 # LoginActivity.kt, activity_login.xml 생성

 ```xml
 <?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="24dp">

    <EditText
        android:id="@+id/emailEditText"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <EditText
        android:id="@+id/passwordEditText"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:inputType="textPassword"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/emailEditText" />

    <Button
        android:id="@+id/loginButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="로그인"
        app:layout_constraintEnd_toEndOf="@id/passwordEditText"
        app:layout_constraintTop_toBottomOf="@id/passwordEditText" />

    <Button
        android:id="@+id/signupButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginEnd="4dp"
        android:text="회원가입"
        app:layout_constraintEnd_toStartOf="@id/loginButton"
        app:layout_constraintTop_toBottomOf="@id/passwordEditText" />


</androidx.constraintlayout.widget.ConstraintLayout>
```

```kotlin
// LoginActivity.kt
package com.jyh.snsauth

import android.os.Bundle
import android.util.Log
import android.widget.Button
import android.widget.EditText
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.core.widget.addTextChangedListener
import com.google.firebase.auth.FirebaseAuth
import com.google.firebase.auth.ktx.auth
import com.google.firebase.ktx.Firebase

class LoginActivity:AppCompatActivity() {

    // FirebaseAuth 객체 변수
    lateinit var auth:FirebaseAuth
    lateinit var emailEditText:EditText
    lateinit var passwordEditText:EditText
    lateinit var loginButton:Button
    lateinit var signupButton:Button

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_login)

        auth = Firebase.auth // Firebase.getInstance()

        emailEditText = findViewById<EditText>(R.id.emailEditText)
        passwordEditText = findViewById<EditText>(R.id.passwordEditText)
        initLoginButton()
        initSignupButton()
        initEmailAndPasswordEditText()
    }

    private fun initLoginButton() {
        loginButton = findViewById<Button>(R.id.loginButton)
        loginButton.isEnabled= false
        loginButton.setOnClickListener {
            val email = getInputEmail()
            val password = getInputPassword()

            // 파이어베이스 이메일 로그인 기능
            auth.signInWithEmailAndPassword(email, password)
                .addOnCompleteListener(this){
                    if(it.isSuccessful){
                        finish()
                    }
                    else{
                        Toast.makeText(this, "$email $password",Toast.LENGTH_SHORT).show()
                    }
            }
        }
    }

    private fun initSignupButton() {
        signupButton = findViewById<Button>(R.id.signupButton)
        signupButton.isEnabled = false
        signupButton.setOnClickListener {
            val email = getInputEmail()
            val password = getInputPassword()

            auth.createUserWithEmailAndPassword(email,password)
                .addOnCompleteListener(this) {
                    if(it.isSuccessful){
                        Toast.makeText(this, "회원가입 성공\n 로그인을 해주세요",Toast.LENGTH_SHORT).show()
                    }else{
                        Toast.makeText(this, "회원가입 실패",Toast.LENGTH_SHORT).show()
                    }
                }
        }
    }

    private fun getInputEmail():String{
        return emailEditText.text.toString()
    }

    private fun getInputPassword() :String{
        return passwordEditText.text.toString()
    }

    private fun initEmailAndPasswordEditText(){
        emailEditText.addTextChangedListener {
            val enable = emailEditText.text.isNotEmpty() && passwordEditText.text.isNotEmpty()
            loginButton.isEnabled = enable
            signupButton.isEnabled = enable
            Log.d("tag", "$enable")
        }
        passwordEditText.addTextChangedListener {
            val enable = emailEditText.text.isNotEmpty() && passwordEditText.text.isNotEmpty()
            loginButton.isEnabled = enable
            signupButton.isEnabled = enable
        }
    }
}
```

<br>

# MainActivity에서 onStart() 오버라이딩

회원 정보가 없으면 자동으로 로그인 엑티비티로 이동하기!

```kotlin
// onStart() 메서드 오버라이딩, life cycle 복습하기
    override fun onStart() {
        super.onStart()

        if(auth.currentUser==null){
            startActivity(Intent(this, LoginActivity::class.java))
        }
    }
```

<Br>

# Firebase Auth 콘솔 Facebook 로그인 추가하기

* FaceBook Developer 회원 가입, 앱 추가

Facebook 로그인 기능 설정

<img src="/public/img/snsauth2.png"  width="500" height="300">

<br>

* 로그인 -> 설정 -> 앱 ID 확인, Firebase Auth 콘솔에 Facebook ID 추가

<img src="/public/img/snsauth3.png"  width="600" height="100">

<Br>

* Firebase Auth 콘솔에 보이는 OAuth 리다이렉션 uri를 Facebook Developer 콘솔 앱의 로그인 설정에 추가

<img src="/public/img/snsauth4.png"  width="600" height="250">

<br>

# Android Facebook 공식 문서 1~5 번 진행

[문서]에 따라 1번 부터 5번 까지의 과정을 진행한다.

6번 부터는 릴리즈 용 키를 생성하므로 릴리즈 할 때 문서를 참고해 수행해보자

[문서]: https://developers.facebook.com/docs/facebook-login/android/?locale=ko_KR

<br>

# 페이스북 로그인 버튼 추가하기

```xml
    <com.facebook.login.widget.LoginButton
        android:id="@+id/facebookLoginButton"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="@id/passwordEditText" />
```

<br>

# LoginActivity.kt에 CallbackManager 추가하기(페이스북 로그인 콜백 관리)

현재 onActivityResult() 메소드와 startActivtyForResult() 메소드는 Deprecated 되었습니다

[이전 포스팅 참고]

[이전 포스팅 참고]: https://yonghanju.github.io/codelab/2022/02/19/PermissionRequest.html

```kotlin
//LoginActivity.kt
...

    private val callbackManager: CallbackManager by lazy { CallbackManager.Factory.create() }

...

    initFacebookLoginButton()

...

    private fun initFacebookLoginButton() {
        val facebookLoginButton = findViewById<LoginButton>(R.id.facebookLoginButton)
        facebookLoginButton.setPermissions("email", "public_profile")
        facebookLoginButton.registerCallback(callbackManager, object:FacebookCallback<LoginResult>{

            override fun onSuccess(result: LoginResult) {
                // 로그인 성공
                val credential = FacebookAuthProvider.getCredential(result.accessToken.token)
                auth.signInWithCredential(credential)
                    .addOnCompleteListener(this@LoginActivity) {
                        if(it.isSuccessful){
                            finish()
                        }else{
                            Toast.makeText(this@LoginActivity, "로그인 실패", Toast.LENGTH_SHORT).show()
                        }
                    }
            }

            override fun onCancel() {
                // 로그인 중도 포기
            }

            override fun onError(error: FacebookException) {
                // 로그인 실패, 컨택스트 this -> this@LoginActivity !!!
                Toast.makeText(this@LoginActivity, "로그인 실패", Toast.LENGTH_SHORT).show()
            }

        })
    }

...

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)

        callbackManager.onActivityResult(requestCode, resultCode, data)
    }
```

<br>

앱 검수를 마치면 정상적으로 유저의 아이디를 읽어올 수 있다.

[앱 검수 참고]

[앱 검수 참고]: https://developers.facebook.com/docs/app-review