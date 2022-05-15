---
layout: post
title:  Collections Operations(plus, groupBy)
date:   2022-02-15 19:50:00 +0900
categories:   Android
---

# Collection Operations

<Br>

## Plus and minus operators, Group Elements

## plus(= +), minus(= -)

<br>

컬렉션을 합치거나(합집합), 뺀다(차집합)

```kotlin
val numbers = listOf("one", "two", "three", "four")
​
val plusList = numbers + "five"
val minusList = numbers - listOf("three", "four")
println(plusList)
println(minusList)

// Output
// [one, two, three, four, five]
// [one, two]
```

<br>

## groupBy

__associateBy__ 와 같이LinkedHashMap 을 반환한다. 하지만 __람다식의 result가 key__ 가 되며 같은 키를 갖는 __요소들의 list가 value__ 가 된다.

```kotlin
val numbers = listOf("one", "two", "three", "four", "five")
​
println(numbers.groupBy { it.first().uppercase() })
println(numbers.groupBy(keySelector = { it.first() }, valueTransform = { it.uppercase() }))

// Output
// {O=[one], T=[two, three], F=[four, five]}
// {o=[ONE], t=[TWO, THREE], f=[FOUR, FIVE]}
```

<br>

## groupingBy

앞선 __groupBy__ 와 같은 동작을 하지만 이후 eachCount(), reduce{key, aac:Any, elem ->  }, fold(){}, aggregate() 등 확장 함수를 사용할 수 있다.


```kotlin
val numbers = listOf("one", "two", "three", "four", "five", "six")
println(numbers.groupingBy { it.first() }.eachCount())
println(numbers.groupingBy { it.first() }.reduce{key, acc:String, element -> acc+element})

// Output
// {o=1, t=2, f=2, s=1}
// {o=one, t=twothree, f=fourfive, s=six}
```