---
layout: post
logo: "assets/images/15balloon.png"
navigation: True
cover: "assets/images/cover1.jpg"
title: "Kotlin Coroutines 정리: 1"
author: "15balloon"
date: 2023-09-22 23:20:06 +0900
tags: android
categories: 15balloon
subclass: "post tag-test tag-content"
---

Kotlin을 사용해 Android 앱 개발을 하는 사람이라면 비동기적으로 코드를 실행시키기 위해 Coroutine 한 번쯤은 다 써봤을 것이다.   
Coroutine을 사용해보았지만 기본 내용만 알고 Channel이나 Flow와 같은 심화 내용은 잘 모르는 사람도 많을 것이다.   

OlymPos 프로젝트를 시작하게 되면서 MVVM 패턴이나 Jetpack Compose와 같이 알고 있었지만 제대로 사용해본 적은 없던 기술을 사용하게 됐다.   
그러면서 내부 코드들도 최신 기술로 채우고 싶다는 열망이 가득했다.   
그래서 먼저 그동안 미뤄왔던 Coroutine 공부를 시작했다.   
Coroutine은 초반부 내용이 쉬워서 조금만 배워도 금방 적용할 수 있지만 Channel이나 Flow 같은 도구들의 활용 부분부터는 러닝커브가 가파른 것으로 알려져 있다.   
목표는 Channel과 Flow까지 학습해서 프로젝트에 사용하는 것으로 잡았다.   

#### Coroutine 이란
Coroutine은 비동기적으로 코드를 수행하는 동시성 디자인 패턴이다.   
**Suspension**을 지원하기 때문에 단일 Thread에서 다수의 Coroutine을 사용할 수 있다.   
Suspension은 실행 중인 Thread를 block(차단)하지 않는 것을 말하는데, 이로 인해 메모리가 절약된다.   
**Structed concurrency**를 사용하여 scope 내에서 작업들이 수행된다.   
Coroutine 계층 구조를 통해 **Cancellation**이 전달된다.   

#### Setup
Coroutine을 사용하려면 build.gradle(:app)에 아래의 dependency를 추가해야 한다.   
```
dependencies {
    <!-- 2023.09.22 기준 최신 버전은 1.6.4 -->
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.6.4'
}
```

#### Executing in a background thread
메인 Thread(UI Thread)에서 네트워크 요청을 보내면 응답을 받을 때까지 Thread가 wait(대기)하거나 block(차단)된다.   
이를 방지하기 위해 Coroutine을 사용해서 이 동작을 메인 Thread 외부에서 실행할 수 있다.   
*viewModelScope*는 ViewModel KTX extensions에 사전 정의된 CoroutineScope이다.   
*launch()* 는 Coroutine을 생성하고 Dispatcher에 함수 body 실행을 전달한다.   
*Dispatchers.IO*는 해당 Coroutine이 I/O 작업을 위한 Thread에서 실행되야 함을 나타낸다.   
이 Coroutine은 ViewModel scope에서 실행되므로, ViewModel이 소멸되면 실행 중인 모든 Coroutine이 취소된다.   
```
class LoginViewModel(
    private val loginRepository: LoginRepository
): ViewModel() {

    fun login(username: String, token: String) {
        // Create a new coroutine to move the execution off the UI thread
        viewModelScope.launch(Dispatchers.IO) {
            val jsonBody = "{ username: \"$username\", token: \"$token\"}"
            loginRepository.makeLoginRequest(jsonBody)
        }
    }
}
```

#### withContext
함수 사용 시 메인 Thread에서 UI 업데이트를 차단하지 않을 때 main-safe 하다고 간주한다.   
*makeLoginRequest()* 를 메인 Thread에서 호출하면 UI가 차단되므로 이 함수는 main-safe 하지 않다.   
위의 코드에서 *makeLoginRequest()* 는 viewModelScope에서 동작하는데, 이 scope는 메인 Thread에서 실행되고 있다.
이때 **withContext()** 함수를 사용하여 Coroutine을 다른 Thread에서 실행할 수 있다.   
**withContext(Dispatchers.IO)** 는 Coroutine 실행을 I/O Thread에서 수행할 수 있도록 한다.   
**suspend** 키워드는 이 함수가 Coroutine에서 호출되도록 강제한다.   
그리고 다른 suspend 함수에서 호출하거나 Coroutine Builder(ex. launch)를 사용하는 경우에만 호출할 수 있다.
```
class LoginRepository(...) {
    ...
    suspend fun makeLoginRequest(
        jsonBody: String
    ): Result<LoginResponse> {

        // Move the execution of the coroutine to the I/O dispatcher
        return withContext(Dispatchers.IO) {
            // Blocking network request code
        }
    }
}
```

#### Dispatchers
Dispatcher는 Coroutine 실행에 사용되는 Thread를 확인하기 위해 사용한다.   
- **Dispatchers.Main** - 메인 Thread에서 Coroutine을 실행한다. UI와 상호작용을 하거나 빠른 작업을 수행할 때만 사용해야 한다. 예를 들어 UI 업데이트나 LivaData 객체를 업데이트 하는 경우에만 사용한다.
- **Dispatchers.IO** - 메인 Thread 외부에서 Disk 또는 네트워크 I/O를 실행하도록 최적화되어 있다.
- **Dispatchers.Default** - 메인 Thread 외부에서 CPU를 많이 사용하는 작업을 실행하도록 최적화되어 있다. 예를 들어 List를 정렬하거나 JSON을 파싱하는 경우에만 사용한다.

##### Reference:

1. [Kotlin coroutines on Android](https://developer.android.com/kotlin/coroutines)