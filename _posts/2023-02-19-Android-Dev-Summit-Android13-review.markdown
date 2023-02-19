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
        SDK 버전 33에서 변화한 다른 동작들은 추후 업데이트할 예정이다.   

---

##### Reference:

1. [Migrate Your Apps to Android 13](https://youtu.be/wBx3-ZObxY8?list=PLWz5rJ2EKKc8PO99T1QQLrPAJILqxJXW6)