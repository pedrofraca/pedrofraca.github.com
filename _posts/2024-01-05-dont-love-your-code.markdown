---
layout: post
title:  "Don't love your code (too much) !"
date:   2024-01-05 00:38:24 +0100
categories: theory 
---
In this post we'll talk about a topic I have been theorising with for a while, it's about the problems of *falling in love with your code or achictecture*, and how this can be counterproductive in the long term. As creators, we tend to idealize our own code, it's natural, as our creation we're biased and we consider that it's already better than other options. Let's deep dive into some examples and references and try to valide the theory with some experiences and just have fun reading and discussing.


What does *falling in love with code or/and architecture* mean?
---------------
First, let's take a brief look to what the Oxford Dictionary has to say about love:

>an intense feeling of deep affection
>
>a great interest and pleasure in something

Usually, whenever we have a deep affection relation with something, we don't want to part with it, even if that's the best option with the facts on the table. Despite our rational screaming, we tend to stick with our deep affection. I can relate myself in such situations if we change that *something* with code or an architectural choice of our own.

In most of the situations that are coming to mind this wasn't the right decision in the long run. An action you have to refrain of, is to stay commited with something in an ever changing world. That being said, you should be ready to change anything at any point in time in the less amout of time possible. Any action opposing it might cause issues in the long run, and becoming emotionally attached to your code and/or achitecture is an example of such a problem.

Let's now take a look to some cases where I could see that theory applied.

Examples
---------------

- It still feels like yesterday, but it was some years ago. We had the amazing opportunity to build from scratch an android app, so here we go!. Was the right moment to set the foundations to build up such ambitious project. Since I was the first developer I decided to use a very simple Model View Presenter architecture with some small common classes that helped to build up basic functionality. Since it was my choice and my code, I can say I felt *a bit* in love with it and that made me go into some situations when the best option was to move away of it but I still fought to keep it. 

- Library for async in android:

- Rewrite whole project in three days:


Detach from your feelings
---------------



![Android app UI flow](/assets/appFlow.png)

Since the tour of 2015 is over (which means no more updates on the data) and I’m getting the info from my API in two calls (one for the stages and another one for the classification per stage) I’d like to just do the API call when starting the app and persists it locally on the device so I don’t need to ask for it again and the navigation between the list and the details is seamless.

We wanted to follow the [clean architecture][clean-arch] approach to define 

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