---
layout: post
logo: "assets/images/15balloon.png"
navigation: True
cover: "assets/images/cover1.jpg"
title: "Jetpack Compose Navigation 정리"
author: "15balloon"
date: 2023-09-21 23:46:57 +0900
tags: android
categories: 15balloon
subclass: "post tag-test tag-content"
---

이 포스트는 Jetpack Compose Navigation에 대한 간단한 내용을 기술한다.   
자세한 내용은 하단의 Reference 참조.   

#### Setup
Compose Navigation을 사용하려면 build.gradle(:app)에 아래의 dependency를 추가해야 한다.   
```
dependencies {
    <!-- 2023.09.21 기준 최신 버전은 2.7.2 -->
    implementation 'androidx.navigation:navigation-compose:2.7.2'
}
```

#### NavController
NavController는 Navigation 컴포넌트의 중심 API이다.   
Stateful이며 앱의 화면과 각 화면 상태를 구성하는 Composable의 Back Stack을 추적한다.   
NavController는 이를 참조해야 하는 모든 Composable이 접근 가능한 곳에 만들어야 한다.   
```
val navController = rememberNavController()
```

#### NavHost
NavHost는 Navigation Graph와 NavController를 연결한다.   
NavController는 반드시 단일 NavHost와 연결되어야 한다.   
Composable 간의 이동(Navigation)을 수행하면 NavHost의 콘텐츠는 자동으로 Recompose 된다.   
Navigation Graph에 속한 Composable들은 각각의 route와 연결된다.   
route는 Composable의 경로를 정의하는 String이다.   

NavHost를 만들려면 Graph의 startDestination도 필요하다.   
```
NavHost(navController = navController, startDestination = "profile") {
    composable("profile") { Profile(/*...*/) }
    composable("friendslist") { FriendsList(/*...*/) }
}
```

#### Navigate
navigate() 메서드를 사용하여 Composable 간의 이동을 할 수 있다.   
이 메서드는 새로운 destination을 Back Stack에 추가한다.
```
navController.navigate("friendslist")
```

#### Arguments
Composable 간의 매개변수 전달을 할 수 있다.   
전달된 매개변수는 NavBackStackEntry에서 인수를 추출해서 사용한다.   
복잡한 데이터 객체를 전달하지 않고 고유 식별자 또는 ID와 같은 필요한 최소 정보를 전달하는 것이 좋다.   
그 이유는 Navigation Pass Data를 참고.
```
NavHost(startDestination = "profile/{userId}") {
    composable("profile/{userId}") { backStackEntry ->
        Profile(
            navController, 
            backStackEntry.arguments?.getString("userId")
        )
    }
}
```

매개변수는 선택적으로 전달할 수도 있다.   
쿼리 문법을 사용해야 하고, defaultValue나 nullability = true로 설정되어 있어야 한다.   
```
composable(
    "profile?userId={userId}",
    arguments = 
        listOf(navArgument("userId") { defaultValue = "user1234" })
) { backStackEntry ->
    Profile(navController, backStackEntry.arguments?.getString("userId"))
}
```

#### Deep links
[딥 링크에 대한 설명](https://developer.android.com/jetpack/compose/navigation?#deeplinks)

#### Nested Navigation
Nested Graph로 앱 UI 특정 흐름을 모듈화할 수 있다.   
NavGraphBuilder의 확장 메서드로 만들어서 사용할 수 있다.   
```
fun NavGraphBuilder.loginGraph(navController: NavController) {
    navigation(startDestination = "username", route = "login") {
        composable("username") { ... }
        composable("password") { ... }
        composable("registration") { ... }
    }
}
```

```
NavHost(navController, startDestination = "login") {
    loginGraph(navController)
}
```

##### Reference:

1. [Jetpack Compose Navigation](https://developer.android.com/jetpack/compose/navigation)
2. [Navigation Pass Data](https://developer.android.com/guide/navigation/navigation-pass-data?#supported_argument_types)