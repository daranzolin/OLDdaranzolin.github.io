---
layout: post
title:  "My Packages"
date:   2016-10-06 21:00:02 -0700
categories: jekyll update
---
While neglecting my Python coursework this fall, I've written two #rstats packages: [rcicero](https://github.com/daranzolin/rcicero) and [rcanvas.](https://github.com/daranzolin/rcanvas) 
Both are R clients for web APIs. Neither are super impressive. If you're an activist of sorts, `rcicero` may interest you, 
and if you're an educator on the Canvas LMS, `rcanvas` could help. 

Here's a code snippet from `rcicero`:

```
library(rcicero)
set_token_and_user("your_account_email_address", "your_password")

jl <- get_official(first_name = "John", last_name = "Lewis")
```

And here's a code snippet from `rcanvas`:

```
library(rcanvas)
items <- get_course_items(course_id = 20, item = "users", include = "email")
```

More `rcicero` and `rcanvas` goodness is forthcoming.