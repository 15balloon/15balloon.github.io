---
layout: post
logo: "assets/images/15balloon.png"
navigation: True
cover: "assets/images/cover1.jpg"
title: "Android Dev Summit 2022 : 'Migrate Your Apps to Android 13' 세션 후기"
author: "15balloon"
date: 2023-02-19 16:00:37 +0900
tags: android conference
categories: 15balloon
subclass: "post tag-test tag-content"
---

##### 세션 내용 요약

1. Android 13 버전의 변화   
    Android 13 버전이 릴리즈됨에 따라 모든 앱에 영향을 미치는 사항과 SDK 버전이 33 이상인 경우에만 영향이 미치는 사항이 생겼다.   
    모든 앱에 영향을 미치는 사항은 사용자에게 앱의 백그라운드 동작을 알리고 이를 제어할 수 있도록 하기 위함이다.
    1. 모든 앱에 영향을 미치는 사항
        - 사용자가 포그라운드 서비스(이하 FGS)를 종료할 수 있도록 Task Manager 제공한다.   
        FGS가 실행 중이라면 이를 새로운 인터페이스에서 실행 중인 앱을 보고 종료할 수 있도록 한다.   
        adb 커맨드를 사용하여 FGS를 종료할 때와 동일한 상황을 테스트할 수 있다.   
        ```
        adb shell cmd activity stop-app PACKAGE_NAME
        ```
        force-stop 커맨드와도 유사한데, FGS Task Manager와는 다른 점이 있다.   
        ```
        adb shell cmd activity force-stop PACKAGE_NAME
        ```
        FGS Task Manager로 인한 앱 종료 시에는 기록(History)에서 앱이 삭제되지 않고, 예약된 Job이 취소되지 않는다.   
        <img src="{{site.baseurl}}/assets/images/2023-02-19-Android13_1.jpg" width="100%" height="100%" title="Effects of closure" alt="Image: Effects of closure"/>
        그렇기에 앱에 장기적인 영향이 덜 미친다.   
        FGS Task Manager에 의해 종료된 앱은 콜백을 받지는 않으나, 앱 시작 시 ApplicationExitInfo를 통해 당시 상황을 알 수 있다.   
        이를 통해 사용자에 의한 종료인지 여부와 오류, ANR, Exception, 메모리 부족 문제 등을 확인할 수 있다.   
        FGS Task Manager를 통해 데이터 손실이 최소화되도록 종료 명령을 처리하고, FGS가 정말 앱에 필요한지 확인해야 한다고 당부했다.   
        - 백그라운드 액티비티가 제한된 앱 대기 버킷(Restricted App Standby Bucket)으로 이동하기 까지 대기하는 기간이 45일에서 8일로 짧아졌다.   
        제한된 앱 대기 버킷은 Android 12에서 도입되었는데, Job들이 하루에 10분 동안만 수행이 되고, 긴급 Job들도 더 적게 실행이 되며, 알람도 하나만 호출할 수 있도록 하는 기능이다.   
        아래 코드를 통해, 앱이 해당 상태로 변경된 경우 필수적인 Job을 예약할 수 있도록 지원할 수 있다.   
        ```kotlin
        val usageStatsManager = getSystemService<UsageStatsManager>()
        usageStatsManager?.let {
          if (it.appStandbyBucket == STANDBY_BUCKET_RESTRICTED) {
            ...
          }
        }
        ```
        Job과 알람을 실행할 때 앱 상태를 로깅(Logging)해야 예상치 못한 동작들을 추적할 수 있다.   
        앱 대기 버킷 상태도 adb로 테스트할 수 있다.   
        ```
        adb shell am set-standby-bucket PACKAGE_NAME 45
        ```
        앱이 실행 중인 상태에서는 테스트 할 수 없다.   
        앱이 상호작용 중이기 때문에 버킷에서 빠져 나가기 때문이다.   
        앱이 제한된 버킷 상태로 가지 않으려면 해당 앱에 대한 상호작용과 함께 실행 가능한 알람을 받도록 사용자의 동의를 얻어야 한다.   
        Android 13 부터는 권한을 얻어야 해당 작업을 수행할 수 있다.   
        만약 사용자가 이 권한을 거부한다면 다시는 이 권한을 요청하지 않는다.   
        앱 삭제 후 재설치하거나, 앱이 SDK 버전 33에 타겟팅된 경우는 예외이다.   
        AndroidX 함수를 통해 앱에서 알림이 활성화 되었는지 확인할 수 있다.   
        ```kotlin
        val notificationsManagerCompat = 
            NotificationManagerCompat.from(this)
        notificationsManagerCompat.areNotificationsEnabled()
        ```
        adb 커맨드를 통해 권한을 삭제하여 테스트할 수 있다.   
        ```
        adb shell pm revoke PACKAGE_NAME
        android.permission.POST_NOTIFICATIONS
        adb shell pm clear-permission-flags PACKAGE_NAME \
          android.permission.POST_NOTIFICATIONS user-set
        adb shell pm clear-permission-flags PACKAGE_NAME \
          android.permission.POST_NOTIFICATIONS user-fixed
        ```   
        - 앱이 SDK 버전 33에 타겟팅된 경우, 인텐트를 명시적으로 지정하여 인텐트 필터 요소와 일치하는 경우에만 전달되도록 동작이 변경되었다.   
        다른 앱에서 내부 코드를 갑자기 트리거하는 오류를 막기 위해 BroadcastReceiver에 허용되지 않은 intent.action을 방지하는 코드를 추가해야 한다.
        ```kotlin
        if (intent.action.equals(allowedAction)) {
          ...
        }
        ```   
        - 클립보드 콘텐츠를 시각적으로 확인하는 기능이 추가되었다.   
        이에 따라 개인 정보나 신용카드 정보 등 민감한 정보들을 복사했을 때 클립보드에 보여지지 않도록 처리해야 한다.
        ```kotlin
        // SDK version 33 or higher
        clipData.apply {
          description.extras = PersistableBundle().apply {
            putBoolean(ClipDescription.EXTRA_IS_SENSITIVE, true)
          }
        }
        // SDK version lower than 33
        clipData.apply {
          description.extras = PersistableBundle().apply {
            putBoolean("android.content.extra_IS_SENSITIVE", true)
          }
        }
        ```
    2. SDK 버전 33에서 변화한 동작(Privacy를 중심으로)   
        - WiFi 주변 기기(Nearby WiFi Devices) 권한을 도입했다.   
        이전 안드로이드 버전에서는 주변 기기 탐색을 위해 WiFi Manager 클래스를 사용해야 했고, 위치 권한도 필요했다.   
        안드로이드 13에서는 NEARBY_WIFI_DEVICES 런타임 권한을 사용하여 Wifi Manager의 메소드나 위치 권한 없이 주변 기기를 쉽게 탐색할 수 있다.   

          ```xml
          <uses-permission
          android:name="android.permission.NEARBY_WIFI_DEVICES"
          android:usesPermissionFlags="neverForLocation" />
          ```
        앱을 SDK 버전 33에 타겟팅하는 경우, 주변 WiFi 기기 탐색 시 NEARBY_WIFI_DEVICES 권한을 요청하지 않는다면 많은 Exception을 일으키게 되니 이 점을 유의해야 한다.   
        - 미디어 권한이 세분화되었다.   
        이에 따라 READ_EXTERNAL_STORAGE 권한 대신 미디어 종류에 따라 다음과 같은 권한을 요청해야 한다.   
        <img src="{{site.baseurl}}/assets/images/2023-02-19-Android13_2.jpg" width="100%" height="100%" title="Granular media permissions" alt="Image: Granular media permissions"/>
        - 생체 신호 센서(Body sensor)를 백그라운드에서 사용하기 위해 허용해야 하는 권한이 추가되었다.   

          ```xml
          <uses-permission
          android:name="android.permission.BODY_SENSORS_BACKGROUND" />
          ```
        - 더 이상 SDK에 포함되지 않는 인터페이스들이 있다.   
        해당 인터페이스는 다음 4개이며, 대체 인터페이스를 사용하면 된다.   
        <img src="{{site.baseurl}}/assets/images/2023-02-19-Android13_3.jpeg" width="100%" height="100%" title="Non-SDK interfaces" alt="Image: Non-SDK interfaces"/>

#### 후기

최근 여러 기종에 대한 Android 13 업그레이드가 적용되면서 회사에서 서비스 중이던 한 모바일 앱에 크고 작은 문제들이 발생했었다.   
그런데 이에 대한 대비가 안 되어 있어, 급하게 수정 작업을 진행하여 배포가 이루어졌다고 한다.   
문제가 해결된 이후에 해당 모바일 앱의 유지보수 업무가 우리 팀으로 이관되면서 안드로이드 버전에 따른 앱 마이그레이션 사항을 살펴보게 되었다.   
안드로이드 개발자임에도 안드로이드 버전에 따른 변경 사항을 잘 알지 못했던 것은 지금까지 TV 앱 개발을 진행해왔기 때문이다.   
특히나 일반 TV 앱이 아니고, 특정 셋탑에 종속되는 앱이었기 때문에 더욱 관심도가 떨어져 있었다.   
아무리 TV 앱을 개발한다지만 어떻게 안드로이드 관련 최신 내용을 공부하지 않았냐고 묻는다면 딱히 할 말이 없다.   
지금이라도 하고자 하니 따뜻하게 바라봐 주길 바란다.   
<br>
일단 정리한 내용은 당장 주요하게 봐야 하는 것들이다.   
생략된 내용은 추후에 정리하고자 한다.   
'모든 앱에 영향을 미치는 사항'은 뜬금없이 발생하는 오류를 막기 위해 꼭 숙지해야 하는 내용이라 주의 깊게 들었다.   
일단은 인텐트 관련 내용 말고는 유지보수 시 신경 써야 하는 내용은 없긴 했다.   
그래도 FGS와 버킷 관련 내용은 자칫하면 앱에 영향을 크게 미칠 수 있는 것들이라, 최대한 꼼꼼히 살펴보았다.   
특히 버킷 관련 내용은 처음 듣는 내용이라 공식 문서를 읽었다.   
안정적인 앱 서비스를 위해 버킷 관련 예외 처리가 꼭 필요할 듯하다.   
권한 관련 추가 내용은 꼭 숙지해야 하는 사항이기에 세션에서 소개한 세부 세션을 들어볼 계획이다.   
<br>
Android 14의 개발자 프리뷰가 이미 공개되었고, 곧 베타 버전 공개도 예정되어 있으니 앞으로 이를 주의 깊게 살펴볼 계획이다.   
Android 13의 변경 사항은 늦게 알게 되었지만, Android 14는 미리 알아보고 준비해서 안정적인 서비스를 제공하도록 노력할 것이다.


---

##### Reference:

1. [Migrate Your Apps to Android 13](https://youtu.be/wBx3-ZObxY8?list=PLWz5rJ2EKKc8PO99T1QQLrPAJILqxJXW6)