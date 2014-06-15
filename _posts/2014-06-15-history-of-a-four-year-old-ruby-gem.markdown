---
layout: post
title: "The History of a Four-Year-Old Ruby Gem"
date: 2014-06-15 09:32
comments: true
categories: rails
---

This is a story about learning, taking pride in your code, and tastefully sprinkling profanity into boring blog posts. It started out as an introduction to [ClassyEnum](https://github.com/beerlington/classy_enum), but I hate writing code examples, so this is what you get instead.

## The Early Days

Four years ago I was working on a rewrite of a PHP application in Rails. I had no idea what I was doing. Subversion was still something startups used (I keep telling myself that).

We were building software for monitoring commercial-scale solar power plants, and I was working on a feature related to detecting data anomalies like low power conditions and equipment failure. When an anomaly was detected, some sort of action would be taken. Each type of alarm could be individually configured with different settings for what action should occur when the alarm was triggered. Some alarms would just be logged to the database, others would send an email, and a few even required manual verification from a user.

At the time, I was a total noob to application architecture and had been taught in school that redundancy in a relational database was a bad thing and that all data should be as normalized as possible. One of the techniques I learned was to use a "lookup" table in order to keep modifications limited to a single table. In theory this sounds like it's a good idea because it allows you to make changes in one place and have it update everywhere. In practice, obsessing over database normalization is a giant turd of an idea that academics and DBAs preach because that's what expensive textbooks say to do. [Suck it normal forms.](http://en.wikipedia.org/wiki/Database_normalization#Normal_forms).

Here's a simplified example of what our tables looked like:

<img src='/images/alarm-architecture.png' width='400' height='347'>

We had the **alarms** table to define the various types of anomalies being monitoring for a particular project, and the **alarm_actions** table defining what to do when an alarm was activated. The **alarm_actions** table consisted of four or five records with a few combinations of the different settings.

Quick digression - Here's a self survey about your database architecture to figure out whether you really need a lookup table.

1. How often do you [CRUD](http://en.wikipedia.org/wiki/Create,_read,_update_and_delete) records in the lookup table?
2. How much of your application logic depends on the state of that table?
3. If you removed this table, would your application still run?
4. How many records are in this table?

If you answered "never" or "rarely" to the first two questions, "fuck no" to the third and "less than ten" to the fourth, you've got a great candidate for denormalizing that table and replacing with ClassyEnum (or a perhaps a [state machine](https://github.com/beerlington/classy_enum/wiki/ClassyEnum-vs-other-gems)). I'd recommend checking out the [README](https://github.com/beerlington/classy_enum#getting-started--example-usage) for an overview on how to get started with this.

Anyway... we had hired a contractor at this time with real-world software development expertise who basically told us our design sucked. He brought up the concept of enums in Java and how they would be better than dealing with join tables and managing application logic. My first reaction was "no fucking way am I writing Java code in Ruby". My second reaction was "why the fuck is a Java developer touching my Ruby code?". But I'm a rational person so I finally listened to him. After all if it weren't for Java we wouldn't have the Eclipse IDE (trollface).

He put together an initial proof of concept to show us how we could replace the lookup table with an enum class. He also showed us how you could delegate logic to the class (ala polymorphism) instead of having conditionals based on the value. This changed everything. I took his example, put it into a module, and got rid of the lookup table.

For the sake of this being a complete history lesson, [here's a gist of my first iteration](https://gist.github.com/beerlington/0a986548f66822779eb8) which would later become ClassyEnum. I removed the application specific code so it's not 100% complete, but you can tell how much I sucked at Ruby because the `ActiveEnumValue` class explicitly inherits from Object. In my defense, the contractor had it that way in his concept code. In his defense, he was a Java developer.

After going through a few iterations in our application, [ClassyEnum 0.0.1](http://rubygems.org/gems/classy_enum/versions/0.0.1) was released on September 21, 2010 (a Tuesday if you were wondering).

I originally named it ActiveEnum, but when I extracted it to a gem, there was already a [gem with the same name](http://rubygems.org/gems/active_enum). Ironically that one never made it to version 1.0. Also ironically, there's another gem called [enum](http://rubygems.org/gems/enum) which was released exactly 11 days *before* ClassyEnum, based on the exact same Java enum concept. History lessons are fun.

## The Mid-life Crisis

Fast forward to the spring of 2012. Our application was in production and I was a much more seasoned Ruby developer than I had been two years prior. I knew that all classes inherit from Object and you don't need to do this explicity. I was also much more knowledgable in the subject of object oriented programming in general. As controversial a subject as test driven development is these days in the Ruby community, I have to credit it with my understanding of OOP and specifically concepts like [encapsulation](http://en.wikipedia.org/wiki/Encapsulation_(object-oriented_programming)) and [designing interfaces](http://en.wikipedia.org/wiki/Protocol_(object-oriented_programming)). I was also very interested in DSLs at the time and what makes one "better" than another.

With my new found skills, I decided to rewrite ClassyEnum from the ground up and think about the experience from the perspective of someone who is using it for the first time. There were a few goals I had in mind: 1) It had to be simpler and more intuitive, 2) compatible with every major version of Rails, and 3) future proof to require less changes when new versions of Rails came out.

At a time when my favorite gems were getting more and more bloated ([ahem](https://github.com/thoughtbot/factory_girl/blob/master/GETTING_STARTED.md)), I wanted to simplify. I cut out features such as built-in Formtastic support, and reduced the DSL to a single model declaration. I even declined pull requests adding new features I didn't want, despite the hard work that people put into their code. Telling someone why their idea sucks in a courteous way is not always easy.

I made the mistake of releasing 2.0 too soon and was not happy with the results. I ended up cutting even more out and releasing 3.0 three months later. I take [semantic versioning](http://semver.org/) seriously, but was not experienced enough to know when I was satisfied with the results.

I really like this quote from the Semantic Versioning 2.0 FAQ:

> Having to bump major versions to release incompatible changes means you'll think through the impact of your changes, and evaluate the cost/benefit ratio involved.

I now spend a lot more time letting releases "ferment" before making them final. This is why we have betas and release candidates.

## The Latter Days

Fast forward another two years to today. I'm no longer working on the monolithic application I started back in 2009. These days I am contracting and typically work on one or two different Rails applications a month. However, for the last year nearly every single application I have worked on has included ClassyEnum. According to [rubygems.org](https://rubygems.org/gems/classy_enum) it has been installed over 80,000 times and had 45 releases. Yet despite the history and popularity, only 38 issues have been opened in four years (all but two are closed). The reason I mention this is because I take pride in releasing stable code that is well tested and documented. I want it to be easy for new comers to pick up, and I take every bug report seriously.

I believe my strict policy of only adding features that have inherent value and cutting ones that do not has helped me sustain development on it. While new releases have slowed in recent years, its usage has only increased, and I see this as a sign of maturity and stability. I don't want to add features just for the sake of adding them because the person who has to maintain them is me.

I encourage you to think about a project you're working on, whether it be open source or otherwise, and ask yourself how you can make the experience better for yourself and your fellow developers - today, next year, and well beyond its expected life.
