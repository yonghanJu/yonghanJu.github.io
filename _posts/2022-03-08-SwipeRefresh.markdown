---
layout: post
title:  Swiperefreshlayout(WebView), ContentLoadingProgressBar
date:   2022-03-08 15:50:00 +0900
categories:   Codelab
---


# 당겨서 새로고침 구현

* Swiperefreshlayout 종속성 추가

<br>

* Swiperefreshlayout xml 추가

<br>

```xml
<androidx.swiperefreshlayout.widget.SwipeRefreshLayout
    ...>

    <WebView>

</androidx.swiperefreshlayout.widget.SwipeRefreshLayout>
```

<br>

* 리프레시 event 정의

<Br>

```kotlin
// 리프레시 레이아웃 선언
private val refreshLayout: SwiperefreshLayout by lazy{
    binding.refreshLayout
}

private fun bindViews(){
    refreshLayout.setOnRefreshListener{ // 리프레시 event 설정 
        //TODO(), 웹뷰를 새로고침
        webView.reload()
    }
}
```

<br>


* 리프레시 후 isRefreshing 값 변경, __웹뷰 클라이언트__ 상속

<br>

```kotlin
// 웹 로딩에 관한 메서드들 정의
// inner class 주의! 상위 클래스의 맴버에 접근 할 수 있음!!
inner class WebViewClient: android.webkit.WebViewClient(){ 

    // 웹 로딩이 시작되면
    override fun onPageStarted(view:WebView?, url:String?, favicon:Bitmap?){
        super.onPageStarted(view, url, favicon)
        // progressBar.show()
    }

    // 웹 로딩이 끝나면  isRefreshing 완료
    override fun onPageFinished(view:WebView?, url:String?){ 
        super.onPageFinished(view, url)

        refreshLayout.isRefreshing = false
        // progressBar.hide()   // 웹 뷰 로딩 프로그래스 바 숨기기
        // goBackButton.isEnable = webView.canGoBack() // 뒤로가기가 가능할 때만 버튼 활성화
        // goForwardButton.isEnable = webView.canGoForward() // 앞으로가기가 가능할 때만 버튼 활성화
        // addressBar.setText(url)  // 실제 로딩된 url을 주소창에 띄우기
    }

}
```

<Br>

* 로딩 프로그레스바 선언, __웹크롬 클라이언트__ 상속

<Br>

```kotlin
// xml에 ContentLoadingProgressBar 생성 후 선언
private val progressBar: ContentLoadingProgressBar by lazy{binding.progressBar}

// 웹뷰 클라이언트는 브라우저 차원에서 웹 이벤트 관한 메서드 정의
// inner class 주의! 상위 클래스의 맴버에 접근 할 수 있음!!
inner class WebChomeClient: android.webkit.WebChromeClient(){
    override fun onProgressChanged(view:WebView?, newProgress:Int){ // 로딩 정도가 변경되면 실행됨(0~100)
        super.onProgressChanged(view, newProgress)

        progressBar.progress = newProgress
    }
}

webView.webChromeClient = WebChomeClient()
```