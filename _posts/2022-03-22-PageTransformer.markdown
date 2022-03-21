---
layout: post
title:  ViewPager2 PageTransformer
date:   2022-03-22 01:05:00 +0900
categories:   Codelab
---

# ViewPager2 

<Br>

[ViewPage2]는 기존 ViewPager과 달리 __수직방향, DiffUtil, PageTransformer__ 기능을 지원해주며 PageTransformer은 페이지 변경 모양을 커스텀할 수 있게 해준다.

이번 코드랩에서는 간단하게 페이지가 넘겨지면 점점 희미해져가면서 사라지도록 만들어보자.


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

[ViewPage2]: https://developer.android.com/reference/androidx/viewpager2/widget/ViewPager2