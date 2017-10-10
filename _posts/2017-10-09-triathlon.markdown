---
layout: post
title:  "R at the Golden State Sprint Triathlon"
date:   2017-10-09 18:15:02 -0700
categories: Post
tags: R
---

Earlier today I completed my first (sprint) triathlon. For me, it was 2 hours and 12 minutes of barbarism--a 1/2 mile swim, 
a 15 mile bike ride, and a three mile run to boot. I knew my time was poor; I struggled to wiggle out of my wet suit, was passed 
by over 100 people while cycling,[^1] heard the winner announced before I even started running, and completed the race amongst
septuagenarians. But *how* poor? I needed data.

[^1]: But not by the guy on a BMX.

<!--more-->

[TBF Racing](http://totalbodyfitness.com/site/) did not disappoint--within hours the results were online. A few lines of web scraping later, 
and the data was mine:


{% highlight r %}

library(rvest)
library(magrittr)
library(tidyverse)
library(chron)
url <- "http://totalbodyfitness.com/site/results/2017-golden-state-triathlon-sprint/"
tri <- url %>% 
  read_html() %>% 
  html_nodes(".raceSchedule") %>% 
  html_table() %>%
  .[[1]]

convert_mins <- function(x) {
  ch <- times(x)
  60 * hours(ch) + minutes(ch)
}

tri %<>%
  mutate(minutes = convert_mins(`Finish Time`))


{% endhighlight %}

Now, some questions:

**What percentile was my time? (132 minutes)**

```
> round(pnorm(132, mean = mean(tri$minutes), sd = sd(tri$minutes), lower.tail = FALSE) * 100)
[1] 13

```

Yeesh. It doesn't look any better on a histogram:

{% highlight r %}

ggplot(tri, aes(minutes)) +
  geom_histogram(fill = "lightgreen", color = "black") +
  geom_vline(xintercept = 132, color = "red", linetype = "dashed") +
  labs(x = "",
       y = "",
       title = "The 2017 TBF Racing Sprint Triathlon",
       subtitle = "Race Minutes, Reversed") +
  scale_x_reverse() +
  theme_minimal()
  
{% endhighlight %}

![useful image]({{ site.url }}/assets/Trihist.png)

Bonus: Age distribution

![useful image]({{ site.url }}/assets/Triage.png)


**Is there a correlation between age and finish time?**

The resounding 'No!' was cool to see.


![useful image]({{ site.url }}/assets/Triscatplot.png)

**What was the distribution by gender?**


![useful image]({{ site.url }}/assets/Tribox.png)

Some summary statistics:


{% highlight r %}

tri %>% 
  group_by(Gender) %>% 
  summarize(mean_minutes_to_finish = mean(minutes),
            sd_minutes = sd(minutes),
            median_minutes = median(minutes),
            best = min(minutes))

{% endhighlight %}

```
# A tibble: 2 x 5
  Gender mean_minutes_to_finish sd_minutes median_minutes  best
   <chr>                  <dbl>      <dbl>          <dbl> <dbl>
1      F               118.7636   17.35892            117    84
2      M               106.7521   18.76530            104    74

```

Would I do it again? Maybe.







