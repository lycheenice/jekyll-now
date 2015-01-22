---
layout: post
title:  "Blogging Like a [Data] Hacker"
date:   2014-01-15 17:29:00
tags:   
 - blogging
 - data hacker
 - infographics

---

I use [Wunderlist](http://www.wunderlist.com) to organize my
todo-lists (work, personal, movies to watch, etc.) 
Around a year ago, I thought it would be fun to keep a list of ideas
that could make a nice blog post, so I started a **future blog posts**
list. 

Over the last year I've added 113 items to all my lists. However, in the
summer the activity on all other lists tapered, whereas ideas for
potential blog posts kept accumulating steadily. (Toggle the series
separately in the chart below to highlight the behavior) 

{% include bladh/wunderlist_infographic.html %}

As it almost always happens, [quantifying your own
behavior](http://quantifiedself.com) leads to interesting insights. 

The analysis made me realize that the **future blog posts** list
is the only reason why I still use *Wunderlist*. It also served the
purpose to remind me that it's about time to **start marking off
items** from that list. Hence blogging.

## Blogging Like a Hacker

When I start something, I usually spend an insane amount of time
researching on all the possible alternatives to accomplish my task. 
Then I move on to something else. 

> Getting out of the door is the hardest part.

A good way to counter this is to embrace the *make first, think
later* motto. Or, [keep your dopamine up](http://blog.idonethis.com/post/70179626669/the-science-of-motivation-your-brain-on-dopamine).
For me, this means using the tools that I already master to quickly
make progress. That is, I should be blogging like I code, leveraging
my favorite revision control system and editor.

Five years ago, [Github](http://github.com) CEO **Tom
Preston-Werner** thoroughly elaborates on that in his post [Blogging Like a
Hacker](http://tom.preston-werner.com/2008/11/17/blogging-like-a-hacker.html),
a recommended read.

Today, Tom's idea has evolved to become [Github
Pages](http://pages.github.com) is a fully-fledged, **free hosting
service for static webpages**, on *Github*. You write your pages in a
simple markdown language, organize them in a straightforward
directory structure, and *Github* does the rest.

Unsurprisingly, *Github Pages* is quickly **gaining popularity**.

The chart below shows the **activity** on *Github Pages* (measured as
number of pushes per month to \*.github.io repositories, usually
used to host personal pages) normalized on the activity on all
*Github* repositories. 

{% include bladh/github_io_infographic.html %}

All in all, *Github Pages* is both convenient and trendy.
However, the reason why I want my blog and personal pages on *Github*
is neither a matter of convenience nor a matter of trendiness. It's a matter of principle. 

> If you make something good, people will use it.

No, they won't. They most likely won't even know you did it. 

If you really want others to build on your work, you have to **lower the
entrance barrier** for them in a similar fashion as you had to do it for
yourself to be able to get started. 

*Github* is a **brilliant solution** to the entrance barrier problem. It
 makes it dead-simple to get started building on the work of others, and
 safeguards the concept of **intellectual property**, which is
 automatically built in the forking process. 
In addition, *Github* provides social filters (rankings, voting) to
 allow the good content to get recognition. 

I expect to see the *Github* mantra [applied to all sort of things](http://www.wired.com/wiredenterprise/2013/09/github-for-anything/), other than
code. Like this **recipe**? Fork it, replace garlic with shallots, re-share. Love
this **song** but want to change the lyrics? Love the **movie** but not the
ending? Maybe some days we'll fork artworks, companies, and even
**physical objects**.

Like this? [Fork it, make it better](https://github.com/LucaFoschini/lucafoschini.github.io)!

## Beauty Matters

Ease of access is however not enough to make your content appealing to
others. It will get a chance to be adopted only if it **looks
beautiful**.

> Responsive, typography-oriented, mobile first. Beautiful. 

I spent some time daydreaming about my blog looking like the visual
wonderland of [Bret Victor](http://worrydream.com/)'s page. Then I
woke up. 

I'm not a designer. I have an eye for design but don't know how to
make it, much like one can love food without being an iron chef. 
To make good food I **must follow recipes**, perhaps slightly
twisting them to meet my needs. Cut down on salt, add some vinegar,
but never wing quantities or cooking times, that's a recipe for
a recipe disaster. So, templates.

Since I was set on *Github pages*, I spent a few hours looking around
for *Jekyll* (the engine powering *Github Pages*) themes with
non-boring yet minimal typography that would look good on mobile.  See
[this series of
articles](http://paulstamatiou.com/responsive-retina-blog-development-part-1)
by [Paul Stamatiou](http://paulstamatiou.com/about), and [this](http://nicolashery.com/fast-mobile-friendly-website-with-jekyll/)
concise-yet-precise by [Nicolas Hery](http://nicolashery.com/) for
some background on the topic.

I then came across *Ghost* and *Vapor*. [Ghost](https://ghost.org/) is the
new kid on the block when it comes to blogging, the media [waxed
poetic](http://techcrunch.com/2013/05/07/ghost-will-take-your-boring-blog-to-the-next-astral-plane/)
about it. *Vapor* is a *Ghost* theme by [Seth
Lilly](http://sethlilly.com/), designed for responsiveness and with an
eye on typography. 

*Ghost* and *Vapor* look great. Except, they're not based on *Jekyll*,
 nor they can be hosted on *Github pages*. Make first, think later: I
 ended up porting *Vapor* to *Jekyll* so I could use it on *Github pages*.
[Go grab it](https://github.com/LucaFoschini/jekyll-vapor), or see it live [here](http://lucafoschini.github.io/jekyll-vapor/).

## It's the Data, Stupid.

Now that we've gone to great lengths elaborating on the *how* of
blogging, we should devote some attention to the *what*. What is that
I want so eagerly to blog about? To a data scientist, the *what* is
necessarily the data. Or better, the *what* is *in* the data.

> The data tells the story.

Many stories, actually. And the decision on which one becomes more
prominent is up to whoever provides the interpretation of the
data. Therefore, such **interpretation** should be **reproducible** by others.

Practically, that translates into making available the plethora of
tools and **intermediate results** that were involved in processing the
data, starting from the sources to the final output.

*Github* is again a winner on this front, as it makes it easy to
 organize all the heterogenous components that play a role in writing a
 post like this one, **enabling anyone to reproduce it on their system**.

This post is fully reproducible, as the source
code can be found [here](https://github.com/LucaFoschini/lucafoschini.github.io/tree/master/_posts), the source for the charts is
[here](https://github.com/LucaFoschini/lucafoschini.github.io/tree/master/_includes/bladh),
the data used is [here](https://github.com/LucaFoschini/lucafoschini.github.io/tree/master/data), and the scripts used to
process that data (*IPython Notebooks*) are [here](https://github.com/LucaFoschini/lucafoschini.github.io/tree/master/notebooks). 

Even more importantly, *Github pages* (rather, *Jekyll*) allows to
seamlessly [interlink all those heterogenous
components](http://jekyllrb.com/docs/templates/). That means that if
you clone this post, play with the notebooks to perform a different
analysis, you'll see the your local copy of the chart **updating instantly**.
   
## Conclusion

As data-driven blogging is becoming a thing
(and for some even a [profitable business](http://techcrunch.com/2013/11/26/priceonomics-data-services/)) and infographics are
starting to replace traditional journalism (see the tongue-in-cheek
chart below) I felt the urge to jump on the bandwagon. That meant
doing some research on the available tools that would serve the
purpose of making blogging about data easy to (re-)produce. 

{% include bladh/data_driven_journalism_infographic.html %}

*Github pages* seems to be the right tool to harbor the **diversified
 ecosystem** that a data hacker needs to produce and share data
 insights. That comes with the benefit of **fostering reproducibility**, which is a key tenet in data
 analysis.  
 Finally, *Github pages* is flexible enough to allow **customizing the
 details of the presentation** with the goal of making it visually appealing, also a
 requirement for data-driven journalism.

I hope my findings can help other people with the same need in their
decision process, and I'd love to hear what their conclusion will
end up being, so please **share your thoughts**.

In the meanwhile, let the blogging begin!
