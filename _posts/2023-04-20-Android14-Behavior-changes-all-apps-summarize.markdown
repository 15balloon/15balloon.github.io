---
layout: post
logo: "assets/images/15balloon.png"
navigation: True
cover: "assets/images/cover1.jpg"
title: "Android 14 Behavior changes: all apps 정리"
author: "15balloon"
date: 2023-04-20 00:35:13 +0900
tags: android
categories: 15balloon
subclass: "post tag-test tag-content"
---

이 포스트에서는 targetSdkVersion과 상관없이 Android 14에서 작동하는 모든 앱들에 영향을 미치는 주요 동작 변화(behavior changes)를 다룬다.   
이 변화들은 안드로이드 플랫폼의 성능, 보안 그리고 사용자 경험성 향상시키기 위해 도입되었다.   
안드로이드 개발자라면, Android 14에서 앱을 테스트해야 하고, 이러한 변화를 지원하기 위해 필요한 수정을 해야 한다.   

다음은 Android 14에서 모든 앱에 적용되는 주요 동작 변화들이다.   

#### 핵심 기능   
 - SCHEDULE_EXACT_ALARM 권한의 기본 값이 거부(Denied)로 변경:   
 정확한 알람(Exact Alarm)은 정확한 시간에 발생해야 하는 사용자 지정 알림(user-intentioned notification) 혹은 action을 의미한다. Android 14부터 Android 13 이상을 타겟팅한 앱 설치 시 SCHEDULE_EXACT_ALARM 권한은 기본적으로 거부된다.   
 - 앱이 캐시되는 동안 컨텍스트(Context) 기반 브로드캐스트들은 큐에 적재(queued):   
 Android 14에서 해당 브로드캐스트들은 사용자에게 보여지지 않으며 낮은 메모리 우선순위를 가지게 된다. 이는 Android 12에서 도입된 비동기 바인더 전환(async binder transaction)과 유사하다. 매니페스트에 정의된 브로드캐스트들은 큐에 쌓이지 않고, 해당 앱들은 브로드캐스트 전달을 위해 캐시 상태에서 삭제된다. 앱이 포그라운드로 돌아가는 등, 캐시 상태를 벗어나게 되면 시스템은 큐에 쌓인 브로드캐스트들을 전달한다. 어떤 브로드캐스트가 여러 인스턴스들을 가진다면 하나의 브로드캐스트로 합친다.  
 - 앱은 자기 자신의 백그라운드 프로세스들만을 죽일 수 있음:   
 Android 14부터는 앱이 killBackgroundProcesses()를 호출하여 자기 자신의 백그라운드 프로세스만을 죽일 수 있게 된다. 만약 다른 앱의 백그라운드 프로세스를 죽이려 한다면 아무런 영향없이 로그캣에 경고 메세지만을 출력하게 된다. 안드로이드는 캐시된 앱을 백그라운드에 유지하고 시스템에 메모리가 필요할 때 죽이도록 설계되어 있다. 만약 앱이 다른 앱을 불필요하게 종료시킨다면 나중에 앱을 완전히 재시작해야 한다. 이는 캐시된 앱을 재시작하는 것보다 더 많은 리소스를 필요로 하기 때문에 시스템 성능 저하와 배터리 소모량이 증가할 우려가 있다.   

#### 보안   
 - 설치 가능한 최소 API 레벨:   
 사용자의 보안과 개인 정보 보호를 위해 Android 14부터 targetSdkVersion이 23 미만인 앱을 설치할 수 없다. SDK 버전 23 미만의 앱을 새로 설치하려고 한다면 설치에 실패하고, 로그캣에 다음과 같은 메세지를 남긴다.   
 ```
 INSTALL_FAILED_DEPRECATED_SDK_VERSION: App package must target at least SDK version 23, but found 7
 ```
 앱을 새로 설치하는 경우에만 해당되기 때문에, 기기를 Android 14로 업그레이드해도 targetSdkVersion이 23 미만인 앱들은 남아 있는다.   
 - 미디어 소유 패키지명(Media owner package name) 변경:   
 미디어 스토어(media store)는 특정 미디어 파일을 저장한 앱을 나타내는 OWNER_PACKAGE_NAME 컬럼에 대한 쿼리를 지원한다. 다음 조건 중 하나를 만족한다면 이 값이 바뀌게 된다.   
   - 미디어 파일을 저장한 앱에 다른 앱에 항상 표시되는 패키지 이름이 있음   
   - QUERY_ALL_PACKAGES 권한을 요청하는 앱   

#### 사용자 경험   
 - 닫을 수 없는 알림 방식 개선:   
 닫을 수 없는 포그라운드 알림을 표시하는 경우, Android 14부터는 이 알림을 닫을 수 있도록 변경했다. Notification.FLAG_ONGOING_EVENT에서 Notification.Builder#setOngoing(true) 혹은 NotificationCompat.Builder#setOngoing(true)를 설정한 경우가 해당된다. 다음 조건에서는 적용되지 않는다.   
   - 폰이 잠겨 있는 경우   
   - 사용자가 '모든 알림 지우기' 작업을 선택한 경우(실수로 눌렀을 수도 있으므로)   

    그리고 다음 사례들에서도 적용되지 않는다.

   + MediaStyle에 의해 생성된 알림   
   + 보안과 개인 정보 보호를 위한 제한 정책이 적용된 경우
   + 기기 정책 컨트롤러(Device policy controller)와 기업용 패키지인 경우   
 - 사진과 비디오에 대한 부분 접근 권한 부여:   
 (만약 'photo picker'를 사용하는 앱인 경우 다음 내용을 적용하지 않아도 된다.)   
 Android   14에서는 Android 13에 적용된 미디어 권한을 요청할 때 부분 접근 권한을 부여할 수 있다. 새로운 Dialog는 다음과 같은 권한 선택지를 보여준다.   
   - Select photos and videos: Android 14에 추가된 선택지. 사용자는 특정 사진과 비디오만을 앱에서 사용할 수 있도록 선택할 수 있다.   
   - Allow all: 모든 사진과 비디오를 접근하도록 허용한다.   
   - Don't allow: 모든 권한을 거부한다.   

    이 사항을 앱에서 원활히 처리하고 싶다면 READ_MEDIA_VISUAL_USER_SELECTED 권한 선언을 고려해야 한다.   

#### 접근성   
 - 최대 200%의 비선형 글꼴 크기 조정(non-linear font scaling) 지원:   
 Android 14부터 저시력 사용자를 위해 최대 200%의 글꼴 크기 조정 옵션을 제공한다. 이미 앱에서 scaled pixels(sp) 단위로 텍스트 크기를 정의한 경우에는 큰 영향을 미치지 않는다. 그러나 UI 테스트를 수행하여 앱이 200%의 글꼴 크기를 가져도 사용성에 문제가 없는지 확인해야 한다.   
 <img src="{{site.baseurl}}/assets/images/2023-04-20-Android14-Behavior-changes-all-apps-summarize_1.png" width="100%" height="100%" title="Non-linear scaling" alt="Image: Non-linear scaling"/>
 비선형 크기 조정이란, 화면의 텍스트 요소가 너무 커지는 것을 방지하기 위해 시스템에서 비선형 배율 곡선을 적용하는 것을 말한다. 이 크기 조정 방식은 큰 텍스트와 작은 텍스트의 크기가 같은 비율로 조정되지 않음을 뜻한다. (위 이미지 참조)

##### Reference:

1. [Behavior changes: all apps Android 14](https://developer.android.com/about/versions/14/behavior-changes-all)
2. [The first developer preview of Android 14](https://android-developers.googleblog.com/2023/02/first-developer-preview-android14.html)