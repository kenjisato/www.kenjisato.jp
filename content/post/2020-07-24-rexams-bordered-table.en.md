---
title: Generate bordered tables with exams::exams2mooodle
author: kenjisato
date: '2020-07-24'
slug: moodle-bordered-table
categories: []
tags: ["R","exams"]
description: 'R/exams'
---

## Motivation

By default, tables in moodle problems created with `exams::exams2moodle` doesn't have table or cell borders. I want tables with borders! 

If you does too, you might find this post useful. 

## Stylesheet

Create a style for a class you will use for tables. Note that this is not a CSS file but a HTML code snippet that you want to include in markdown files. 

<script src="https://gist.github.com/kenjisato/b8c3d14d20eaf11d5bc8238a97a61139.js"></script>

Instead of styling a class, it's possible to style `table` tag itself. But I strongly discourage you from doing this. That will affect the appearance of other tables used in Moodle. 

## Make a problem with tables

 Two important points:

- Embed the style under the Question section. I would use something like  `writeLines(readLines("https://git.io/JJ8DD")`. 
- When you make a table, use HTML format and add the class attribute to the table tag (`bordered-table`). `knitr::kable()` has `table.attr` argument that does this for you. 


### Example

Here's a sample problem. 


````
```{r}
N <- 3

mountains <- 
  data.frame(
    Mountain = c("Everest", "K2", "Kangchenjunga", "Lhotse", "Makalu", 
                 "Cho Oyu", "Dhaulagiri", "Manaslu"),
    `Height (m)` = c(8848, 8611, 8586, 8516, 8485, 8188, 8167, 8163), 
    check.names = FALSE
  )

choice <- sample(nrow(mountains), N)

table <- mountains[choice, ]
table$`Height (m)` <- paste0("\\##ANSWER", seq_len(N), "##")

```

Question
========

```{r CSS, echo = FALSE, results = "asis"}
writeLines(readLines("https://git.io/JJ8DD"))
```

How high are the following mountains? Answer the height above sea level in meters. 

```{r question, echo = FALSE, results = "asis"}
knitr::kable(table, format = "html", 
             table.attr='class="bordered-table"',
             align = "lr",
             row.names = FALSE)
```

Solution
========

```{r solution, echo = FALSE, results = "asis"}
knitr::kable(mountains[choice, ], format = "html", 
             table.attr = 'class="bordered-table"',
             row.names = FALSE)
```

Meta-information
================

extype: cloze
exsolution: `r paste(mountains[choice, "Height (m)"], collapse = "|")`
exclozetype: `r paste(rep("num", 3), collapse = "|")`
exname: bordered-table
extol: 0
exextra[numwidth,logical]: TRUE


````

## Result

You will get a quiz and solution that look like the following screenshots.

#### Quiz

![quiz](/images/postimage/rexams-bordered-table1.png)

#### Solution

![solution](/images/postimage/rexams-bordered-table2.png)

