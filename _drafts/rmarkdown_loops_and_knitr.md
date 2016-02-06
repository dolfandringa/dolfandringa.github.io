---
layout: post
title: Outputting test results and graphs with R, RMarkdown and Knitr
category: blog
tags: 
    - R
    - research
    - statistics
    - reporting
    - RMarkdown
    - Knitr
---

Recently I have been analyzing a large amount of data with [R](https://www.r-project.org/). A great tool to do this is [Rstudio](https://www.rstudio.com). It is an IDE for R that makes it easy to write your R code, explore the data and show the graphs. But if you want to communicate your results with others, sitting behind an IDE isn't the best way. Fortunately Rstudio integrates with Knitr, RMarkdown and Pandoc. With those tools you can create PDF, HTML or word files from your R code.

I was playing with this and ran into some problems. I eventually solved all of them and wanted to show my problems and solutions here, both for myself and other coming across the same problems. I am not going to give a complete how-to though. I suggest looking at the [RMarkdown documentation of Rstudio](http://rmarkdown.rstudio.com/) and the [Knitr RMarkdown documentation](http://kbroman.org/knitr_knutshell/pages/Rmarkdown.html) to get you started. This post will go over the following topics:
- How can I combine R loops with Markdown syntax to repeatedly show the test/plot results formatted with RMarkdown?
- How can I display a table nicely in the resulting HTML/PDF?
- How do I show captions with figures?
- How do I get pretty colors automatically for graphs

{% highlight md linenos %}
---
title: "Species Analysis per site of Rapid Visual Census of Fish Species"
author: "Dolf Andringa, Annelies Andringa, Marine Conservation Philippines"
date: "Jan 28th, 2016"
output:
  html_document:
    toc: yes
    fig_caption: yes
---
```{r setup, eval=TRUE, echo=FALSE, message=FALSE, warning=FALSE}
source("getdata.R")
library(knitr)
library("MASS", lib.loc="/usr/lib/R/library")
```
# Methods

Some introduction text....

# Results
```{r loop, echo=FALSE, eval=TRUE, include=FALSE}
for(sg in species_groups[1:3]){
  data <-subset(samples,num_participants>1 & species_group==sg)
  out = NULL
  for(sp in sort(species[species$species_group==sg,]$species_colname)){
    out = c(out, knit_child('individual_species_display.Rmd', envir=globalenv()))
  }
}
```
`r paste(out, collapse='\n')`
{% endhighlight %}

This is my child template.
{% highlight md %}
## `r sp`
```{r eval=TRUE, echo=FALSE, message=FALSE, warning=FALSE}
col = paste0(sp,"_corr_rand")
m<-mean(data$y)
model.norm<-glm(data$y~data$site_name)
```

ANOVA using a GLM assuming a Gaussian distribution.
```{r eval=TRUE, echo=FALSE, message=FALSE, warning=FALSE}
x<-aov(model.norm)
sum<-summary(x)
print(sum)
pvalue <-sum[[1]]$`Pr(>F)`[[1]]

```

```{r eval=TRUE, echo=FALSE, message=FALSE, warning=FALSE, fig.cap="QQ-plot of the residuals of the ANOVA above."}
if(pvalue<0.05){
  qqnorm(x$residuals,main="Normal Q-Q Plot")
  qqline(x$residuals) 
}
```
{% endhighlight %}
