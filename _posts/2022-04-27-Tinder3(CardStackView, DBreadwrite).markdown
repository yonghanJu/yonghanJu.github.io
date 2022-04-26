---
layout: post
title:  Tinder Clone Coding Part 3(Realtime DB read and write, CardStackView)
date:   2022-04-27 01:33:00 +0900
categories:   Codelab
---

# Component

이번 실습에서 진행할 내용

* Firebase Authenetication (SNS로 로그인하기)

* Firebase Realtime Database (서버 데이터 베이스)

* __CardStackView(카드 스택 뷰 구현, Opensource)__

<br>

# 라이브러리 추가

CardStackView [오픈소스 주소]를 확인하고 라이브러리 추가

[오픈소스 주소]: https://github.com/yuyakaido/CardStackView

__(예전 방법)__
```kotlin
    // app level build.gradle
    // CardStackView 오픈소스
    implementation 'com.yuyakaido.android:card-stack-view:2.3.4'

    // 현재 com.github.yuyakaido:cardstackview:2.3.4
```

<br>

__현재__

[변경된 사용 법]

[변경된 사용 법]: https://jitpack.io/p/yuyakaido/cardstackview

settings.gradle 파일에  ``maven { url 'https://jitpack.io'}`` 를 추가 해주고 디펜던시 (``'com.github.yuyakaido:cardstackview:2.3.4'``) 추가


```gradle
// settings.gradle
import org.gradle.api.initialization.resolve.RepositoriesMode

pluginManagement {
    repositories {
        gradlePluginPortal()
        google()
        mavenCentral()

        // 추가
        maven { url 'https://jitpack.io'}
    }
}
dependencyResolutionManagement {
    // FAIL_ON_PROJECT_REPOS -> PREFER_PROJECT
    repositoriesMode.set(RepositoriesMode.PREFER_PROJECT)
    repositories {
        google()
        mavenCentral()
        
        //추가
        maven { url 'https://jitpack.io'}
    }
}
rootProject.name = "SNSAuth"
include ':app'

```

<br>

# 카드 모델 생성

```kotlin
// CardItem.kt 
data class CardItem (
    val userId: String,
    val name: String
)
```

<br>

# activity_like.xml , card_item.xml구현

```xml
<!--activity_like.xml-->
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".LikeActivity">

    <com.yuyakaido.android.cardstackview.CardStackView
        android:id="@+id/carStackView"
        android:layout_width="match_parent"
        android:layout_height="300dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

<br>

```xml
<!--card_item.xml-->
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".LikeActivity">

    <com.yuyakaido.android.cardstackview.CardStackView
        android:id="@+id/carStackView"
        android:layout_width="match_parent"
        android:layout_height="300dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

<Br>

# CardStackView Adapter 생성

```kotlin
// CardItemAdapter.kt
// 직접 
class CardItemAdapter: ListAdapter<CardItem, CardItemAdapter.ViewHolder>(diffUtil) {

    inner class ViewHolder(private val binding: ItemCardBinding): RecyclerView.ViewHolder(binding.root) {
        fun bind(cardItem: CardItem){
            binding.nameTextView.text = cardItem.name
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        return ViewHolder(ItemCardBinding.inflate(LayoutInflater.from(parent.context)))
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.bind(currentList[position])
    }

    companion object{
        val diffUtil = object : DiffUtil.ItemCallback<CardItem>(){
            override fun areItemsTheSame(oldItem: CardItem, newItem: CardItem): Boolean {
                return oldItem.userId==newItem.userId
            }

            override fun areContentsTheSame(oldItem: CardItem, newItem: CardItem): Boolean {
                return oldItem==newItem
            }

        }
    }
}
```

<Br>

# LikeActivity.kt 카드 스택뷰 어뎁터 구현

```kotlin
// 직접 CardStackListener 를 구현함
class LikeActivity : AppCompatActivity(), CardStackListener {

    private lateinit var auth:FirebaseAuth
    private lateinit var usersDB: DatabaseReference
    private val adapter = CardItemAdapter()
    private val cardItems = mutableListOf<CardItem>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_like)

        initDB()
        initCardStackView()
    }

    private fun initDB() {
        //...
    }

    private fun initCardStackView() {
        val stackView = findViewById<CardStackView>(R.id.carStackView)

        // 첫번째 인자로 context, 두번째 인자로 CardStackListener 을 받는다
        // 리스너가 구현해야하는 함수가 너무 많아 현재 엑티비티 자체를 상속시켜서 this 를 받는다
        stackView.layoutManager = CardStackLayoutManager(this, this)
        stackView.adapter = adapter
    }

    // 스와이프 하나만 구현
    override fun onCardSwiped(direction: Direction?) {

    }

    override fun onCardDragging(direction: Direction?, ratio: Float) {}

    override fun onCardRewound() {}

    override fun onCardCanceled() {}

    override fun onCardAppeared(view: View?, position: Int) {}

    override fun onCardDisappeared(view: View?, position: Int) { }

}
```

<br>

# 기존 initDB()에서 데이터 가져오는 부분 구현

__initDB()__ 를 통해 최초 1회만 데이터 가져오고 __getUnSelectedUsers()__ 함수를 실행 (saveUserName() 함수  뒤에도 추가)


```kotlin
    private fun initDB() {
        auth= FirebaseAuth.getInstance()
        usersDB = Firebase.database.reference.child("Users")
        val currentUserDB = usersDB.child(getCurrentUserId())

        // 데이터를 1회 받아오고 onDataChange 실행
        // 현재 사용 유저에서 스냅샷 가져오기
        currentUserDB.addListenerForSingleValueEvent(object :ValueEventListener{
            override fun onDataChange(snapshot: DataSnapshot) {
                if(snapshot.child("name").value == null){
                    showNameInputPopup()
                    return
                }
                // 추가
                getUnSelectedUsers()
            }
            override fun onCancelled(error: DatabaseError) {} 
        })
    }

    // 좋아요, 싫어요가 없는 데이터 가져오기
    private fun getUnSelectedUsers() {
        // db 전부 가져와서 작업하기
        usersDB.addChildEventListener(object: ChildEventListener{
            override fun onChildAdded(snapshot: DataSnapshot, previousChildName: String?) {
                if(snapshot.child("userId").value != getCurrentUserId() // 본인이 아니면서
                    && snapshot.child("likedBy").child("like").hasChild(getCurrentUserId()).not()
                    && snapshot.child("likedBy").child("disLike").hasChild(getCurrentUserId()).not()){

                    val userId = snapshot.child("userId").value.toString()
                    var name = "undecided"
                    if(snapshot.child("name") != null){
                        name = snapshot.child("name").value.toString()
                    }

                    cardItems.add(CardItem(userId, name))
                    adapter.submitList(cardItems)
                    adapter.notifyDataSetChanged()
                }
            }

            override fun onChildChanged(snapshot: DataSnapshot, previousChildName: String?) {
                cardItems.find { it.userId == snapshot.key }?.let {
                    it.name =snapshot.child("name").value.toString()
                }

                adapter.submitList(cardItems)
                adapter.notifyDataSetChanged()
            }

            override fun onChildRemoved(snapshot: DataSnapshot) {}

            override fun onChildMoved(snapshot: DataSnapshot, previousChildName: String?) {}

            override fun onCancelled(error: DatabaseError) {}

        })
    }
```

<Br>

# 스와이프 기능 구현하기(like, dislike)

```kotlin
    private fun like(){
        val card = cardItems[manager.topPosition-1]
        usersDB.child(card.userId)      // 상대방 사용자의 DB 접근
            .child("likedBy")
            .child("like")
            .child(getCurrentUserId())  // 현재 사용 id
            .setValue(true)             // value 설정, 위의 id는 key 값이 됨
    }

    private fun disLike(){
        val card = cardItems[manager.topPosition-1]
        usersDB.child(card.userId)      // 상대방 사용자의 DB 접든
            .child("likedBy")
            .child("disLike")
            .child(getCurrentUserId())  // 현재 사용 id
            .setValue(true)             // value 설정, 위의 id는 key 값이 됨
    }


    override fun onCardSwiped(direction: Direction?) {
        when(direction){
            Direction.Left->{ disLike() }
            Direction.Right->{ like() }
            else->{}
        }
    }
```

<br>

# 서로 like 선택시 match 정보 추가

```kotlin

    private fun like(){
        val card = cardItems[manager.topPosition-1]
        //...
        // 추가하기
        // 내가 좋아요를 누른 시점에서
        // 상대방이 날 좋아요 했다면 바로 match
        saveMatchedIfOtherLikedMe(card.userId)
    }


    private fun saveMatchedIfOtherLikedMe(otherId:String) {
        val otherUserDB = usersDB.child(getCurrentUserId()).child("likedBy").child("liked").child(otherId)
        otherUserDB.addListenerForSingleValueEvent(object :ValueEventListener{
            override fun onDataChange(snapshot: DataSnapshot) {
                if(snapshot.value==true){
                    usersDB.child(getCurrentUserId())
                        .child("likedBy")
                        .child("match")
                        .child(otherId)
                        .setValue(true)
                    usersDB.child(otherId)
                        .child("likedBy")
                        .child("match")
                        .child(getCurrentUserId())
                        .setValue(true)
                }
            }

            override fun onCancelled(error: DatabaseError) {
            }

        })
    }
```

<br>

# MatchedAdapter.kt, activity_matched.xml, item_matched_user.xml 생성하기

```xml
<!--activity_matched.xml-->
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MatchedActivity">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/matchedUserRecyclerView"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
</androidx.constraintlayout.widget.ConstraintLayout>


<!--item_matched_user.xml-->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="10dp">

    <TextView
        android:id="@+id/userNameTextView"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="10dp" />
</LinearLayout>
```

<br>

```kotlin
// MatchedAdapter.kt
class MatchedUserAdapter: ListAdapter<CardItem, MatchedUserAdapter.ViewHolder>(diffUtil) {


    inner class ViewHolder(private val binding: ItemMatchedUserBinding): RecyclerView.ViewHolder(binding.root) {
        fun bind(cardItem: CardItem){
            binding.userNameTextView.text = cardItem.name
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        return ViewHolder(ItemMatchedUserBinding.inflate(LayoutInflater.from(parent.context)))
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.bind(currentList[position])
    }

    companion object{
        val diffUtil = object : DiffUtil.ItemCallback<CardItem>(){
            override fun areItemsTheSame(oldItem: CardItem, newItem: CardItem): Boolean {
                return oldItem.userId==newItem.userId
            }

            override fun areContentsTheSame(oldItem: CardItem, newItem: CardItem): Boolean {
                return oldItem==newItem
            }

        }
    }
}
```

<Br>

# MatchedActivity 생성  

```kotlin
// MatchedActivity.kt
class MatchedActivity : AppCompatActivity() {

    private lateinit var auth:FirebaseAuth
    private lateinit var usersDB: DatabaseReference
    private val adapter = MatchedUserAdapter()
    private val cardItems = mutableListOf<CardItem>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_matched)

        initDB() // 디비 초기화
        getMatchedUsers()   // 매칭 유저 정보 담기
        initMatchedUserRecyclerView()  // 리사이클러뷰 초기화
    }

    private fun getMatchedUsers(){
        val matchedUserDB = usersDB
            .child(getCurrentUserId())
            .child("likedBy")
            .child("match")

        // 모든 데이터에 실행
        matchedUserDB.addChildEventListener(object :ChildEventListener{
            override fun onChildAdded(snapshot: DataSnapshot, previousChildName: String?) {
                getUserByKey(snapshot.key.orEmpty())
            }

            override fun onChildChanged(snapshot: DataSnapshot, previousChildName: String?) { }

            override fun onChildRemoved(snapshot: DataSnapshot) {}

            override fun onChildMoved(snapshot: DataSnapshot, previousChildName: String?) { }

            override fun onCancelled(error: DatabaseError) { }
        })
    }

    private fun getUserByKey(userId:String) {
        // 1회 실행
        usersDB.child(userId).addListenerForSingleValueEvent(object :ValueEventListener{
            override fun onDataChange(snapshot: DataSnapshot) {
                cardItems.add(CardItem(userId, snapshot.child("name").value.toString()))
                adapter.submitList(cardItems)
            }
            override fun onCancelled(error: DatabaseError) { }
        })
    }

    private fun initMatchedUserRecyclerView() {
        val matchedUserRecyclerView = findViewById<RecyclerView>(R.id.matchedUserRecyclerView)
        matchedUserRecyclerView.adapter = adapter
        matchedUserRecyclerView.layoutManager = LinearLayoutManager(this,)
    }

    private fun initDB() {
        auth= FirebaseAuth.getInstance()
        usersDB = Firebase.database.reference.child("Users")
    }

    private fun getCurrentUserId(): String {
        return auth.currentUser?.uid.orEmpty()
    }
}
```

<br>

# 매칭 엑티비티 이동 버튼, 로그아웃 버튼 생성하기

```xml
<!--activity_like.xml 버튼 추가-->

    <Button
        android:id="@+id/matchListButton"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="매치 리스트 보기"
        app:layout_constraintBottom_toTopOf="@id/logoutButton"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent" />

    <Button
        android:id="@+id/logoutButton"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="로그아웃"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent" />
```

<br>

```kotlin
// LikeActivity.kt
    private fun initButtons() {
        val logoutButton = findViewById<Button>(R.id.loginButton)
        val matchListButton = findViewById<Button>(R.id.matchListButton)

        logoutButton.setOnClickListener {
            // 로구아웃 함수
            auth.signOut()
            startActivity(Intent(this,MainActivity::class.java))
            finish() // 현재 화면은 종료
        }

        matchListButton.setOnClickListener {
            startActivity(Intent(this, MatchedActivity::class.java))
        }
    }

// MainActivity.kt에 onStart() 수정(로그인 시 메인 엑티비티 닫기)
    override fun onStart() {
        super.onStart()

        if(auth.currentUser==null){
            startActivity(Intent(this, LoginActivity::class.java))
        }else{
            startActivity(Intent(this, LikeActivity::class.java))
            // 추가
            finish()
        }
    }
```