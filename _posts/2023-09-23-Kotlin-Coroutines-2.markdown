---
layout: post
logo: "assets/images/15balloon.png"
navigation: True
cover: "assets/images/cover1.jpg"
title: "Kotlin Coroutines 정리: 2"
author: "15balloon"
date: 2023-09-23 23:29:52 +0900
tags: android
categories: 15balloon
subclass: "post tag-test tag-content"
---

#### Launch & Async
- launch - 새로운 Coroutine을 시작하며 호출자에게 결과를 반환하지 않는다.
- async - 새로운 Coroutine을 시작하며 *await*라는 suspend 함수로 결과를 반환할 수 있다.

async는 Coroutine 내부에서만 사용하거나 suspend 함수 내에서 작업을 병렬적으로 처리할 때 사용한다.   
suspend 함수 내에서 시작되는 Coroutine은 함수가 반환되면 중지되어야 한다.   
그러므로 반환 전에 Coroutine이 완료되도록 보장해야 한다.   
이를 위해 여러 Coroutine을 실행할 수 있는 **coroutineScope**를 정의할 수 있다.   
그리고 **await()** , **awaitAll()** 을 사용하여 함수가 반환되기 전에 Coroutine이 완료되도록 보장할 수 있다.   
아래 코드는 async를 통해 시작한 Coroutine을 await 함수를 사용하여, 실행된 Coroutine이 결과를 반환할 때 까지 기다린다.   
그러나 await 함수를 사용하지 않았더라도 coroutineScope Builder는 모든 Coroutine이 완료될 때 까지 기다린다.   
또한 coroutineScope는 Coroutine이 발생시키는 Exception(예외)를 호출자에게 전파한다.   
await()를 사용한 코드는 다음과 같다.   

```
suspend fun fetchTwoDocs() =
    coroutineScope {
        val deferredOne = async { fetchDoc(1) }
        val deferredTwo = async { fetchDoc(2) }
        deferredOne.await()
        deferredTwo.await()
    }
```

awaitAll()을 사용한 코드는 다음과 같다.   

```
// called on any Dispatcher (any thread, possibly Main)
suspend fun fetchTwoDocs() =
    coroutineScope {
        // fetch two docs at the same time
        val deferreds = listOf(
            // async returns a result for the first doc
            async { fetchDoc(1) },
            // async returns a result for the second doc
            async { fetchDoc(2) }
        )

        // use awaitAll to wait for both network requests
        deferreds.awaitAll()
    }
```

#### CoroutineScope
CoroutineScope는 launch 혹은 async로 생성된 Coroutine을 추적한다.   
실행 중인 Coroutine은 *scope.cancel()* 을 호출하여 취소할 수 있다.   
ViewModel이나 Lifecycle은 자체 CoroutineScope를 제공한다.   
ViewModel은 viewModelScope가 있고, Lifecycle에는 lifecycleScope가 있다.   
그러나 CoroutineScope는 Coroutine을 실행하진 않는다.   
*scope.cancel()* 에 의해 Coroutine이 한 번 취소되면 다시는 해당 scope에서 Coroutine을 생성할 수 없다.   
즉, *scope.cancel()* 은 해당 클래스가 destroy된 경우에만 사용해야 한다.   
ViewModel의 경우 *onCleared()* 메서드에서 자동으로 취소된다.   

```
class ExampleClass {

    // Job and Dispatcher are combined into a CoroutineContext which
    // will be discussed shortly
    val scope = CoroutineScope(Job() + Dispatchers.Main)

    fun exampleMethod() {
        // Starts a new coroutine within the scope
        scope.launch {
            // New coroutine that can call suspend functions
            fetchDocs()
        }
    }

    fun cleanUp() {
        // Cancel the scope to cancel ongoing coroutines work
        scope.cancel()
    }
}
```

#### Job
**Job**은 Coroutine을 다루기 위해 사용한다.   
launch나 async로 만들어진 Coroutine은 Coroutine을 고유하게 식별하고 lifecycle을 관리하는 Job 인스턴스를 반환한다.   
다음 예시 코드와 같이 Job을 CoroutineScope에 전달하여 lifecycler을 관리할 수 있다.   

```
class ExampleClass {
    ...
    fun exampleMethod() {
        // Handle to the coroutine, you can control its lifecycle
        val job = scope.launch {
            // New coroutine
        }

        if (...) {
            // Cancel the coroutine started above, 
            // this doesn't affect the scope
            // this coroutine was launched in
            job.cancel()
        }
    }
}
```

#### CoroutineContext
CoroutineContext는 다음 요소를 사용하여 Coroutine의 동작을 정의한다.   
- Job: Coroutine의 lifecycle 제어
- CoroutineDispatcher: 적절한 Thread에 작업 전달
- CoroutineName: Coroutine의 이름. 디버깅 시 사용
- CoroutineExceptionHandler: uncaught Exception 처리

scope 내에서 만들어진 Coroutine은 새로운 Job 인스턴스가 새 Coroutine에 할당되고 CoroutineContext 요소는 해당 scope에서 상속된다.   

```
class ExampleClass {
    val scope = CoroutineScope(Job() + Dispatchers.Main)

    fun exampleMethod() {
        // Starts a new coroutine on Dispatchers.Main 
        // as it's the scope's default
        val job1 = scope.launch {
            // New coroutine with CoroutineName = "coroutine" (default)
        }

        // Starts a new coroutine on Dispatchers.Default
        val job2 = scope.launch(
            Dispatchers.Default + CoroutineName("BackgroundCoroutine")
            ) {
                // New coroutine with CoroutineName 
                // = "BackgroundCoroutine" (overridden)
        }
    }
}
```

#### Recommendation
##### Inject Dispatcher   
새 Coroutine을 만들거나 withContext를 호출할 때 Dispatchers를 하드코딩 하지 않는 것이 좋다.   
하드코딩 시 테스트가 어렵기 때문이다.   
##### 안전한 suspend 함수   
suspend 함수는 메인 Thread에서 호출하기 때문에 main-safe 해야 한다.   

```
class NewsRepository(private val ioDispatcher: CoroutineDispatcher) {

    // As this operation is manually retrieving the news from the server
    // using a blocking HttpURLConnection, it needs to move the execution
    // to an IO dispatcher to make it main-safe
    suspend fun fetchLatestNews(): List<Article> {
        withContext(ioDispatcher) { }
    }
}

// This use case fetches the latest news and the associated author.
class GetLatestNewsWithAuthorsUseCase(
    private val newsRepository: NewsRepository,
    private val authorsRepository: AuthorsRepository
) {
    // This method doesn't need to worry about moving the execution of the
    // coroutine to a different thread as newsRepository is main-safe.
    // The work done in the coroutine is lightweight as it only creates
    // a list and add elements to it
    suspend operator fun invoke(): List<ArticleWithAuthor> {
        val news = newsRepository.fetchLatestNews()

        val response: List<ArticleWithAuthor> = mutableEmptyList()
        for (article in news) {
            val author = authorsRepository.getAuthor(article.author)
            response.add(ArticleWithAuthor(article, author))
        }
        return Result.Success(response)
    }
}
```
##### ViewModel에서의 Coroutine 생성   
View에서 Coroutine을 만들고 ViewModel에서 suspend 함수를 사용하지 말아야 한다.   
이러한 경우 테스트가 어렵고 configuration 변경에 수동으로 처리해야 한다.   
ViewModel에서 Coroutine을 생성한다면 단위 테스트를 진행할 수 있고, viewModelScope에서 실행된 작업은 configuration 변경에도 자동으로 유지된다.   
##### GlobalScope 금지   
GlobalScope는 어느 Job에도 종속되지 않고, Application의 lifecycle을 따른다.   
그러나 테스트가 어렵고 실행을 제어할 수 없으며 공통 CoroutineContext를 가질 수 없다.   

```
// DO inject an external scope instead of using GlobalScope.
// GlobalScope can be used indirectly. Here as a default parameter makes sense.
class ArticlesRepository(
    private val articlesDataSource: ArticlesDataSource,
    private val externalScope: CoroutineScope = GlobalScope,
    private val defaultDispatcher: CoroutineDispatcher = Dispatchers.Default
) {
    // As we want to complete bookmarking the article even if the user moves
    // away from the screen, the work is done creating a new coroutine
    // from an external scope
    suspend fun bookmarkArticle(article: Article) {
        externalScope.launch(defaultDispatcher) {
            articlesDataSource.bookmarkArticle(article)
        }
            .join() // Wait for the coroutine to complete
    }
}

// DO NOT use GlobalScope directly
class ArticlesRepository(
    private val articlesDataSource: ArticlesDataSource,
) {
    // As we want to complete bookmarking the article
    // even if the user moves away from the screen,
    // the work is done creating a new coroutine with GlobalScope
    suspend fun bookmarkArticle(article: Article) {
        GlobalScope.launch {
            articlesDataSource.bookmarkArticle(article)
        }
            .join() // Wait for the coroutine to complete
    }
}
```

##### Reference:

1. [Kotlin coroutines on Android](https://developer.android.com/kotlin/coroutines)