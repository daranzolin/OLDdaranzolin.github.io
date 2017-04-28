---
layout: post
title:  "Picking a Winner: Textmining Award Nominations"
date:   2017-04-27 21:27:02 -0700
categories: Post
tags: R tidytext
---

I recently had the privilege of reviewing internal award nominations. In a secret duel of persuasion, hundreds of employees seized 
the opportunity to outboast their colleagues in boasting about their colleagues. Letterhead was prepared, anecdotes collected,
and thesaureses consulted. 

While the review process on a whole was a charming experience, the sheer volume of submissions made the work somewhat tedious. Yes,
each nomination required a human's careful persusal, but it wasn't long before I was imagining a more...programmatic approach. That
meant R, and that meant `tidytext`.

<!--more-->

For reference, each submission looked like this:

***

[metadata]

**Your First Name:** Lorum ipsum

**Your Last Name:** Lorum ipsum

**Your Email:** Lorum ipsum

**Nominee's First Name:** Lorum ipsum

**Nominee's Last Name:** Lorum ipsum

**Nomination Statement:** Lorum ipsum blah blah

***


My plan was this: read the folders of PDFs into R, scrape and tidy the text, and calculate a winner based on a formula
that weighs total nominations, total words, and total positive words. The nominations came in a zip file, with sub-directories of
each nominee, and then PDFs of the nomination submissions within the sub-directories. I began by unzipping the PDFs and ingesting
the text:

{% highlight r %}

library(pdftools)
library(tidytext)
library(tidyverse)

unzip("Award Nominations.zip")
unlink("Award Nominations.zip")

nom_files <- dir(full.names = TRUE, recursive = TRUE) 

nom_subs <- nom_files %>% 
  map(pdf_text) %>% 
  map(function(x) {
    stringr::str_replace_all(paste(x, collapse = ""), "[\r\n]", "")
  }) %>% 
  map_chr(function(x) {
    stringr::str_extract(x, "(?<=Statement).*$")
  })

nom_names <- str_extract(nom_files, "\\/(.*)\\/")
nom_names <- str_replace_all(nom_names, "\\/", "")

{% endhighlight %}

The desired text occurs after "Statement:", so I was only interested in the text after that point. `nom_names` is derived from
the file paths. 

Now the show begins. Below I: (1) build the `nominations_df` data frame; (2) calculate the number of nominations per employee,
(3) unnest the words; (4) total the words, positive words, and calculate the positive word ratio and nomination score; and
(5) order the data frame by nomination score. When this runs, we'll see our winner!



{% highlight r %}

nominations_df <- tibble(
  nomination_name = nom_names,
  nomination_text = nom_subs
) %>% 
  group_by(nomination_name) %>% 
  mutate(nominations = n()) %>% 
  ungroup() %>%
  unnest_tokens(word, nomination_text) %>% 
  filter(!grepl("[0-9]", word)) %>% 
  mutate(positive_word = if_else(word %in% (get_sentiments("bing") %>% 
                                   filter(sentiment == "positive") %>% 
                                   .$word), 
                                 TRUE, FALSE)) %>% 
  group_by(nomination_name) %>% 
  mutate(total_nomination_words = n(),
         total_positive_words = sum(positive_word),
         positive_word_ratio = round(total_positive_words/total_nomination_words, 2),
         nomination_score = round(nominations^1.5 + (total_nomination_words * positive_word_ratio), 2)) %>% 
  ungroup()
  
nominations_df %>% 
  distinct(nominee_name, nominee_score) %>% 
  arrange(desc(nominee_score))

{% endhighlight %}

There are other, more interesting analyses to run: which positive words occur most? Is there an observable difference in 
the positive words used in the nominations of male vs. female employees? The data is rich, but very much private. The above is
just a playful, albeit lazy idea. 

But far be it from me to deny the nominators and selection committee my full subjectivity and whatever measure of arbitrariness included. 
Much better to read the submissions myself--sorry R.  






