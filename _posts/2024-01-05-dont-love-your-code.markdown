---
layout: post
title:  "Don't love your code (too much) !"
date:   2024-01-21 23:23:24 +0100
categories: theory 
---
In this post we'll talk about a topic I have been theorising with for a while, it's about the problems of *falling in love with your code or architecture*, and how this can be counterproductive in the long term. As creators, we tend to idealise our own code, it's natural, as our creation we're biased and we consider that it's already better than other options. Let's deep dive into some examples and references and try to valide the theory with some experiences and just have fun reading and discussing.

What does *falling in love with code or/and architecture* mean?
---------------
First, let's take a brief look to what the Oxford Dictionary has to say about love:

>an intense feeling of deep affection
>
>a great interest and pleasure in something

Usually, whenever we have a deep affection relation with something, we don't want to part with it, even if that's the best option with the facts on the table. Despite our rational screaming, we tend to stick with our deep affection. I can relate myself in such situations if we change that something with code or an architectural choice of our own.

In most of the situations that are coming to mind this wasn't the right decision in the long run. An action you have to refrain of, is to stay committed with something in an ever changing world. That being said, you should be ready to change anything at any point in time in the less amount of time possible. Any action opposing it might cause issues in the long run, and becoming emotionally attached to your code and/or architecture is an example of such a problem.

Let's now take a look to some real life moments where I could see that theory applied.


Don't be biased with your decissions.
---------------

It still feels like yesterday, but it was some years ago. We had the amazing opportunity to build from scratch an android app, so here we go!. Was the right moment to set the foundations to build up such ambitious project. Since I was the first developer I decided to use a very simple Model View Presenter architecture with some small common classes that helped to build up basic functionality. Since it was my choice and my code, I can say I felt a bit in love with it and that made me go into some situations when the best option was to move away of it but I still fought to keep it. We ended moving away from that approach, but we delayed the decision just because I liked the simplicity of the solution and it was understandable by me.

Your code will be become obsolete from the very beginning.
---------------

How many times have you implemented your own version of a library or a better version of your library just got released out and you still pretend that your version is way better. Asynchronous wasn't really a thing in the beginnings of android, it was common to build up your own library to help you in that matter. Once the platform became more and more mature, better options started to arise. Even, a better async engine builtin to use out of the box. *It's a question of time that you'll get better options provided by the community or by the platform you are developing in.* As with things that advance so quickly, you shouldn't get too much in love with them. Take a look to the interesting point of view from this [article][tech-debt] , where every code is actually tech debt since the moment it's written.

Love your domain not your implementation.
---------------

This is an interesting story, which it's gonna help me to illustrate another example why you shouldn't love your code. A really talented engineer that I briefly interacted for a project, rewrote the entire project we were working on in three days. True, wasn't the biggest project in the world, but it was complex enough to be something you don't want to rewrite at all. But here the key comes, he knew the logic behind so well that the language just became a mere tool to represent it. Ideally we should reach that level of understanding of our domain that we shouldn't attach to the code itself, but just to the logic. This is gonna help also to be able to share the important parts of our projects to the rest of the team members.


Detach your implementations from feelings.
---------------
Our main mission is to design and build tools that are solving problems, and is our responsibility to have the deepest knowledge we can about the problems we're trying to solve, solving problems should be the driver, not the technology used to solve them. *And since the important part of the equation are the problems, that's the part we should love.* For more problems to solve!


![all you need is love](/assets/love.jpg)


[tech-debt]: https://medium.com/@thecodingteacher_52591/all-code-is-tech-debt-22ac58de05da