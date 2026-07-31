---
title: "typeof streetFood === 'undefined'"
date: 2013-03-09T15:38:00+11:00
draft: false
tags: ["javascript", "node.js", "rails", "ruby"]
---

When DHH made his ['Rails is Omakase'](http://david.heinemeierhansson.com/2012/rails-is-omakase.html) statement last year, [Tom Dale](https://twitter.com/tomdale/status/291788972961701888) tweeted:

![Node is street food ...](/images/street_food.png)

[Rod Vagg](https://twitter.com/rvagg) pointed this statement out to me in February, while we were talking about the lack of conventions in the Node world and whether it was a good or bad thing. In the Ruby world there is a 'Rails Way', lately nearing authoritarianism (although I know of Ruby devs who won't do Rails). In the Node world there isn't an equivalent option.

Instead, and this is what Tom Dale's comic statement touched on, there are numerous coders building libraries that solve similar problems, each with its own approach. Many of them are doing so at a smaller scale, the drive to modularize at the level of the function is a big factor (see the ['substack pattern'](http://www.futurealoof.com/posts/coffee-perspective.html)).

If you develop a Node application you have to be able to understand how to combine these smaller libraries into something larger for your own purpose (notice that I'm refering to ability here, more on that below).

This is one of the best things about Node. It's highly creative. It's the opposite of omakase, it shifts the onus of creating a quality outcome to the programmer.

It is also potentially problematic if you write code professonally and in software teams.

Learning node is frustrating for programmers with a background in synchronous languages (which is probably everyone). Programmers have to adjust their mental image of how to solve problems using asynchrony. It took me some time, for example, to learn that it's better not to use promises and to rely solely on the standard callback pattern alongside some functional programming techniques with something like [async.js](https://github.com/caolan/async) to handle control flow.

They also have to be willing to deal with the poorer parts of Javascript, by now everyone has seen the [WAT](https://www.destroyallsoftware.com/talks/wat) talk about it.

Javascript has been around since [Eich](http://en.wikipedia.org/wiki/Brendan_Eich) started it at Mozilla in 1995. It's the language of the web. Most programmers have a smattering of it but only to the extent that it has a C syntax and they've done some web programming. To do Node well requires understanding [closures](https://developer.mozilla.org/en-US/docs/JavaScript/Guide/Closures) and [prototypal inheritance](https://developer.mozilla.org/en-US/docs/JavaScript/Guide/Inheritance_and_the_prototype_chain). Javascript cannot be a secondary language.

Lastly, if you are working in a team environment, with the absence of convention, you have to be able to find consensus with your colleagues about how to solve problems. This is sometimes not trivial — programmers bear strong opinions on some topics.

Let's summarize the above. To get anywhere with Node you have to be able to

- read and think about source code
- combine libraries to produce your own software
- learn asynchrony and control flow
- work around the cantankerous nature of Javascript
- utilize Javascript as a first class language
- establish enough teamwork to solve an advanced problem

I don't believe that this is every programmer. I think it's only a smaller percentage of them. The ordinariness of the street food metaphor hides the speciality aspect of the programmer who is willing to practice these abilities. On a global scale, this isn't really a problem. The stats of Javascript [usage](https://github.com/languages) conveys this. Locally, though, finding programmers like this isn't a simple task.

Whether this is a curse or a blessing for Node I'm uncertain. Companies have to assess risk. They not only want to own high quality software, they need to know they can readily hire for it and that they can build a productive team. There is certainly a payoff to be had with Node, following the argument of performance, but you need the programming talent to secure it.

In the role of a consultant to an employer discussing how to approach a new project, I'd only recommend Node as a solution for a particular class of commercial problems, particularly in IO bound contexts. I'd mention that Node requires a specialist skillset and, unless the problem required it, the employer might be better off choosing a more mainstream language, like Ruby, Python, Java, whatever.

If someone gave me a programming problem and said I could hire whomever I wanted to form a team, I'd go with Node generally. I enjoy using it and I know some talented programmers. Examples of problems I wouldn't currently solve with Node include distributed computation, math based applications and generic content editing web platforms.

What appeals to me about Node is the smallness of its [API](http://nodejs.org/api/), its basis in functional programming, the fact that I get to craft smaller elements and form a larger, elegantly functioning application and, of course, its speed.

Programmers are constantly trying to invent *the* Rails equivalent in Node. I think this is a mistaken effort. The negative feedback about omakase has been [obvious](http://www.youtube.com/watch?v=E99FnoYqoII). What I think programmers should be creating are open source examples of how to build Node web projects elegantly.

Granted, for Node to find its mainstream footing it will need to build some convention based programming techniques. At the close of [JSConfAU](http://jsconf.com.au/) last year [Mikael Rogers](https://twitter.com/mikeal) spoke about the importance of trying on programming standards and letting them compete to find the best result.

I'm not concerned to make predictions about the future. I am interested in how things arrive at places. If Node can find mainstream adoption via its interest in modularity I know that I'll enjoy working in this space.
