---
layout: post
title:  Tinder Clone Coding Part 2(Firebase Realtime Database)
date:   2022-04-26 01:30:00 +0900
categories:   Codelab
---

# Component

이번 실습에서 진행할 내용

* Firebase Authenetication (SNS로 로그인하기)

* __Firebase Realtime Database (서버 데이터 베이스)__

* CardStackView(카드 스택 뷰 구현, Opensource)

<br>

# 라이브러리 추가

모듈 레벨 build.gradle에 추가

```kotlin
    //파이어 베이스 데이터베이스
    implementation 'com.google.firebase:firebase-database-ktx'
```

<br>

# LikeActivity, layout 생성 후 MainAtivity에서 라우팅

```kotlin
// MainActivit.kt

    override fun onStart() {
        super.onStart()

        if(auth.currentUser==null){
            startActivity(Intent(this, LoginActivity::class.java))
        }else{
            startActivity(Intent(this, LikeActivity::class.java))
        }
    }
```

<br>

# 기존 LoginActiviy finish() 함수 교체

LoginActiviy.kt에서 로그인 성공 시 호출하는 finish() 함수를 handleSuccessLogin()으로 교체

```kotlin
    private fun handleSuccessLogin(){
        if(auth.currentUser==null){
            Toast.makeText(this, "로그인 실패",Toast.LENGTH_SHORT).show()
            return
        }

        // 데이터베이스에 유저 id 저장
        val userId = auth.currentUser?.uid.orEmpty()
        // Firebase.database.reference = 루트
        val currentUserDB = Firebase.database.reference.child("Users").child(userId)
        val user = mutableMapOf<String, Any>()
        user["userId"] = userId
        currentUserDB.updateChildren(user)

        finish()
    }
```

<br>

# LikeActivity.kt 구현

```kotlin
// LikeActivity.kt
class LikeActivity : AppCompatActivity() {

    private lateinit var auth:FirebaseAuth
    private lateinit var usersDB: DatabaseReference

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_like)

        auth= FirebaseAuth.getInstance()
        usersDB = Firebase.database.reference.child("Users")

        val currentUserDB = usersDB.child(getCurrentUserId())
        currentUserDB.addListenerForSingleValueEvent(object :ValueEventListener{
            override fun onDataChange(snapshot: DataSnapshot) {
                if(snapshot.child("name").value == null){
                    showNameInputPopup()
                    return
                }
                // todo 갱신
            }

            override fun onCancelled(error: DatabaseError) {}

        })


    }

    private fun showNameInputPopup() {
        val editText = EditText(this)

        AlertDialog.Builder(this)
            .setTitle("이름을 입력해주세요")
            .setView(editText)
            .setPositiveButton("저장"){ _,_->
                if(editText.text.isEmpty()){
                    showNameInputPopup()
                }else{
                    saveUserName(editText.text.toString())
                }
            }.setCancelable(false)
            .show()
    }

    private fun saveUserName(name:String) {
        val userId = getCurrentUserId()
        val currentUserDB = usersDB .child(userId)
        val user = mutableMapOf<String, Any>()
        user["userId"] = userId
        user["name"] = name
        currentUserDB.updateChildren(user)
    }

    private fun getCurrentUserId():String{
        if(auth.currentUser==null){
            Toast.makeText(this, "로그인되지 않았습니다.", Toast.LENGTH_SHORT).show()
            finish()
        }
        return auth.currentUser?.uid.orEmpty()
    }

}
```