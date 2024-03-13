---
layout: post
logo: "assets/images/15balloon.png"
navigation: True
cover: "assets/images/cover1.jpg"
title: "Flutter WebView 흰 화면 문제 해결"
author: "15balloon"
date: 2024-03-13 23:14:13 +0900
tags: android error flutter
categories: 15balloon
subclass: "post tag-test tag-content"
---

회사에서 새로운 프로젝트를 진행하게 되었다.   
일단은 안드로이드 앱을 먼저 출시하겠지만 추후 iOS 앱도 출시해야 하기 때문에 Flutter로 개발하기로 했다.   
이 프로젝트는 PG사 결제 연동도 구현해야 하는데, 결제 기능이 WebView를 통해 이루어지고 있어서 **webview_flutter** 라이브러리를 사용해서 개발을 하였다.   

#### 이상 현상
결제 모듈을 앱에 붙여서 테스트를 진행하였는데, 다른 앱으로 전환했다가 다시 앱에 진입하면 **WebView가 흰 화면으로 보여지는 문제**가 발생했다.   
로그를 찍어 살펴보니 다시 앱에 진입할 때 WebView가 두 번씩 reload 되고 있는데 이게 문제인건지 알 수가 없었다.   
그래서 구글 검색을 해보니 Flutter Github의 Issue 탭에 올라온 글을 발견할 수 있었다.   

#### 문제 해결
webview_flutter 라이브러리 사용 시 Android 14 버전을 가진 삼성 기기에서 흰 화면이 표시되는 문제가 있다는 것을 어떤 사람이 리포트한 것이다.   
갤럭시 S23에 Android 14가 정식 배포된게 11월인데 글이 11월 말에 올라왔으니 거의 바로 리포트된 셈이다.   
글을 쭉 읽으면 삼성에도 이 문제를 알렸지만 아직까지도 별다른 대응은 없는 것을 확인할 수 있다.   
*mxnortal*이라는 유저가 임시 해결 방안을 올려놓았는데, 이를 적용하여 문제를 해결할 수 있었다.   
코드를 살펴보면 다음과 같다.   
```dart
@override    
void didChangeAppLifecycleState(AppLifecycleState state) {
    if (state == AppLifecycleState.resumed) {
        _shouldReloadWebView();
    }
}

Future<void> _shouldReloadWebView() async {
    if (Platform.isAndroid) {
        try {
            var androidInfo = await DeviceInfoPlugin().androidInfo;
            var sdkInt = androidInfo.version.sdkInt;
            var manufacturer = androidInfo.manufacturer;

            if (sdkInt == 34 && manufacturer == 'samsung') {
                setState(() {
                    _isInProgress = true;
                });

                Future.delayed(const Duration(milliseconds: 300), () {
                    setState(() {
                        _isInProgress = false;
                    });
                });
            }
        } catch (_) {}
    }
}

@override
  Widget build(BuildContext context) => _isInProgress
      ? const SizedBox.shrink()
      : WebViewWidget();
```
onResume 상태가 된 경우, 사용하는 기기가 삼성 기기이면서 Android 14 버전이면 빈 widget을 그렸다가 WebView를 그린다.   
이렇게 코드를 수정하면 다른 앱으로 전환 후에 다시 돌아와도 WebView가 잘 보이는 것을 확인할 수 있다.   

#### Conclusion
삼성 기기만의 문제일 것이라고는 생각도 못해서 빠르게 해결하지는 못했다.   
Flutter도 처음이었고, 결제 모듈 연동도 처음이어서 코드에 문제가 있지 않을까 라는 생각이 먼저 들었기 때문이다.   
다행히 Issue에 리포트된 사항이었고, 임시 해결 방안까지 있어서 해결할 수 있었다.   
물론 이후에 POST 방식으로 데이터를 전송함과 동시에 페이지를 여는 기능이 필요해져서 **flutter_inappwebview** 라이브러리를 사용하게 되어 이 해결 방안을 사용하게 되진 않았지만 말이다.   

##### Reference:

1. [Samsung phones running Android 14 stops drawing platform views on resume](https://github.com/flutter/flutter/issues/139039)