---
layout: post
title:  "How well do you know Calvin and Hobbes? An R Game"
date:   2017-03-21 21:27:02 -0700
categories: Post
tags: R
---

One of my most prized possessions is *The Complete Calvin and Hobbes,* the undisputed greatest comic of all time. I defy anyone to name a strip more hilarious, more 
cohesive, more emblematic of the vicissitudes of life. I've read and reread those volumes countless times, and
I rank both Calvin and Hobbes as two of the most influential fictional characters in my life. In fact, I once boasted that I knew
the strip so well, I could predict the dialogue of each strip's final panel while only being shown the first two. Incredulous friends would
test my ability, and it became a fun game.

However, what if I want to play and no one is around? Thankfully, my friend R is always there for me.

<!--more-->

Thus, I set out to create a "game" whereby I would be shown a portion of a Calvin and Hobbes comic, and I would have
to guess the ending before time expired. In the end, it was a relatively simple thing to create, thanks to the `rvest` and
`magick` packages. First, I needed to get the comics from somwhere, and I arrived at a [Calvin and Hobbes Daily tumblr](http://calvinhobbesdaily.tumblr.com/) after
some googling. All that remained was scraping and reading the right images:

{% highlight r %}

library(tidyverse)
library(rvest)
library(magick)

random_url <- sprintf("http://calvinhobbesdaily.tumblr.com/page/%d", sample(1:232, 1))
comics <- random_url %>% 
  read_html() %>% 
  html_nodes("#stat-articles img") %>% 
  html_attr("src")

comics <- map(comics, image_read)

{% endhighlight %}

There are 232 pages on the tumblr account, and I want a random one each time I "play". `comics` is now an object with each image strip.

I then needed to: (1) initiate a score; (2) display the cropped strips; (3) wait for me to guess the final panels; (4) display the full strip to either
agony or triumph; (5) increment my score; and (6) repeat. 

{% highlight r %}

correct <- 0
for (comic in comics) {
  print(image_crop(comic, geometry = "255 x 200"))
  Sys.sleep(5)
  print(comic)
  answ <- readline(prompt="Did you know the end? ('Y' for Yes, 'N' for No) ")
  if (answ == "Y") {
    correct <- correct + 1
  }
}
score <- correct/length(comics)
message("Your score was ", score)

{% endhighlight %}

There you have it. A *bona fide* Guess-The-Final-Panel-in-a-Calvin-and-Hobbes-Strip game. If you're curious, I just went 4/10. Time
for another rereading.


