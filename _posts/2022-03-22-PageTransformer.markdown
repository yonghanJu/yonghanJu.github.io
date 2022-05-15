---
layout: post
title:  ViewPager2 PageTransformer
date:   2022-03-22 01:05:00 +0900
categories:   Android
---

# ViewPager2 

<Br>

[ViewPage2]는 기존 ViewPager과 달리 __수직방향, DiffUtil, PageTransformer__ 기능을 지원한다.

<br>

__DiffUtil__ 은 RecyclerView나 viewPager2 에서 사용되는 아이탬 리스트가 변경될 때 아이탬 리스트 전부를 다시 로드하는게 아니라 변경된 점만 적용시키는 유틸 라이브러리이다.

<br>

그리고 __PageTransformer__ 는 페이지의 변경을 감지하고 실행하는 동작을 설정할 수 있으며 페이지 변경 모양을 커스텀할 수 있게 해준다.

<br>

이번 코드랩에서는 간단하게 페이지가 넘겨지면 점점 희미해져가면서 사라지도록 만들어보자. (초기 시작 페이지 설정도 포함)


<br>

```kotlin
// 2022-03-22
// 이미 구현된 뷰 페이지에 PageTransformer 효과 추가하기
// 페이지에 변경이 감지되면 실행
viewPager.setPageTransformer{ page, position ->
    when{
        // 현재 위치를 0으로 삼고 오른쪽은 1,2,3... 왼쪽은 -1,-2,-3... 의 포지션을 갖는다
        // 포지션의 절대값이 1을 넘기면(현재 위치기 아니면) 감추기
        position.absoluteValue >= 1F -> page.alpha = 0F
        
        // 현재 페이지는 선명하게
        position.absoluteValue =- 0F -> page.alpha = 1F 

        //움직이는 동안에는 점차 흐려지게
        else ->{
            page.alpha = 1F - 2*position.absoluteValue
        }
    }
}

```

<br>
<br>

# 초기 시작 페이지 설정법

<br>

__ViewPager2__ 객체에 __currentItem__ 을 변경함으로 현재 페이지를 바꿀수 있다.

<br>


```kotlin
viewPager.currentItem = 3
``` 

<Br>

좌우로 __무한 스크롤__ 지원을 위해서는 아이탬의 갯수를 Int.MAX_VALUE 로 설정하고 시작 위치를 그 중간값으로 해야한다.

하지만 위와 같은 방식으로는 초기 시작 페이지를 설정이 불가하다.

<br>

위 예제처럼 코드를 작성하면 자동으로 __ViewPager__ 객체에서 

``setCurrentItem(position :Int, smoothScroll :Boolean)``

함수가 실행되고 __smoothScroll__ 기본 값이 true기 때문에 뷰페이져가 직접 움직여서 초기값으로 이동된다.


<br>


따라서 우리는 __currentItem__ 값을 바로 변경해주는게 아닌 ``setCurrentItem()`` 함수를 직접 호출해야 한다.

<br>

```kotlin
viewPager.setCurrentItem(3, false)
```







[ViewPage2]: https://developer.android.com/reference/androidx/viewpager2/widget/ViewPager2