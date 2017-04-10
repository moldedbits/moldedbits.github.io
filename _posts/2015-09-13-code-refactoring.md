---
layout: post
title:  "Code Refactoring"
date:   2015-09-13 11:11:11
author: aashish
categories: Technical Android
comments: true
---

No one can argue about the importance of good code architecture and clean code. Every good programmer or a team of experienced developers gives special attention to these as they understand how much effect this has on product quality and their productivity. Our team at [moldedbits][moldedbits] is no exception.

But one aspect of software development is generally neglected by developers and that is __Code refactoring__. Some engineers do not understand its importance and other just do not think it is worth their time.

#### So what Is Refactoring?


Refactoring is the process of changing a software system in such a way that it does not alter the external behavior of the code yet improves its internal structure.


#### And why it is important?

Code quality is function of time. Over time the code will be modified and the integrity of the system, its structure according to that design, gradually fades. The code slowly sinks from engineering to hacking. So no matter how well designed initially a system is, if it is not maintained properly over the time, it will loose its true beauty.

#### OK. But why people ignore this?

1. Refactoring is risky. It requires changes to working code that can introduce subtle bugs. Refactoring, if not done properly, can set you back days, even weeks. So to avoid digging your own grave, refactoring must be done systematically.

2. There is always a chance that you are running out of time because a deadline is near and picking up this another task of refactoring simply means a missed deadline.

3. It is difficult to convince a project manager that i'll not be adding new features and also not fixing any bug, instead i'll be just rearranging my code. (At least i faced these kind of situations.)

4. And why should i change my code when it is working perfectly?

#### Hmm. These are good reasons. So why should i do this? Convince me.

1. A poorly designed system is hard to change. Hard because it is hard to figure out where the changes are needed. If it is hard to figure out what to change, there is a strong chance that the programmer will make a mistake and introduce bugs.

2. When you find you have to add a feature to a program and the program's code is not structured in a convenient way it becomes harder to extend functionality.

3. Well...Actually i would say all those reasons which are given in favour of clean code can be added here like maintainability, debugging, code quality and many more.

![Clean Code] [clean-code]

#### You got my attention. So when to do refactoring?

#####Rule of Three
<img src="http://moldedbits.github.io/assets/images/r1.png" alt="Drawing" style="width: 200px;"/>

1. When you do something for the first time, you are just doing it to get done.
2. When you do something similar for the second time, you cringe at having to repeat but do the same thing anyway.
3. When you do something for the third time, you start refactoring.

#####When Adding a new Feature
<img src="http://moldedbits.github.io/assets/images/r2.png" alt="Drawing" style="width: 200px;"/>

1. Refactoring helps to understand other people's code. If it becomes necessary to add a feature to unclear code, refactoring it makes things obvious for you and those coming after you.
2. Refactoring makes it easier to add features. New features are added more smoothly and with less effort.

#####When fixing a bug
<img src="http://moldedbits.github.io/assets/images/r3.png" alt="Drawing" style="width: 200px;"/>

1. Bugs resemble cockroaches: they love to live in the darkest, mustiest places of your code. Clean up your code and the errors will practically find themselves.
2. Managers appreciate proactive refactoring, since it eliminates the need to create special refactoring tasks later. Happy bosses mean happy programmers!

#####When reviewing code
<img src="http://moldedbits.github.io/assets/images/r4.png" alt="Drawing" style="width: 200px;"/>

1. Review may be your last chance to tidy up your code before it becomes available to the public.
2. The best way to review is together with the author of the code. You propose changes and decide, together with the author, how difficult it will be to implement various refactoring techniques. Small changes can be made right on the spot.

#### And how it is done?
![process][process]

Refactoring is best performed as a series of minor changes. Each change improves the existing code slightly while keeping it in functional condition.

Three key components of correct refactoring:

1. No new functionality is created during refactoring.
2. All existing tests should be successfully passed after refactoring.
3. The code becomes simpler after refactoring.

#### Great. Tell me more, Please.

Well.. There are many good books available which can be referred to like [Refactoring: Improving the Design of Existing Code](http://www.amazon.com/Refactoring-Improving-Design-Existing-Code/dp/0201485672/ref=sr_1_1?s=books&ie=UTF8&qid=1442119326&sr=1-1&keywords=refactoring+improving+the+design+of+existing+code). Hope it helps.

Happy coding!

The moldedbits Team


[moldedbits]: http://moldedbits.com
[clean-code]: http://moldedbits.github.io/assets/images/stack-en.png
[process]: http://moldedbits.github.io/assets/images/process-en.png

{% if page.comments %}
{% include disqus.html %}
{% endif %}
