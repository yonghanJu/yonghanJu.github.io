---
layout: post
title:  Collections Operations(Transformations)
date:   2022-02-13 22:50:00 +0900
categories:   Android
---

# Collection Operations

<br>

일반적인 컬렉션 연산들은 read-only, mutable 컬렉션 둘다 사용 가능하며 크게 아래와 같이 분류된다.

Transformations, Filtering, plus and minus operators, Grouping, Retrieving collection parts, Retrieving single elements, Ordering, Aggregate operations

오늘은 그중에서 __Transformations__ 에 대해 정리해보았다.

## Map

<br>

리스트의 형태를 변환

```kotlin
val numbers = listOf("one", "two", "three", "four")
​
println(numbers.associateBy { it.first().uppercaseChar() })
println(numbers.associateBy(keySelector = { it.first().uppercaseChar() }, valueTransform = { it.length }))
println(numbers.associateBy(keySelector = { it.first().uppercaseChar() }, valueTransform = { it.length })::class.simpleName)

// Output
// {O=one, T=three, F=four}
// {O=3, T=5, F=4}
// LinkedHashMap
```

<br>

## Zip

두 리스트를 Pair로 묶어서 활용 가능

```kotlin
val colors = listOf("red", "brown", "grey")
val animals = listOf("fox", "bear", "wolf")

println(colors zip animals)​
println(colors.zip(animals) { color, animal -> "The ${animal.replaceFirstChar { it.uppercase() }} is $color"})

// Output
// [(red, fox), (brown, bear), (grey, wolf)]
// [The Fox is red, The Bear is brown, The Wolf is grey]
```

<br>

## Associate

리스트를 활용해 __LinkedHashMap__ 을 생성함.

1. associateWith() : 리스트의 __요소를 Key로__ 채택

```kotlin
val numbers = listOf("one", "two", "three", "four")
println(numbers.associateWith { it.length })

// Output
// {one=3, two=3, three=5, four=4}
```

<br>

2. associateBy() : 기본으로 리스트의 __요소를 value로__ 채택, __Key 와 Value를 둘 다 설정 가능.__ (중요)

```kotlin
val numbers = listOf("one", "two", "three", "four")
​
println(numbers.associateBy { it.first().uppercaseChar() })
println(numbers.associateBy(keySelector = { it.first().uppercaseChar() }, valueTransform = { it.length }))

/// Output 
// {O=one, T=three, F=four}
// {O=3, T=5, F=4}
```

## Flatten

중첩 컬렉션을 풀어 준다.

```kotlin
val numberSets = listOf(setOf(1, 2, 3), setOf(4, 5, 6), setOf(1, 2))
println(numberSets.flatten())

// Output
// [1, 2, 3, 4, 5, 6, 1, 2]
```