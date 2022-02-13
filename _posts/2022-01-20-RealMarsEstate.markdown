---
layout: post
title:  RealMarsEstate Codelab
date:   2022-01-20 13:00:00 +0300
categories:   Codelab
---

2022-01-10

// Glide 사용법, web server에서 사진 가져오기, recylclerView사용하기
# Real Estate Mars App codelab -(1): Getting data from the internet

https://developer.android.com/codelabs/kotlin-android-training-internet-data?index=..%2F..android-kotlin-fundamentals#3

https://github.com/yonghanJu/android-kotlin-fundamentals-starter-apps/tree/master/MarsRealEstate-Starter

앱의 네트워크 계층을 만들기

1. Add Retrofit dependencies to Gradle
2. Add support for Java 8 language features
3. Implement MarsApiService
4. Call the web service in OverviewViewModel
5. Define the internet permission

<br>

오류발생, 해결: https://stackoverflow.com/questions/67039491/some-problems-were-found-with-the-configuration-of-task-appgeneratesafeargsde
!!!!!! gradle 업그레이트 후 android.arch -> androidx 로 namespace를 변경 주의!!!

<br>

## Parse the JSON response with Moshi (json 데이터를 kotlin object로 변환하기)
There's a library called __Moshi__, which is an Android JSON parser that converts a JSON string into Kotlin objects
이전에 사용한 ScalarsConverter 는 String과 promitive type으로 변환해줌

1. Add Moshi library dependencies
2. Implement the MarsProperty data class (Double can be used to represent any JSON number.)
3. Update MarsApiService and OverviewViewModel

<br>

## Use coroutines with Retrofit

콜백 방싱x, 코루틴 사용, __중요__

https://developer.android.com/codelabs/kotlin-android-training-internet-data?index=..%2F..android-kotlin-fundamentals#5

1. Update MarsApiService and OverviewViewModel


2022-01-11
# Real Estate Mars App codelab -(2): Loading and displaying images from the Internet

https://developer.android.com/codelabs/kotlin-android-training-internet-images#2

BindingAdapter.kt를 생성, 아래 코드를 이용해서 ImageView에 Glide를 통해 이미지 로드 작업 설정


``` kotlin
// BindingAdapter.kt

// 데이터가 변경될 때 자동으로 실행되는 코드들
@BindingAdapter("imageUrl") // connected with grid_view_item.xml, 40 line
fun bindImage(imgView: ImageView, imgUrl: String?) {
    imgUrl?.let {
        val imgUri =
            imgUrl.toUri().buildUpon().scheme("https").build()
        Glide.with(imgView.context)
            .load(imgUri)
            .apply(
                RequestOptions()    // 에러, 로딩 등 상태 옵션 사용
                .placeholder(R.drawable.loading_animation) // 로딩 화면
                .error(R.drawable.ic_broken_image)) // 에러 발생시 화면
            .into(imgView) // 이미지 넣을 view
    }
}   
```

## Loading and displaying images from the 
인터넷이 없을 때, 로딩 중일 때 사용자에게 알려주기 

1: Add status to the view model

``` kotlin
// OverviewViewModel.kt
enum class MarsApiStatus { LOADING, ERROR, DONE }
```

<br>

Summary
To simplify the process of managing images, use the Glide library to download, buffer, decode, and cache images in your app.
Glide needs two things to load an image from the internet: the URL of an image, and an ImageView object to put the image in. To specify these options, use the load() and into() methods with Glide.
Binding adapters are extension methods that sit between a view and that view's bound data. Binding adapters provide custom behavior when the data changes, for example, to call Glide to load an image from a URL into an ImageView.
Binding adapters are extension methods annotated with the @BindingAdapter annotation.
To add options to the Glide request, use the apply() method. For example, use apply() with placeholder() to specify a loading drawable, and use apply() with error() to specify an error drawable.
To produce a grid of images, use a RecyclerView with a GridLayoutManager.
To update the list of properties when it changes, use a binding adapter between the RecyclerView and the layout.


<br>

# Filtering and detail views with internet data

<br>

```
data class MarsProperty(
    val id: String,
    @Json(name = "img_src") val imgSrcUrl: String,
    val type: String,
    val price: Double
) { val isRental get() = type == "rent" } // get() 함수 사용법,item.xml 60line 확인
```

<br>

grid_item_view.xml 60 line
```
android:visibility="@{property.rental ? View.GONE : View.VISIBLE}"/> // get()함수 활용, property.rental 변수 생성
```

<br>

property 변수 선언
```
    <data>
        <variable
            name="property"
            type="com.example.android.marsrealestate.network.MarsProperty" />
        <import type="android.view.View"/> // __VIEW.GONE 사용을 위해__
    </data>
```

<br>

OverviewFragment.kt
```
    override fun onOptionsItemSelected(item: MenuItem): Boolean { // 메뉴 선택시 실행 함수 오버라이딩
        viewModel.updateFilter(
            when (item.itemId) {    // R.id로 아이탬 구분
                R.id.show_rent_menu -> MarsApiFilter.SHOW_RENT
                R.id.show_buy_menu -> MarsApiFilter.SHOW_BUY
                else -> MarsApiFilter.SHOW_ALL
            }
        )
        return true
    }
```
overflow_menu.xml

```

<menu xmlns:android="http://schemas.android.com/apk/res/android"> 
   <item
       android:id="@+id/show_all_menu"
       android:title="@string/show_all" />
   <item
       android:id="@+id/show_rent_menu"
       android:title="@string/show_rent" />
   <item
       android:id="@+id/show_buy_menu"
       android:title="@string/show_buy" />
</menu>
```

## Filtering and detail views with internet data (아이탬 클릭시 화면 전환하기, Safe Agrs, RecyclerView with ViewModel, Parcelable(직렬화))

<br>

https://developer.android.com/codelabs/kotlin-android-training-internet-filtering#4

<br>

__중요!!!!!!!! ViewModel과 AndroidViewModel 차이점__

<br>

https://developer.android.com/codelabs/kotlin-android-training-internet-filtering#5

<br>


AndroidViewModel의 경우 Context를 포함, 아래와 같은 상황(String리소스 사용)에서 사용됨

DetailViewModel.kt

```
val displayPropertyPrice = Transformations.map(selectedProperty) { // __Transformations 클래스 이용, LiveData 타입변경__
   app.applicationContext.getString(
           when (it.isRental) {
               true -> R.string.display_price_monthly_rental
               false -> R.string.display_price
           }, it.price)
}
```

이번 코드랩에서 배운것 

요약

https://developer.android.com/codelabs/kotlin-android-training-internet-filtering#7

learn more

https://developer.android.com/codelabs/kotlin-android-training-internet-filtering#8




