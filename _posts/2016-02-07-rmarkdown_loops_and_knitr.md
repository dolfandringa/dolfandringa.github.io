---
layout: post
title: Outputting HTML or PDF results in a loop with R, RMarkdown and Knitr
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


## Table output in RMarkdown

To start with the quickest question first, you have to look at the second code block at the bottom. At the end it says `knitr::kable(coefs)`. This is a function from Knitr that outputs a nice table in markdown syntax. If you pass it a table or data frame in R, the result is a nice looking table. Quite a nice function to have.

## Loops with RMarkdown to repeat results with different data

With RMarkdown you can write Markdown syntax in an (Rmd) file, interspersed with code blocks with R code. Knitr reads the R-code, executes it in R and pastes the results back into the markdown output. That is then converted into HTML or PDF. 
The trick to looping over a set of data and running the same tests/plots for all parts of the data is splitting your file up into multiple files.
You have one "parent" file which loops over your data, and for each loop iteration, it calls a child file to do the actual analysis.

Let's use an example. I have data for about 300 species. For each one I would like to run the same analysis. Furthermore my species are grouped into species_groups. I would like to group the analyses in these species_groups.
I have a data frame called `samples` with the actual data, and a list called `species_groups` and a list called `species`. All are saved in samples.RDa. By loading that the objects become available. 

In my parent file (see below) I loop over all species groups in `for(sg in species_groups)` and with `for(sp in sort(species[species$species_group==sg,]$species_colname)){` I loop over a sorted list of all species in the species group. Inside the inner loop the Knitr magic happens. The call to knit_child causes Knitr at that point to load the file 'individual_species_blog.Rmd' specified as an argument. This file is another Rmd file (see second code block) that contains Markdown with R code blocks interspersed.

The child file does the actual analysis per species and outputs the results. The parent file only calls it.
What about the `out<-NULL` `out<-c(out,knit_child(...))` and `paste(out...)` lines in the parent file, I hear you ask? Well, the result of knit_child is already Markdown syntax. But anything inside the triple-back-quoted blocks in RMarkdown is printed in a code block, which would make a mess of the Markdown syntax that is the result of knit_child. So we instead make a vector called 'out' and append all the results of knit_child onto it. So in the end, after the last loop, out is a vector of a whole lot of strings. The inline code `paste(out, collapse='\n')` prints the actual results and puts newline characters between the different strings.

Talking about output, there are different ways to modify how the R code is being output into Markdown syntax. These are all specified in the `{r ...}` blocks for each block of R code. One that is important in this respect is the include=FALSE parameter that is in the R-block where knit_child is called. If you don't include it, Knitr will print all kinds of other output in the Markdown file that has to do with the knit_child call. As it is stuff we don't want in our final output, I say eval=TRUE so the code does get evaluated normally, but include=FALSE makes sure no output is actually included in the resulting Markdown file. As you can see, the subsequent paste(output...) call is outside this block, so the output of that statement ís included in the markdown file.

The last thing tying it all together is making variables from the parent file available in the child file. This is where the environment comes in. Inside our species loop, we make a new environment for each species with `env.new()`. This environment inherits from the parent environment. So anything that was available in our parent environment (like the variables `sg` `sp`  and `specnum`) is also available in the new environment. But variables inside the child environment don't accidentally carry over to other child environments. That way we can safely refer to `sp` `specnum` and `sg` in our child Rmd file. The only problem is that the child Rmd file can't be run individually anymore because these variables aren't available anymore if the child Rmd file isn't called from the parent Rmd file. The solution to this is to put something like `if(!exists('sp'))` in your child Rmd file to specify a species if it isn't specified already.


The parent Rmd file:

<pre><code>
---
title: "test"
author: 'Dolf Andringa'
date: "February 7, 2016"
output:
  html_document:
    fig_caption: yes
    keep_md: yes
    number_sections: no
    theme: journal
    toc: yes
    toc_depth: 2
    toc_float:
      collapsed: true
---
```{r getdata, results=FALSE, eval=TRUE, echo=FALSE, message=FALSE, warning=FALSE}
load("samples.RDa")
library(knitr)
```
# Methods

.... some explanation

# Results
```{r echo=FALSE, eval=TRUE, include=FALSE}
out <- NULL
for(sg in species_groups){
  print(sg)
  specnum<-0
  for(sp in sort(species[species$species_group==sg,]$species_colname)){
    sp<-sp #update the sp variable in the environment
    sp.common<-species[species$species_colname==sp,c("common_name")] #update the sp.common variable in the environment
    env=new.env() #create a new empty environment, it inherits objects from the current environment.
    out <- c(out, knit_child('individual_species_blog.Rmd', envir=env))
    specnum<-specnum+1
  } 
}
```
`r paste(out, collapse='\n')`
</pre></code>

This is the child Rmd file (called individual_species_blog.Rmd):

<pre><code>
---
title: "Individual Species"
author: 'Dolf Andringa'
date: "February 7, 2016"
output: html_document
---
```{r echo=FALSE, eval=TRUE, include=FALSE}
library(knitr)
library(lmtest)
load("samples.RDa")
data <-subset(samples, species_group==sg)
data$y <- data[,c(sp)]
```

```{r eval='TRUE', echo=FALSE, results='asis'}
if(specnum==0) cat('\n##',sg,'\n');
```

### `r sp` {.tabset}
`r sp.common`

```{r eval=TRUE, echo=FALSE}

m<-glm(data$y~data$site_name, family=poisson)

coefs<-NULL
coefs$coefficient<-NULL
coefs$value<-NULL
coefs$pvalue<-NULL
coeft<-coeftest(m)
for(name in names(m$coefficients)){
    p<-round(coeft[name,c("Pr(>|z|)")],3)
    coefs$coefficient<-c(coefs$coefficient,name)
    coefs$value<-c(coefs$value,m$coefficients[[name]])
    coefs$pvalue<-c(coefs$pvalue,p)
}
coefs<-as.data.frame(coefs)
coefs<-coefs[with(coefs,order(coefs$pvalue)),]
knitr::kable(coefs)
```
</pre></code>
