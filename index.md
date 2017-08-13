---
# You don't need to edit this file, it's empty on purpose.
# Edit theme's home layout instead if you wanna make some changes
# See: https://jekyllrb.com/docs/themes/#overriding-theme-defaults
layout: home
---
[Design patterns](http://www.blackwasp.co.uk/gofpatterns.aspx) was a seminal work in codifying elements of resuable object oriented design, but it's dry and hard to understand for someone first venturing into the world of patterns.

Some claim that design patterns are a smell, that they are used to cover up warts in your language or tools. I wouldn't agree. While I do think that many usages of design patterns are wildly overused, just like with any tool you should recognize when is the right time to use it and when to not. I also don't think that many of the patterns in Gang of Four are orthogonal to functional programming.  

This site is an attempt at boiling down some of the Gang of Four nomenclature and to provide simple examples that a reader can extrapolate upon.  In my own original readings of Gang of Four I was constantly confused about the subtleties of one pattern over another.  While Gang of Four is an extremely thorough in analyzing and defining the patterns, I think that c++/smalltalk plus UML diagrams are not necessarily the best way to grok what are extremely valuable tools in the engineers toolbox.

It's important to recognize patterns, whether you agree with them or not, because they are part of our shared professional jargon.  Understanding the nuances between a decorator and an adapter means you can communicate more clearly to your peers.  That said, I don't want to encourage anyone to fall into the classical [`AbstractSingletonProxyFactoryBean`](http://docs.spring.io/spring/docs/2.5.x/javadoc-api/org/springframework/aop/framework/AbstractSingletonProxyFactoryBean.html) joke.  

In the end, like with all tools, patterns are there to help you go fast, go correct, and go forth.