---
layout: post
title:  SoundPool, SeekBar, CountDownTimer
date:   2022-02-28 19:12:00 +0900
categories:   Codelab
---


# 사운드 풀 예제

<br>

```kotlin
private val soundPool = SoundPool.Builder().build()
private val soundId:Int? = null
private fun initSoundPool(){
    soundId = soundPool.load(this:Context, R.raw.ID:Int, 1)
}

// id, left, right, loop(0 = no, -1 = forever), rate 
soundId?.let{ id-> soundPool.play(id, 1F, 1F, 0, -1, 1F))}

// 라이프사이클 조심
override onResume(){
    super.onResume()
    soudPool.outoResume()
}

override onPause(){
    super.onPause()
    soudPool.outoPause() 
}

override onDestroy(){
    super.onDestroy()
    soundPool.release()
}
```

<br><br>

# SeekBar 속성

<Br>

```xml
<!--현재 위치 표시 아이콘-->
android:thumb="drawable/ic_thumb" 

<!--바 색깔-->
android:progressDrawable="@color/transparent"

<!--눈금 이미지-->
app:tickMark="@drawable/tick_mark"
```

<br><br>

# 카운트다운타이머 예제

<Br>

```kotlin
private fun createCountDownTimer(initialMillis: Long) = 
    object: CountDownTimer(initialMillis, 1000){
        override fun onTick(millisUntilFinished: Long){
            val num = 6
            Log.d("tag","%02d".format(num)) // 06
        }

        override onFinish(){
            TODO()
        }
    }
```

