---
layout: post
title: "How We Won Hack|VT"
date: 2012-10-21 20:21
comments: true
categories: vermont software
---

This post describes how our team, **Datamorphosis**, won the [HackVT hackathon](http://hackvt.com/), and the strategy we used to ensure that if we didn’t win, we’d at least be proud of what we built. Read on to find out how we formed our team, brainstormed an idea, researched and planned our application, executed during the event, which technologies we used, and how we presented our winning application: **The Vermont Business Landscape**.

![Insert App screenshot](http://petebrown-pics.s3.amazonaws.com/public/vermont-business-landscape-datamorphosis.png)

## Forming a Team

Hackathons are not just about writing code. They are about solving a problem in the most succinct way possible, and effectively communicating the problem and solution in a remarkably short period of time. Having the right team is critical to accomplishing that goal. There are very few people in this world who alone can brainstorm an idea, aggregate complex data, build APIs, design a usable interface, and present their work in front of a judging audience. The rest of us are mere mortals. Forming a solid team is about understanding the strengths and weaknesses of each member, and how they will fit together.

### Our Team:

* [Adam Bouchard](http://adambouchard.info)
* [Alan Peabody](https://twitter.com/alanpeabody)
* [Katie McCurdy](http://katiemccurdy.com/)
* [Patrick Berkeley](http://www.linkedin.com/in/patrickberkeley)
* [Peter Brown](https://twitter.com/beerlington) (me!)

Combined, our team was highly skilled in web design and development, data analysis, user experience, and communication; precisely the skills needed to succeed in a hackathon. Almost every member of our team had previously worked with another member at some point in the past. We understood each other’s strengths and weaknesses and knew exactly how each person could contribute. This allowed us to trust one another and stay focused on the tasks that we were most qualified for.

## Brainstorming and Planning

When selecting our data set, there was one theme that seemed to resonate with our team: _Incorporating data that was challenging to aggregate and analyze_. The reason for this is that many successful business models are built around simplifying a tedious task. Our team believes in the value of small businesses to Vermont’s economy, and in doing research, we discovered that the state has a need for aggregating business data from disparate data sources. The [Community & Economic Development Office](http://burlingtonvt.gov/cedo/) receives frequent inquiries from prospective new business owners as to the health of various sectors throughout Vermont.

Once we had a vague idea that we wanted to build a tool to assist new business owners, we began to research and plan our application. Our team met a few times during the week leading up to the hackathon, and used [Flowdock](https://flowdock.com/) to discuss ideas (even when we were in the same room).

### Here are a few of the areas that we researched:


* **Market need** - If you’re trying to solve a problem for someone other than yourself, find a real person who has this problem. Talk to them and see how passionate they are about it. In our case, the person we spoke with at the Community & Economic Development Office was ecstatic about our proposed solution. This validated our idea and gave us confidence going into to the competition.
* **Judges** - Information about each judge was available on the [HackVT website](http://hackvt.com/#details). We discussed as a team what sort of things they might be looking for, and planned to account for this in our application. For example, [Justin Cutroni](http://cutroni.com/blog/) works on the Google Analytics team, and we assumed that analyzing and simplifying complex data was something he is passionate about.
* **Technology** - Spending time up front discussing and researching technologies is well worth the effort. After choosing an idea, we made a list of all the potential tools we might need to build it. Members of our team had overlapping skill sets so we were able to make some important decisions before the competition began.
* **How to win a hackathon** - We even spent some time researching [how other people won hackathons](http://news.ycombinator.com/item?id=4345295).

## Execution

Given the amount of upfront work we did, we were able to start the hackathon with confidence and direction. We knew who was doing what, and essentially hit the ground running. We made a couple mistakes along the way and had to pivot a few times, but overall we were able to stay focused on our goal and finish within the allotted time.

![Datamorphosis team at work](http://petebrown-pics.s3.amazonaws.com/public/datamorphosis-at-work.jpg)

### What we learned:

* **Start early** - Though the hackathon didn’t officially kick off until 6PM, they allowed teams to set up and start coding at 3pm. These early hours in the competition are critical. Everyone is awake and can make reasonable decisions, and being there early makes things a little less stressful.
* **Avoid perfection** - This is a hackathon, and it’s ok to write hacky code. People who know me or have seen some of [my presentations](https://speakerdeck.com/beerlington) know that I’m a stickler for good code. Building a production application for your customers is one thing, but hacking something in 24 hours for a three minute presentation is another. Put aside your pride and just get it done.
* **Focus on the core** - Since you will likely run into unexpected hiccups along the way, focus on building the core first and iterate on it. Will your application still function and be presentable without feature X? If so, it’s a low priority. In our case, showing data on a time-lapse map was the core of our application. Without this, it would not have met our goals. We tracked a list of our priorities on a whiteboard and delegated each task in order of importance. By focusing on these must-have features, we were able to have a simple application that was easy to finish and demonstrate. 
* **Don’t panic** - When we are tired and under stress, we tend to make bad decisions. Our team encountered a serious bug in the final hour and realized we didn’t have time to fix it, so we prepared our presentation accordingly. We spent our last 30 minutes planning a path to walk through the application that we knew we could demonstrate with confidence.
* **Know when to cut a feature** - This goes along with focusing on core functionality. Some features will take up too much of your time and don’t add enough value. If it doesn’t feel right, cut your losses and move on. We had a few awesome features that we decided to axe at the last minute. If you’ve prioritized your todo list, the tasks ranked as “nice to have” will likely get pulled. 
* **Design for presentation** - Chances are, your application is going to look 100x worse on a large, lower contrast projector screen during the presentation. We planned for this by using large, readable fonts and designing simple interface. If the judges can’t make out what it is, it will just be visually distracting.
* **Asynchronous communication** - Even when we were sitting at the same table during the event, we continued using Flowdock to communicate. We found this asynchronous form of communication to be vital to our productivity, allowing people to propose ideas, share links, and notify others about code changes, without breaking from their workflow.
* **Don’t bother with user authentication** - No one cares that you added a 3rd party authentication system to your app. If your application isn’t secure for the presentation, this is actually a good thing because you won’t be wasting valuable time trying to remember your password.

## Presentation

In planning for the event, we anticipated having at least five minutes to present our idea. It turned out we only had three. Three minutes is not a lot of time for anything, let alone present a problem, solution and walk through a software demo. Every second is valuable and should be spent selling your idea to the judges.

![presenting at HackVT](http://petebrown-pics.s3.amazonaws.com/public/presenting-at-hackvt.jpg)

### Here are some tips for nailing the presentation:

* **Get some rest** - With a five person team, we had the luxury of making sure the people representing our team were well rested. Only two of my teammates actually coded through the night. It’s easy to make stupid mistakes when you’re tired, and you don’t want to blow all your hard work in the last 3 minutes.
* **Practice your pitch** - Our presenters spent hours preparing for the demo. They practiced what they were going to say and how everything was going to flow. This extra effort definitely paid off in the end.
* **Don’t talk about the technology** - You’re here to brag about your application, not about the tools you used. If people are interested in what you built it in, they will ask. 
* **No one built a perfect application** - I’m sure everyone’s application had at least one embarrassing bug. Ours had at least three that we were aware of. Don’t apologize and bring attention to these faults, the judges probably won’t even notice. We planned our demo to avoid these issues, and no one besides us knew they existed.
* **Focus on your product** - Don’t rely on presentation slides to help sell your app. Though most of the judges looked well rested, everyone else is tired and just wants to see your application in action.
* **Just because you built it, doesn’t mean you have to show it** - There were quite a few features we developed that did not even get mentioned during our demo. They would have been great to show off if we had a few more minutes, but they were not polished and did not really add core value.

## Technologies Used

* [Flowdock](https://flowdock.com/)
* [Github](https://github.com/)
* [Ruby on Rails](http://rubyonrails.org/)
* [OpenRefine](https://github.com/OpenRefine/OpenRefine) (formerly Google Refine)
* [Backbone.js](http://backbonejs.org/)
* [jQuery](http://jquery.com/)
* [LeafLet](http://leaflet.cloudmade.com/)
* [Select2](http://ivaynberg.github.com/select2/)
* [ActiveAdmin](http://activeadmin.info/)
* [PostgreSQL](http://www.postgresql.org/) and [Postgres.app](http://postgresapp.com/)
* [Geocoder](http://www.rubygeocoder.com/) (Ruby gem)

## Final Thoughts

Winning the hackathon means so much more to our team than the cash or prizes. HackVT was by far the most well organized Vermont event I have ever attended. Every last detail, from the quality of food to the mounted, decorative data sets, made the developers feel welcome and comfortable during our 24 hour stay. This event demonstrated the level of commitment that companies like MyWebGrocer and Dealer.com have made to Vermont, ensuring our state becomes a sustainable technology hub and attract talented individuals for years to come. Thank you to all the volunteers, sponsors, and participants who made it happen. See you again in 2013!
