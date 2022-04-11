---
layout: post
title:  Permission Request Process(Storage Access Framework)
date:   2022-02-20 00:00:00 +0900
categories:   Codelab
---

# 앱 권한 요청 프로세스



권한 요청 프로세스는 [여기]를 통해 자세히 확인하기

[여기]: https://developer.android.com/training/permissions/requesting?hl=ko

<br>

### 예제 코드

<br>

* 권한 요청 코드 작성

```kotlin
//권한 요청하기 코드
when{

    // 1. 권한이 허용되어있는 경우
    ContextCompat.checkSelfPermission(
        this,
        Manifest.permission.READ_EXTERNAL_STORAGE
    ) == PackageManager.PERMISSION_GRANTED ->{
        TODO()
    }

    // 2. 해당 권한이 거부 된 경우
    ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.READ_EXTERNAL_STORAGE) ->{
    // 교육용 팝업을 띄운 후 권한 요청
    //showPermissionContextPopup() // 직접 팝업을 생성
    }

    // 3. 앱을 처음 실행한 경우
    else ->{
    // 권한 요청하기
        ActivityCompat.requestPermissions(this,arrayOf(Manifest.permission.READ_EXTERNAL_STORAGE),1000) // 1000 코드 기억
    }
}
```


<br><Br>


* 권한 거부시 교육용 팝업 설정


```kotlin
// 위 when 절의 2번에서 사용된 교육용 팝업 함수
private fun showPermissionContextPopup(){
    AlertDialog.Builder(this:Context)
    .setTitle("권한 요청")
    .setMessage("권한이 필요합니다.")
    .setPositiveButton("동의하기") { _, _ -> 
        // 동의 -> 권한 요청
        ActivityCompat.requestPermissions(this,arrayOf(android.Manifest.permission.READ_EXTERNAL_STORAGE), 1000)
    }
    .setNegativeButton("취소하기"){ _, _ -> }
    .create()
    .show()
}
```

<br><br>

* 권한 요청 후 자동으로 호출되는 콜백 함수

```kotlin
override fun onRequestPermissionsResult(requestCode: Int, permissions: Array<out String>, grantResults: IntArray){
    super.onRequestPermissionsResult(requestCode, permissions, grantResults)

    when(requestCode){
        // 위 권한요청에 사용된 요청코드(SAF권한 요청 코드)
        1000 -> {
            if(grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED){
                // 권한이 부여되었음, 기능 동작
                TODO()
            }
        }
        else -> { 
            // 다른 권한 요청의 경우
        }
    }

}
```

<br><br>

* Manifest에 권한 선언하기

```xml
<uses-permission android:name="android.Manifest.permission.READ_EXTERNAL_STORAGE"/>

<application
    ...
```

<br><br>

* SAF(Storage Access Framework) 사용, 엑티비티 시작

__startActivityForResult(intent, code)__ 함수를 사용해 엑티비티 실행의 후 콜백 함수(__onActivityResult__) 호출

```kotlin
private fun todo(){
    val intent = Intent(Intent.Action_GET_CONTENT) // 컨텐트 가져오기
    intent.type = "image/*"

    // 결과 받아오는데 사용
    // startActivityForResult(intent, 2000)    // 예전 방식입니다. 
    
    // 최신화, activityResultLauncher 객체 생성은 아래에서 나옴
    activityResultLauncher.launcher(intent)
}
```

<br><br>

* ~~intent activity 결과(사진) 받아오기, 엑티비티 실행 후 자동 호출 콜백 함수~~ 예전 방식입니다. 아래 최신화 있음

```kotlin
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?){
    super.onActivityResult(requestCode, resultCode, data)

    if(resultCode != Activity.RESULT_OK){ // 비정상 종료의 경우
        return
    }
    
    when(requestCode){
        2000 -> {
            val selectedImageUri: Uri? = data?.data
            if(selectedImageUri != null){
                // ImageView
                imageView.set.setImageUri(selectedImageUri)
            }else{
                // 사진 가져오기 실패
            }
        }
        else -> { }
    }
}
```

<br>

* Activity 결과 받아오기

__activityResultLauncher__ 객체 생성 후 __startActivityForResult()__ 대신 __launcher()__ 사용!

```kotlin
private val activityResultLauncher = registerForActivityResult(
    ActivityResultContracts.StartActivityForResult()
){
    if(it.resultCode == RESULT_OK){ it:ActivityResult! ->
        val selectedImageUri:Uri? = it.data?data
        if(selectedImageUri != null){
            // ImageView
        imageView.set.setImageUri(selectedImageUri)
        }else{
        // 사진 가져오기 실패
        }
    }
    else -> { }
}
```