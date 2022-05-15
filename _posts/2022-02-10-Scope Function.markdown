---
layout: post
title:  Scope Function
date:   2022-2-10 21:06:00 +0300
categories:   Android
---

# Scope Function(Kotlin)

<br>

스코프 함수(범위 함수)는 해당 객체의 맴버들을 참조 할 수 있다.

종류: apply, with, let, also, run

<br>

### 예시

<br>

1. apply


보통의 경우 apply 함수는 객체의 초기화나 설정에 쓰이며 __해당 객체를 반환.__

```kotlin
// Example
val person = Person().apply{
    firstName = "Ju"
    lastName = "YongHan"
}
```

<br>

2. also

위 apply와 비슷하게 사용되며 차이점은 T가 파라미터it을 통해 전달된다는 점이다. 마찬가지로 __객체를 반환.__

```kotlin
// Example_1
val person = Person().apply{
    it.firstName = "Ju"
    it.lastName = "YongHan"
}

// Example_2
// 생성한 객체를 이용해 print함수 실행, apply와 같이 it 반환
Random.nextInt(100).also{ value-> 
print( " random int= $value.")
}
```

<br>

3. let

apply, also 두 함수와 달리 객체 자체를 반환하지 않고 let의 __실행값을 반환한다.__

또한 null safe 코드 작성을 위해 자주 사용 됨.

```kotlin
// Example
val number: Int?

// non-null type
val numberString: String!! = number?.let{
    "I am String $number."
}.orEmpty() 
```

<br>

4. with

파라미터로 객체를 받으며 객체의 맴버에 접근 가능, 또한 람다식의 __결과를 반환한다.__

```kotlin
// Example
val person = Person()

// 확장 함수로 사용 불가! (person.with() X)
val firstNameStr = with(person){
    work()
    sleep()
    print(age)
    firstName
}
```

<br>

5. run

객체를 활용한 계산이 필요할 때 사용됨. __결과를 반환 함.__

```kotlin
// Example 
// result = service.query() 반환
val result = service.run{
    port = 8080
    query()
}

```
