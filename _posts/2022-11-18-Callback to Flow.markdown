---
layout: post
title:  부스트캠프/Callback 기반 API를 단방향 데이터 흐름으로 Part.1
date:   2022-11-18 02:50:00 +0900
categories:   Competition
---

## Callback 기반 API를 단방향 데이터 흐름으로 바꾸기

이번 포스팅은 부스트 캠프 그룹 프로젝트 중 개인 목표로 잡았던
> Callback 기반 API를 Flow 또는 suspend 함수로 만들기
> 단방향 데이터 흐름 만들기

에 대한 회고 및 정리이다.

단방향 데이터 흐름과 함께 엄격한 관심사 분리를 위해 Callback 기반 API들을 Flow 또는 suspend 함수로 변환하는 과정에서 

문제 상황, 사고의 흐름, 해결 방법과 본질적으로 코루틴에 대해서도 느낀바가 담겨있다.

<br>

## Callback을 Flow, suspend funtion으로 바꿔야하는 이유?

오랜 기간동안 비동기 작업에 대한 순서 제어를 위해서 `Callback`을 사용해왔다.

안드로이드에서는 `setOnClickListener()` 와 같은 콜백의 예시를 쉽게 찾을 수 있다.

언제 클릭 이벤트가 발생할지 모르지만, 클리이 발생한다면 어떤 코드를 실행할지 콜백을 통해 정의할 수 있다.

<br>

이처럼 쉽게 비동기 작업을 도와주는 `Callback`에는 몇 가지 __단점__ 이 존재한다.

1. 코드의 쓰여진 순서대로 실행되지 않는다.
2. 여러 비동기 작업의 순서를 보장해야할 때 너무 깊은 깊이의 코드가 작성된다(콜벡 지옥).
3. 가독성이 떨어진다.
4. 데이터의 흐름을 알기 어려워 테스트가 힘들다.
5. 원하지 않는 순간에 콜백을 실행되면 에러를 발생시킬 수 있다.

<br>

만약 아래와 같이 버튼이 눌렸을 때 특정 비동기 작업 이후 결과를 받아 텍스트를 바꿔주는 코드가 있다고 하자.

```kotlin
// TestFragment.kt (onDestroy() 에서 _binding = null)
binding.button.setOnClickListener {

    // 3초 후 output을 받아서 작업
    asyncCallback { output ->
        binding.textView.text = output
    }
}
```

<br>

위 코드로 작성된 앱에서 버튼을 누르고 3초를 기다린다면 정상적으로 텍스트를 바꿔줄 것이다.

<br>

하지만 버튼을 누르고 3초가 지나기 전에 __`TestFragment` 를 지우고 다른 화면으로 이동한다면 어떻게 될까??__

<br>

3초가 지난 뒤 콜백으로 넘겨준 코드 블럭을 실행시키지만 이미 `TestFragment` 는 `onDestroy()` 가 호출된 상태이며 에러가 발생해 앱이 죽게 된다.

물론 콜백으로 넘겨주는 코드블럭 안에서 엑티비티가 살아있는 상태인지 확인해주거나 `onDestroy()` 안에서 콜백 동작을 실행시키지 않도록 만들면 간단하게 에러를 없엘 수 있지만 __몇 가지 명확한 단점이 존재한다.__

1. 해당 콜백을 사용하는 모든 프로그래머가 상황에 맞는 예외처리를 별도로 해줘야 한다.

2. Crash 발생은 막더라도 콜백은 실행되며 사용되지 않는 값에 대해 __메모리 누수__ 가 발생한다.

3. boilerplat code가 지속적으로 발생

<br>
<br>

위와 같은 문제들을 해결하기 위해서 Flow, suspend function 등의 도움을 받아 __반응형 프로그래밍__ 코드를 작성할 수 있다.

> 비동기적인 데이트 스트림을 관찰하고 반응하는 코드를 작성

<br>

기존에 콜백 기반 API를 `Flow`를 반환한다면 데이터 스트림을 만들어 반응형 프로그래밍 작성을 돕고 단방향 데이터 흐름으로 테스트하기 편한 구조가 만들어진다.


또 `repeatOnLifecycle()` 같은 함수를 활용해 `Coroutine Job`을 관리한다면 메모리도 쉽게 관리할 수 있다.

<br>

## 프로젝트 내 문제 상황

다음 화면은 __Firebase FireStore__ 를 통해 실시간으로 `현재 달리는 사람 수` 를 가져와 업데이트하는 기능의 모듈 구상 모습이다.

<img src="https://user-images.githubusercontent.com/65655825/202519550-bbcefea8-ea96-4202-9796-b6ea289e993b.png" alt="drawing" width="400"/>

그룹 프로젝트 앱 설계를 하면서 다음과 같은 흐름으로 문제 상황을 확인느겼다.

1. 그룹 프로젝트 팀원은 전원 안드로이드 개발자로 백엔드 개발자가 없기 때문에 별도의 서버를 구축하지 않고 `Firebase FireStore` 를 사용하기로 했다.

2. 관심사를 분리하고(ui, domain, data) 단방향 데이터 흐름을 만들기 위해서 `Data Layer` 모듈에서만 `Firebase FireStore` 를 참조한다.

3. `Firebase FireStore`에서 실시간 데이터 흐름 감지를 위해 제공하는 API는 모두 `Callback` 기반이다.

4. 단방향 데이터 흐름을 만들어 반응형 프로그래밍을 작성할 수 없어 가독성이 안좋고 테스트와 예외 처리, 분기 처리가 힘들다.

<br>

위 문제를 해결하기 위해 데이터 레이어 모듈에서 발생하는 데이터의 흐름을 콜백 기반 API에서 `Flow`, `suspend function` 로 바꿔보자는 목표 설정을 했다.

<br><br>

## 해결 방법

한번만 실행되는 콜백의 경우 `Suspend Function`으로 만들어 값을 반환한다!

- __suspendCoroutine__

```kotlin
suspend fun getUser(): User {
    return suspendCoroutine { continuation ->
        getUserCallback(uid)
            .OnSuccess { 
                continuation.resume(fakeUser)
            }
    }
}
```

<br>

간단하게 위 방법으로 콜백 기반 API를 suspend function 으로 바꿔줄 수 있다.

하지만 위 방법으로 코루틴을 생성하면 Job을 취소할 수 없어 메모리 누수가 발생할 수 있다.

메모리 누수 방지를 위해 사용할 수 있는 또 다른 방법은 `suspendCancellableCoroutine`을 사용하는 것이다.

<br>

- __suspendCancellableCoroutine__

```kotlin
suspend fun getUser(): User {
    return suspendCancellableCoroutine { continuation ->
        getUserCallback(uid)
            .OnSuccess { 
                continuation.resume(fakeUser)
            }

        continuation.invokeOnCancellation {
            // 리소스 제거(unRegister Callback)
        }
    }
}
```

사용법에서는 크게 다른점은 없고 언제 취소 될지 모르는 작업에 대해
`continuation.invokeOnCancellation` 함수 안에 리소스 제거 코드를 작성해주면 해당 코드가 완료될 때 까지 `suspend`를 유지한다.

<br>

## 궁금한 점 1

___```suspendCoroutine``` 는뭘까?___

우선 ```suspend``` 함수의 역할은 뭘까??

```suspend``` 키워드가 붙은 함수는 현재 코루틴을 정지시키는 역할을 하고 따라서 코루틴 안에서 실행되어야 한다.

그렇다면 ```suspend function```은 실질적으로 언제 코루틴을 중단시키는가??

```suspend function``` 내부에 또 다른 ```suspend function```이 실행될 때 코루틴을 중단시킨다.

그렇다면 실제로 어떤 함수가 코루틴을 중단시킬까?

<br>

```await()```, ```delay()``` 와 같은 함수들은 중단 함수로 코루틴 안에서만 사용할 수 있는데 이 함수들의 구현부를 찾아보았다.

<br>

<img src="https://user-images.githubusercontent.com/65655825/202514317-2dfc6bdf-bdd8-4fdc-bb28-2b590cf50a05.png" alt="drawing" width="1000"/>


위 사진과 같이 중단 함수들은 ```suspendCoroutine``` 또는 ```suspendCancellableCoroutine``` 를 반환함을 확인했고 
```suspendCoroutine``` 함수는 실질적으로 현재 실행 중인 코루틴을 중단시키는 역할을 수행했다.

<br>

그렇다면 ```suspendCoroutine``` 함수를 직접 살펴보자.

아래는 실제 ```suspendCoroutine``` 함수의 구현부이다.

```kotlin
public suspend inline fun <T> suspendCoroutine(crossinline block: (Continuation<T>) -> Unit): T {
    contract { callsInPlace(block, InvocationKind.EXACTLY_ONCE) }
    return suspendCoroutineUninterceptedOrReturn { c: Continuation<T> ->
        val safe = SafeContinuation(c.intercepted())
        block(safe)
        safe.getOrThrow()
    }
}
```

> When suspendCoroutine is called inside a coroutine (and it can only be called inside a coroutine, because it is a suspending function) it captures the execution state of a coroutine in a continuation instance and passes this continuation to the specified block as an argument.

<br>

위 내용은 kotlin repository에 기재된 내용으로 ```suspendCoroutine``` 함수가 실행되면 ```Continuation```객체안의 ```Coroutine``` 실행 상태를 저장하고 ```Continuation``` 객체를 ```block```의 매게변수로 넣어준다.

```
val safe = SafeContinuation(c.intercepted())
block(safe)
```

(추론) 

1. ```c: Continuation<T>``` 는 현재 실행 중인 코루틴(suspend 블럭을 컴파일 하면서 만들어준 ```Continuation``` 인스턴스) 이고 현재 상태를 ```intercepted()``` 함수를 통해 중지 시킨다.

2. 중지 시킨 Continuation을 ```SafeContinuation``` 타입으로 만들어 `block()`의 매게변수로 넘겨준다.

3. __해결 방법__ 에서 보았듯 작성한 block안에서 매게변수 ```continuation```을 사용해 ```resume```, ```resumeWith```을 사용할 때 까지 현재 코루틴을 정지시킨다.
 
<br>

(결론)

```suspendCoroutine``` 함수를 사용하면 직접적으로 중단 지점을 만들 수 있고 따라서 __Callback 기반 API__ 를 ```suspend``` 함수로 만들어 줄 수 있다.

<br>

## 궁금한 점 2

__CancelableContinuation__ 은 어떻게 취소가 가능한가?

1. ```SafeCoroutine``` 인스턴스를 생성할 때와 다르게 ```SuspendCancelableCoroutineImpl```구현체는 ```resumeMode = MODE_CANCELLABLE``` 을 기본 값으로 생성시점에 주입받음, (Cancel 가능한 코루틴인지 명시)

```kotlin
public suspend inline fun <T> suspendCancellableCoroutine(
    crossinline block: (CancellableContinuation<T>) -> Unit
): T =
    suspendCoroutineUninterceptedOrReturn { uCont ->
        val cancellable = CancellableContinuationImpl(uCont.intercepted(), resumeMode = MODE_CANCELLABLE)
        /*
         * For non-atomic cancellation we setup parent-child relationship immediately
         * in case when `block` blocks the current thread (e.g. Rx2 with trampoline scheduler), but
         * properly supports cancellation.
         */
        cancellable.initCancellability()
        block(cancellable)
        cancellable.getResult()
    }
```

2. CancelableContinuation 은 ```Coroutine Job``` 상태의 하위 상태로 __Active__ __Resumed__ __Canceled__ 3가지 상태 값을 가짐 (reumeMode와는 별개입니다)

 | **State**                           | [isActive] | [isCompleted] | [isCancelled] |
 | ----------------------------------- | ---------- | ------------- | ------------- |
 | _Active_ (initial state)            | `true`     | `false`       | `false`       |
 | _Resumed_ (final _completed_ state) | `false`    | `true`        | `false`       |
 | _Canceled_ (final _completed_ state)| `false`    | `true`        | `true`        |



     +-----------+   resume    +---------+
     |  Active   | ----------> | Resumed |
     +-----------+             +---------+
           |
           | cancel
           V
     +-----------+
     | Cancelled |
     +-----------+

```resume()``` 또는 ```resumeWith()```를 호출하면 __Active__ 상태에서 __Resumed__ 상태로 이동하고

__cancel__ 이 발생되면 __Active__ 상태에서 __Canceled__ 상태로 이동한다.

<br>

### CancelableContinuation이 취소 시점 리소스 해제 방법

```kotlin
/*
 * This function provides **prompt cancellation guarantee**.
 * If the [Job] of the current coroutine was cancelled while this function was suspended it will not resume
 * successfully.

 * The prompt cancellation guarantee is the result of a coordinated implementation inside `suspendCancellableCoroutine`
 * function and the [CoroutineDispatcher] class.
 */
```

> ```SuspendCancelableCoroutine``` 인터페이스 Docs 내용중 

<br>

Kotlin 공식 문서에서는 __Prompt Cacellation Guarantees__ 라는 용어로 표현한다.

__Prompt Cacellation Guarantees__ 는 ```suspendCancelableCoroutine``` 구현 코드와 ```CoroutineDispatcher``` 클래스의 협력으로 보장된다고 한다.

```suspendCancelableCoroutine``` 구현 코드를 확인해보자.

<br>

```invokeOnCancellation()``` 함수는 람다의 형태로 ```CompletionHandler```를 넘겨준다(리소스를 정리하는 코드)

내부적으로 ```InvokeOnCancel``` 라는 클래스를 갖고있으며 ```CompletionHandler``` 를 주입받고 실행시키는 역할을 갖는다.



```kotlin
private class InvokeOnCancel( // Clashes with InvokeOnCancellation
    private val handler: CompletionHandler
) : CancelHandler() {
    override fun invoke(cause: Throwable?) {
        handler.invoke(cause)
    }
    override fun toString() = "InvokeOnCancel[${handler.classSimpleName}@$hexAddress]"
}
```


<br>

```makeCancelHandler()``` 내부 함수를 통해 ```CancelHandler``` 객체를 만들어준다.

```kotlin
    private fun makeCancelHandler(handler: CompletionHandler): CancelHandler =
        if (handler is CancelHandler) handler else InvokeOnCancel(handler)
```

```kotlin
/**
 * Base class for all [CancellableContinuation.invokeOnCancellation] handlers to avoid an extra instance
 * on JVM, yet support JS where you cannot extend from a functional type.
 */
internal abstract class CancelHandler : CancelHandlerBase(), NotCompleted
```

```CanclHandler``` 는 ```CancelHandlerBase``` 클래스를 상속하고 ```CacelHandlerBase``` 클래스는 ```CompletionHandler```를 상속받는다.

```CompletionHandler``` 객체는 ```invokeOnCancellation(handler: CompletionHandler)``` 함수를 사용할 때 작성했던 리소스 제거 역할 익명함수이다.

```kotlin
internal actual abstract class CancelHandlerBase actual constructor() : CompletionHandler {
    actual abstract override fun invoke(cause: Throwable?)
}

public typealias CompletionHandler = (cause: Throwable?) -> Unit
```

<br>

__내부 코드에 의해 우리가 정의했던 익명함수가 ```InvokeOnCancel``` 클래스의 인스턴스로 변환된다.__

```InvokeOnCancel``` 인스턴스는 ```invokeOnCancellation()``` 함수에서 사용되며 내부 ```_state```값에 따라 분기된 처리를 해준다.

<br>

```_state```는 ```CancelableContinuationImpl``` 내부 로직에서 사용되는 상태값으로 기본적으로 __Active__ 값이 설정되어있다.(CancelableContiuation 인터페이스의 상태값과 다름)

또한 ```_state``` 의 상태로 ```CancelHandler```가 될 수 있다!!


```kotlin
    /**
       === Internal states ===
       name        state class          public state    description
       ------      ------------         ------------    -----------
       ACTIVE      Active               : Active        active, no listeners
       SINGLE_A    CancelHandler        : Active        active, one cancellation listener
       CANCELLED   CancelledContinuation: Cancelled     cancelled (final state)
       COMPLETED   any                  : Completed     produced some result or threw an exception (final state)
    **/
    private val _state = atomic<Any?>(Active)
```

```kotlin
// invokeOnCancellation() 구현부
 public override fun invokeOnCancellation(handler: CompletionHandler) {
        val cancelHandler = makeCancelHandler(handler)
        _state.loop { state ->
            when (state) {
                is Active -> {
                    // Active의 경우 Atomic 연산자를 사용해 thread-safe하게 _state를 검사, 문제가 없다면 반환!!
                    if (_state.compareAndSet(state, cancelHandler)) return 
                }
                is CancelHandler -> multipleHandlersError(handler, state) // 핸들러가 여러번 호출되는 것을 방지
                is CompletedExceptionally -> {
                    // 핸들러가 여러번 호출되는 것을 방지
                    if (!state.makeHandled()) multipleHandlersError(handler, state)
                    
                    // 취소 상태라면 핸들러를 실행
                    if (state is CancelledContinuation) {
                        callCancelHandler(handler, (state as? CompletedExceptionally)?.cause)
                    }
                    return
                }
                is CompletedContinuation -> {
                    // 종료 후 cancelhandler를 중복으로 갖는지 확인
                    if (state.cancelHandler != null) multipleHandlersError(handler, state)
                    // BeforeResumeCancelHandler 상태를 갖는다면 이미 리소스가 제거된 상태라 즉시 반환 (CancelableContiuationImpl.kt 560~566 line 참고)
                    if (cancelHandler is BeforeResumeCancelHandler) return
                    if (state.cancelled) {
                        // Was already cancelled while being dispatched -- invoke the handler directly
                        callCancelHandler(handler, state.cancelCause)
                        return
                    }
                    val update = state.copy(cancelHandler = cancelHandler)
                    if (_state.compareAndSet(state, update)) return // quit on cas success
                }
                else -> {
                    // 이미 종료되었지만 CompletedContinuation상태가 아닌 예외 상황에서 _state 값을 업데이트
                    if (cancelHandler is BeforeResumeCancelHandler) return
                    val update = CompletedContinuation(state, cancelHandler = cancelHandler)
                    if (_state.compareAndSet(state, update)) return // quit on cas success
                }
            }
        }
    }
```

완벽하게 이해하진 못했지만 ```CancelableContinuation.invokeOnCancellation()``` 함수는 위와 같은 로직으로 동작하고있고

```CancelableContinuation``` 내부 함수는 대부분 ```_state``` 상태값을 확인해 분기처리를 해주면서 __Prompt Cancellation Guarantees__ 를 구현하는걸로 확인했다.

여러번 호출되는 콜백을 `Flow`로 반환하는 경우는 Part.2에 이어서 작성하겠습니다.

