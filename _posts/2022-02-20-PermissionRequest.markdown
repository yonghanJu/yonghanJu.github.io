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

    // 바로 실행 권한 작업 실행
    ContextCompat.checkSelfPermission(
        this,
        Manifest.permission.READ_EXTERNAL_STORAGE
    ) == PackageManager.PERMISSION_GRANTED ->{
        // 권한 사용 가능
        //todo()
    }

    // 해당 권한이 필요한 경우 팝업 띄우기
    ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.READ_EXTERNAL_STORAGE) ->{
    // 교육용 팝업을 띄운 후 권한 요청
    //showPermissionContextPopup() // 직접 팝업을 생성
    }
    // 권한 요청하기
    else ->{
        ActivityCompat.requestPermissions(this,arrayOf(Manifest.permission.READ_EXTERNAL_STORAGE),1000) // 1000 코드 기억
    }
}
```


<br><Br>


* 권한 거부시 교육용 팝업 설정


```kotlin
// 위 when 절에서 사용된 함수, AlertDialog 생성
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
}
```

<br><br>

* 권한 허용시 즉시 동작 설정

```kotlin
override fun onRequestPermissionsResult(requestCode: Int, permissions: Array<out String>, grantResults: IntArray){
    super.onRequestPermissionsResult(requestCode, permissions, grantResults)

    when(requestCode){
        1000 -> {
            if(grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED){
                // 권한이 부여되었음, 기능 동작
                todo()
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

```kotlin
private fun todo(){
    val intent = Intent(Intent.Action_GET_CONTENT) // 컨텐트 가져오기
    intent.type = "image/*"
    startActivityForResult(intent, 2000)    // 결과 받아오는데 사용
}
```

<br><br>

* intent activity 결과(사진) 받아오기

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