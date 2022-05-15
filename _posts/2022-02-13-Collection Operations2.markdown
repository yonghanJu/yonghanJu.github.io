---
layout: post
title:  Collections Operations(Filtering)
date:   2022-02-14 17:58:00 +0900
categories:   Android
---

# Collection Operations

<br>

일반적인 컬렉션 연산들은 read-only, mutable 컬렉션 둘다 사용 가능하며 크게 아래와 같이 분류된다.

Transformations, Filtering, plus and minus operators, Grouping, Retrieving collection parts, Retrieving single elements, Ordering, Aggregate operations

오늘은 그중에서 __Filtering__ 에 대해 정리해보았다.

<br>

# Filtering

## filter

<br>

람다 안의 __조건에 맞는 요소__ 만을 담은 컬렉션을 반환

```kotlin
val numbers = listOf("one", "two", "three", "four")  
val longerThan3 = numbers.filter { it.length > 3 }
println(longerThan3)
​
val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
val filteredMap = numbersMap.filter { (key, value) -> key.endsWith("1") && value > 10}
println(filteredMap)

// Output
// [three, four]
// {key11=11}
```

<br>

## filterIndexed

filer 함수에서 인덱스를 사용할 수 있음

```kotlin
val numbers = listOf("one", "two", "three", "four")
​
val filteredIdx = numbers.filterIndexed { index, s -> (index != 0) && (s.length < 5)  }
​
println(filteredIdx)

// Output
// [two, four]
```

<br>

## filterIsInstance

__Any__ 타입의 컬렉션 중에서 특정 타입의 요소만을 컬렉션에 담아 반환


```kotlin
val numbers = listOf(null, 1, "two", 3.0, "four")
numbers.filterIsInstance<String>().forEach {
    println(it.uppercase())
}

// Output
// TWO
// FOUR
```

<br>

## filterNotNull

List<T?> 형의 nullable 컬렉션에 사용하여 __non-null 컬렉션__ (List<T!!>)을 반환한다.

```kotlin
val numbers = listOf(null, "one", "two", null)
numbers.filterNotNull().forEach {
    println(it.length)   // length is unavailable for nullable Strings
}

//Output
// 3
// 3
```

<br>

## Partition

filter 함수와 비슷하지만 __matched 와 unmatched(rest)__ 두 부분으로 컬렉션을 나눈다.

```kotlin
val numbers = listOf("one", "two", "three", "four")
val (match, rest) = numbers.partition { it.length > 3 }
​
println(match)
println(rest)

// Output
// [three, four]
// [one, two]
```

<br>

## Test predicates

컬렉션을 테스트 할 수 있는 3가지 함수들

1. any() : 하나라도 조건에 충족한다면 true

2. none() : 단 하나도 조건에 충족되지 못한다면 true

3. all() : 모든 요소가 조건에 충족된다면 true 

```kotlin
val numbers = listOf("one", "two", "three", "four")

println(numbers.any { it.endsWith("e") })
println(numbers.none { it.endsWith("a") })
println(numbers.all { it.endsWith("e") })
println(emptyList<Int>().all { it > 5 })   // vacuous truth, 빈 컬렉션이므로 통과되는 경우

// Output
// true
// true
// false
// true
```