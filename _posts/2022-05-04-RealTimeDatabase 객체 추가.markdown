---
layout: post
title:  Realtime Database에 data class 추가하기
date:   2022-05-04 22:00:00 +0900
categories:   Android
---

# data class 생성

<br>

__Firebase RealtimeDatabase__ 에 입력되려면 data class 에 __constructor__ 를 추가해야 한다.

```kotlin
data class ArticleModel (
    val sellerId: String,
    val title: String,
    val createdAt:Long,
    val price: String,
    val imageUrl:String
){
    // constructor 생성
    constructor():this("", "",0,"","")
}
```

<br>

# RealtimeDatabase에 data class 저장

__db.push()__ 함수를 사용해 임의의 키를 생성
__setValue()__ 함수를 통해 data class 객체를 입력


```kotlin
    private val auth by lazy{
        Firebase.auth
    }
    private val articleDB by lazy{
        Firebase.database.reference.child(DB_ARTICLES)
    }

    ...

    private fun initSubmitButton() {
        binding.submitButton.setOnClickListener {
            val title = binding.titleEditText.text.toString()
            val price = binding.priceEditText.text.toString()
            Log.d("initSubmitButton", "$title $price")
            val sellerId = auth.currentUser?.uid.orEmpty()

            var model = ArticleModel(sellerId, title, System.currentTimeMillis(), "$price 원", "")
            // Articles 밑에 임의의 아이탬을 만듬
            articleDB.push()
                .setValue(model) // 임의의 아이탬(key)에 모델이 할당됨
        }
    }
```

<Br>

# 데이터 베이스 추가된 모습

<br>

<img src="/public/img/20220504_1.png"  width="500" height="250">