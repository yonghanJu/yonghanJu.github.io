---
layout: post
title:  ReComposition Counts 줄이고 렌더링 성능 향상시키기
date:   2023-11-20 02:02:00 +0900
categories:   Android
---

## Recomposition이란?

Recomposition을 이해하기 위해서는 우선 Compose에서 UI를 그리는 방법에 대해 알아야 한다.

- Composition: Composition 트리를 만든다, 하나의 컴포저블 함수는 트리 안 하나의 노드가 된다.
- Layout: 크기와 배치를 결정한다, 자식이 있다면 먼저 측정하고 결과적으로 자신의 크기와 자식들의 배치를 결정한다.
- Draw: Layout 된 정보를 바탕으로 UI를 렌더링한다.

<br>

위 3가지 단계중 컴포저블 함수에 상태 값이 변경된다면 다시 Composition 단계로 돌아가 UI를 그리기 시작하는데 이것을 Recomposition이라고 부른다.

> 매우 중요한 사실은 위 3가지 단계는 필요에 따라 생략될 수 있다는 것이다. 만약 상태 변화가 없다면 Composition Tree를 다시 그리지 않고 Relayout 만 하면 된다. 

따라서 __자주 변경되는 상태를 반영할 때 Lambda Modifier 사용을 선호__ 해야한다. 람다를 사용함으로 컴포지션 단계가 생략될 수 있게 해주기 때문이다.


---

<br>

## Lambda Modifier를 사용해야하는 이유?

컴포지션 트리는 컴포저블에 적용된 Modifier에 의해서도 그려지게 되는데 이때 Modifier는 효율적으로 불변하는 객체이다.
Modifier에 값이 변경되었다면 기존 Modifier는 제거되고 새로운 Modifier가 만들어 추가되게 된다.

Animation, Rotation UI 를 구현하기 위해 __offset__ 또는 __TranslationX, Y__ 값을 변경시겼다면 매 순간 Modifier는 제거되고 다시 생성되며 컴포지션 트리를 그리게 된다.

하지만 이 상황에서 전적으로 __Recompistion은 필요가 없고 Relayout__ 만 실행되면 되기 때문에 불필요한 Composition 단계를 생략할 수 있다. 공식 문서에 [__Drag, swipe, and fling Guides__] 코드로 예시를 들어보자.

[__Drag, swipe, and fling Guides__]: https://developer.android.com/jetpack/compose/touch-input/pointer-input/drag-swipe-fling


---

<br>

## 실제 상황 및 코드 예시

아래 영상은과 코드는 [__Drag, swipe, and fling Guides__] 공식 문서에서 드래그를 구현하는 모습이다.

<img src="https://github.com/yonghanJu/Algorithm/assets/65655825/17112765-5a4a-45fc-b763-9838d51c4442" width="300" height="650">

아래 두 코드는 위 영상과 같이 정확히 같은 동작을 한다.

```kotlin
@Composable
private fun DraggableTextLowLevel() = with(LocalDensity.current) {
    Box(modifier = Modifier.fillMaxSize()) {
        var offsetX by remember { mutableStateOf(0f) }
        var offsetY by remember { mutableStateOf(0f) }

        val offset = IntOffset(offsetX.roundToInt(), offsetY.roundToInt())

        Box(
            Modifier
                .offset(offset.x.toDp(), offset.y.toDp()) // 직접 값을 넣어서 사용
                // ...
        ) {
            // ..
        }
    }
}
```

```kotlin
@Composable
private fun DraggableTextLowLevel() {
    Box(modifier = Modifier.fillMaxSize()) {
        var offsetX by remember { mutableStateOf(0f) }
        var offsetY by remember { mutableStateOf(0f) }
        
        Box(
            Modifier
                .offset { IntOffset(offsetX.roundToInt(), offsetY.roundToInt()) } // 람다를 사용
                // ...
        ) {
            // ..
        }
    }
}
```

<br>

하지만 두 코드의 컴포지션 횟수의 차이는 극명하다.

첫번째 사진은 람다를 사용하지 않고 직접 __offset__ 값을 넣었을 때의 __Recomposition Counts__ 값이다.
드래그가 행해질 때 마다 __offset__ 상태가 변하고 그 상태를 직접 Modifier에 넣어 변경시켰기 때문에 모든 순간에 Recompositoion이 발생하며 컴포지션 트리를 그리게 된다.

이는 성능적인 면에서 매우 불리할 수 있다.

<img width="503" alt="스크린샷 2023-11-20 오전 1 38 05" src="https://github.com/yonghanJu/Algorithm/assets/65655825/26f776a7-b287-4c12-ba45-25aa06e0b366">

---

<br>


하지만 아래 사진은 직접 __offset__ 을 넣는 대신 __offset을 반환하는 람다__ 를 사용했을 때 __Recomposition Counts__ 값으로 __0번이다(최초 1회 제외).__

__Modifier.offset(offset: Density.() -> IntOffset)__ 함수의 매개변수로 넣어준 람다값 자체에 변경사항이 없기 때문에 x, y 좌표가 바뀌어도 리컴포지션 하지 않는 것이다.


<img width="503" alt="스크린샷 2023-11-20 오전 1 45 47" src="https://github.com/yonghanJu/Algorithm/assets/65655825/a925c201-108a-442b-a9a8-c338edd01434">

---

### 결론

자주 변경하는 상태 값에 대해 매개변수로 직접 넣어주기 보다는 람다를 활용해 Composition 단계를 최대한 줄이는 방향으로 코드를 작성해야 성능 저해를 막을 수 있다.

특이한 점은 직접 __offsetX__ , __offsetY__ 객체를 사용하지 않고 변수에 할당만 하더라도 ```// ex) val o = offsetX``` 컴포지션 횟수는 이전 처럼 증가한다.

이 이유는 직관적으로 이해가 되지 않아서 더 공부해 볼 필요가 있다.