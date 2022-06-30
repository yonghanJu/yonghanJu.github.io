---
layout: post
title:  uppercase, joinToString() JadenCase 문자열 만들기  - Pro_LV2
date:   2022-07-01 00:22:00 +0900
categories:   Algorithm
---

# JadenCase 문자열 만들기

[JadenCase 문자열 만들기] 이번 문제는 프로그래머스의 레벨 2단계 중에서도 연습문제로 아주 쉬운 문제이다. 

하지만 짧은 코드 안에서도 앞으로 string 배열에 대해 유용하게 쓰일 함수를 소개하고자 포스팅한다.

<br>

# 문제 설명

1. 문자열을 입력받는다.

2. 문자열을 공백을 기준으로 나눈다.

3. 나눠진 문자열의 첫 문자를 대문자로 바꾼다.

4. 다시 하나의 문자열로 합쳐 반환한다

<br>

[JadenCase 문자열 만들기]: https://programmers.co.kr/learn/courses/30/lessons/12951



<br>

# 함수


- str.split(' ')

    delim 인자를 바탕으로 문자열을 나워 배열에 담는 기본적인 함수.

    ex) ```"110011".split(' ') // ["11","","11"]```

<br>

- joinToString(prefix = "", separator = " ", postfix =  "")

    위 ```split()```의 역함수이다.

    즉, ```split()```의 결과를 ```joinToString()``` 한다면 원래 형태로 돌아와 진다.

<br>

- uppercaseChar(), lowercaseChar()

    기존 java에서 부터 사용되던 ```toUpperCase(), toLowerCase()```함수가 __Deprecated__ 되면서 대체 함수로 개발되었다.

    문자의 변화가 없는 상황에서는 NPE을 발생시키지 않고 아무 동작 하지 않는다.

<br>

# 코드

```kotlin
// 2022-07-01
// https://programmers.co.kr/learn/courses/30/lessons/12951 (JadenCase 문자열 만들기)

class Solution {
    fun solution(s: String): String {

        // String 자료형의 변환을 위해 CharArray 로 변경
        var list = s.split(' ').map{it.toCharArray()} 

        list.forEach{ cArr->
            // uppercaseChar() 함수는 소문자 아닐 경우에 변화 X
            // 따라서 숫자, 기호, 대,소문자 확인 필요 X
            if(cArr.size>0) cArr[0] = cArr[0].uppercaseChar()
            for(i in 1 .. cArr.lastIndex){
                cArr[i] = cArr[i].lowercaseChar()
            }
        }

        // 중요!!
        // joinToString(prefix = "", separator = " ", postfix =  "") 함수를 활용해
        // 나눴던 배열을 다시 하나로 병합 (split()의 역함수)
        return list.map{String(it)}.joinToString(" ")
    }
}
```