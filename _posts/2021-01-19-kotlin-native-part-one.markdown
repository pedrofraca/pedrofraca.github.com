---
layout: post
title:  "Kotlin native clean code (Part I)"
date:   2021-01-19 00:38:24 +0100
categories: kotlin android ios native
---
This is the first post of a series where I’ll be talking about the usage of kotlin native for sharing code between Android, iOS, and potentially backend as well. Note to the reader: this is not a finalized project and I’m working on it in small steps, this might cause that not everything is perfect, and things can change as I’m working on them. Feel free to contact me if you find anything that doesn’t work as expected.

{:refdef: style="text-align: center;"}
![Complete arch diagram](/assets/tour-app.png){:height="400px" width="400px"}
{: refdef}

Art made by [monkik][monkik-web] from [flaticon][flaticon-web] 


Overal Architecture explained
---------------

The main objective of the project as said is to use [Kotlin native][kotlin-native] to share some business logic between iOS and Android. We’ll implement the same app in both platforms as well as implementing new features in coming posts where we’ll analyze the cons/pros and how quickly new features can be developed. Please take a look at the following diagram where you can see what’s the intended final architecture.

{:refdef: style="text-align: center;"}
![Complete arch diagram](/assets/overallArch.png)
{: refdef}

Creating the shared module
---------------

We all know that the Tour of France 2015 was not probably one of the best of the last years, but I still remember watching some stages and having fun, supporting my favorite rider. That year the first version of the android design library was released and I thought that creating an app to learn how to use it can be a great plan for the summer, so I did.

As part of that learning exercise, I also created a small Google App Engine backend in python capable of scrap some webs to obtain the latest results of the stage and serve that info using a simple and not well-designed REST API. We’ll revisit this in the future and move this API to the 21st century.

Five years later, I thought that would be great to give a try to a technology I always wanted to try. Kotlin native. On the paper sounds cool, you just write the code once and you can run it on a few platforms, mmm wait! haven’t heard that before?. Anyhow my test project came to my mind and I started to create a common library that I can reuse later in a potential iOS app or even in a backend. The app is rather simple, once you start the app you get a list with all the stages, if you click on it you’ll see the details of the stage and the profile, and in a third navigation level, you can see all the classifications for that stage (stage, general, mountain, regularity, and teams).

![Android app UI flow](/assets/appFlow.png)

Since the tour of 2015 is over (which means no more updates on the data) and I’m getting the info from my API in two calls (one for the stages and another one for the classification per stage) I’d like to just do the API call when starting the app and persists it locally on the device so I don’t need to ask for it again and the navigation between the list and the details is seamless.

We wanted to follow the [clean architecture][clean-arch] approach to define dependencies and modules. That’s why we’ve exported the first two modules to the kotlin native project, domain, and data. The domain is the module where we’re including the raw models that are important for our app, in this case, we’ve models for the Stage, Classification. In Data we’ve created the repositories which only know about the data, the main mission here is to be a flow of data that comes from the data sources. This module exports a few interfaces for the data sources that will be implemented by th 

![Clean Architecture Diagram](/assets/cleanArchitecture.jpg)

As the article from Uncle Bob says, we're always following the "Dependency rule", This rule says that source code dependencies can only point inwards.

[Here] [tour-app-native-repo] you can find the repository for the native module, which is going to be shared between the both platforms. The repository has actions and GitHub packages in order to expose the version to android, the iOS version gets exposed in another [repository] [tour-app-pod-repo] as a Pod. Now the process of uploading a newer Pod version is a manual one, we’ll try to automatize that with GitHub actions so no manual interaction is required.

Let’s go a bit more in detail to the Data module and see what are we doing in here. As said, this module contains some data sources that are completely generic, I wanted to abstract completely from where the data comes from and create something that can just represent read or read/write data sources. Something like this:

{% highlight kotlin %}
package io.github.pedrofraca.data.datasource

interface ReadOnlyDataSource<T> {
    fun getAll(): List<T>
}

interface ReadOnlyDataSourceWithFilter<T, S> : ReadOnlyDataSource<T>{
    fun get(param : S): T
}

interface WriteDataSource<T> : ReadOnlyDataSource<T> {
    fun save(item: T)
}

interface WriteDataSourceWithFilter<T, S> : ReadOnlyDataSourceWithFilter<T, S> {
    fun save(item: T)
}
{% endhighlight %}

These interfaces are going to be exposed, so any of the platforms can implement each of the datasources based on the needs and inject them in the next layer, the repositories. Let's take a look to the Stages repository source code: 

{% highlight kotlin %}]
package io.github.pedrofraca.data.datasource.stage

import io.github.pedrofraca.data.datasource.ReadOnlyDataSource
import io.github.pedrofraca.data.datasource.WriteDataSource
import io.github.pedrofraca.domain.model.StageModel

/**
 * Repository class to manage stages. Which basically uses two data sources.
 * One to retrieve the Stages and the other one to persists them.
 */

open class StagesRepositoryImpl(private val readDataSource: ReadOnlyDataSource<StageModel>,
                                private val persistenceDataSource: WriteDataSource<StageModel>) : StageRepository {

    override fun refresh(): List<StageModel> {
        val stages = readDataSource.getAll()
        stages.forEach {
            persistenceDataSource.save(it)
        }
        return stages
    }

    override val stages: List<StageModel>
        get() = persistenceDataSource.getAll()
}
{% endhighlight %}

In this case, it has only two methods, one of them is refreshing the data (getting from the API and saving to the persistence layer) and another is getting the currently stored in the persistence layer. We’ll introduce the asynchronous layer in each of the frameworks later, the repository is designed synchronously. The constructor expects to receive two data sources, one for read the data and the other one to read and persist it. As part of this module, we’ve also included unit tests to assert that the repository works as expected. We’re using [mockk][mockk-web] as a mocking library for kotlin.

{% highlight kotlin %}

class StageRepositoryTest {

    private val db = mockk<WriteDataSource<StageModel>>(relaxed = true)
    private val api = mockk<ReadOnlyDataSource<StageModel>>(relaxed = true)
    private val repo = StagesRepositoryImpl(api, db)

    @Test
    fun `test empty response doesn't save anything`() {
        every { api.getAll() } returns emptyList()

        repo.refresh()

        verify { api.getAll() }
        verify(exactly = 0) { db.save(any()) }
    }

    @Test
    fun `test non empty response saves items`() {
        every { api.getAll() } returns listOf(StageModel("This is the first stage",
            stage = 1,
            profileImgUrl = ""))

        repo.refresh()

        verify { api.getAll()}
        verify(exactly = 1) { db.save(any()) }
    }
}

{% endhighlight %}

There’s also something important that happens on this module, which is freezing the Repository for iOS. This is needed since kotlin native doesn’t allow to access objects that are generated in another [thread][kotlin-thread]. Since we’re moving all the thread management to the next layer we need to do so in this data module, to do so I’ve created a factory that does the building of the repository for each of the variants (jvm and iOS), let’s check the iOS implementation:

{% highlight kotlin %}
package io.github.pedrofraca.data.datasource.stage

import io.github.pedrofraca.data.datasource.ReadOnlyDataSource
import io.github.pedrofraca.data.datasource.WriteDataSource
import io.github.pedrofraca.domain.model.StageModel
import kotlin.native.concurrent.freeze

actual class StagesRepositoryFactory actual constructor() {
    actual fun build(
        apiDataSource: ReadOnlyDataSource<StageModel>,
        databaseDataSource: WriteDataSource<StageModel>
    ): StagesRepositoryImpl {
        val repoImpl = StagesRepositoryImpl(apiDataSource, databaseDataSource)
        repoImpl.freeze()
        return repoImpl
    }
}
{% endhighlight %}

Connecting it with the apps
---------------

Now the next step is to use all that logic from the apps. Once included the dependencies, we can access the interfaces and the models that these two layers are exporting to the outside world. In Android we’re going to create three modules, here the [repo][android-repo]:

- Framework : This layer is going to be the one implementing the data sources. We’re going to use room for persistence on android and retrofit as the library to connect to the API. We wanted to isolate the API calls in another module, called API.

- App: The app is going to contain only UI code and the ViewModels. We’ve decided to use ViewModels to export the data to the UI using LiveData. For all the orchestration of the data sources, we’re using Rx. Following a simple premise, the first one of the data sources returning data is the one that gets exposed.

![Android app dependencies](/assets/androidDeps.png)

On iOS we created a simpler dependency management and everything is on the same project as of now, we also used [Rx][rx-swift] in order to manage the orchestration of the datasources and just UrlSession for the API connection. Here you can take a look to the [repository] [tour-app-ios-repo]. 

{:refdef: style="text-align: center;"}
![iOS app](/assets/iosApp.png)
{: refdef}

[SwiftUI] [swift-ui] is used to build the UI for iOS, the UI is inspired in the App Store design and implemented using what's explanied in the following [video][app-store-tutorial], and view model approach as well to translate the information from the datasource to the UI, here a fragment of the orchestration that happens in the view model in swift.

{% highlight swift %}

Observable.concat(databaseObservable, networkObservable)
            .filter { it in !it.isEmpty }
            .first()
            .asObservable()
            .subscribeOn(concurrentBackground)
            .observeOn(main)
            .subscribe(onNext: { it in self.stages = it ?? []},
                       onCompleted: { [weak self] in
                        self?.state = .done
            }).disposed(by: disposeBag)

{% endhighlight %}

And that's all for this first post, in the next ones we will cover a few topics like:

- Refactoring the server in kotlin using quarkus and deploying this to our own kubernetes cluster. 
- Implementing a search functionality step by step and analyze what are the pros and cons.
- Try different approaches, like implementing the networking layer as well in Kotlin native and share it within the platforms.

Please, feel free to write comments in this blog post as well as raise any issue you might found on each of the repositories. PR's are more than welcome. :) 


[tour-app-native-repo]: https://github.com/pedrofraca/tour-app-native
[tour-app-pod-repo]: https://github.com/pedrofraca/tour-app-data-pod
[tour-app-ios-repo]: https://github.com/pedrofraca/tour-app-ios
[kotlin-thread]: https://stackoverflow.com/questions/60180941/uncaught-kotlin-exception-kotlin-native-incorrectdereferenceexception-illegal
[kotlin-native]: https://kotlinlang.org/docs/reference/native-overview.html 
[swift-ui]: https://developer.apple.com/xcode/swiftui/
[mockk-web]: https://mockk.io/
[app-store-tutorial]: https://www.youtube.com/watch?v=o31ZzGuW-1M&list=WL&index=50&t=371s
[clean-arch]: https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html
[rx-swift]: https://github.com/ReactiveX/RxSwift
[monkik-web]: https://www.flaticon.com/authors/monkik
[flaticon-web]: www.flaticon.com
[android-repo]: https://github.com/pedrofraca/tourapp/tree/master/TourApp