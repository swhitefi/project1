Project 1
========

The project for the first half of the semester is due on October 24th as a pull
request to this repository. The emphasis for this projet is on data analysis
using tools of reproducible research including version tracking, knitr, and R
markdown. You are free to answer any question you want, but you must provide a
quantitative report that utilizes the material we have covered durign the first
half of the semester.

***Ground rules...***
* Your document should layout your question in the introduction, provide a brief
synopsis for why it is important, describe how you obtained the data, your general
approach to analyzing the data, a summary of your results, a discussion, and
mention of what your future directions would be.
* You should post your project as a pull request to this repository as `*.Rmd`,
`*.md`, and `.html` files. Prior to submitting the files, you should set
`echo=FALSE` for all of the code chunks. This will hide the R code in the final
document.
* The final document should be approximately 5 pages when the code chunks are
hidden and  you use a normal sized font.
* I don't care what your question is or what subject domain it covers. The key is
that it needs to be data driven. Microbiology, video game statisitics, immunology,
whatever. Do something where it will be easy for you to come up with an interesting
question and fun analysis.

#Replicating Jhansi Leslie's Analysis for her Anaerobe 2014 Poster Entitled *"Pre-Colonization By a Less Virulent Strain of *Clostridium difficile* Protects From Infection by a Lethal Strain,"* using R
###**By Shawn Whitefield**

##Introduction

*Clostridium difficile* is an anaerobic spore forming bacterium that is respobsible for 14,000 deaths and 1 billion dollars in excess healthcare costs annually in the US.(1)  *C. diff* infection (CDI) is primarily healthcare associated, and infected individuals can experience symptoms ranging from asymptomatic colonization, to toxic megacolon requiring colectomy. Protection from CDI is thought to occur through interactions between bacteria in the gut as well as via the adaptive immune response.
The Young lab has a mouse model of CDI which Jhansi used to study the imact of adaptive immunity and pre-colonization by a less virulent strain on infection outcomes (Figure 1 poster below). Briefly, in this model, mice are given cefoperazone for ten days to perturb their indigenous microbiota. They are then treated with either water or a non-lethal *C. diff* strain. Both groups of mice are then given clindamycin. After six weeks mice in each treatment group are challanged with a lethal *C diff* strain and outcomes are measured.  

Jhansi Leslie, a graduate student in the Young lab, created this poster for a conference. There are some changes that could strengthen the presentation of her data. In this project, I am running statistical tests and creating new (and hopefully improved) data visualizations for parts of her mouse colonization data.

###Jhansi's Anaerobe 2014 Poster: ![](/Users/shawnwhitefield/Desktop/r_class/project_1/Anaerobe2014_posterLESLIE_copy.jpg)

##Methods & Results 
Jhansi looked at the effect of the adaptive immune system on *C. diff* infection. She infected mice with *C. diff* strain 630, and compared the level of colonization of wildtype vs. RAG knockout mice (which have no mature B and T lymphocytes) over time.  

The data was stored in a large multi-sheet Excel document, so the first step was to generate usable dataframes with the pertinant information.

###Example 1: Initial Excel Dataset 
![](/Users/shawnwhitefield/Desktop/Rag_longterm_datasheets.jpg)

```
##        ID             cage          sex     treatment genotype
##  Min.   : 1.00   Min.   :1.00   female:11   630:18    RAG:15  
##  1st Qu.: 8.75   1st Qu.:1.75   male  :21   c  :14    WT :17  
##  Median :16.50   Median :3.00                                 
##  Mean   :16.50   Mean   :3.12                                 
##  3rd Qu.:24.25   3rd Qu.:5.00                                 
##  Max.   :32.00   Max.   :6.00                                 
##                                                               
##  X630_infection_day      Day1           Day2           Day4     
##  Min.   :16.2       Min.   :14.8   Min.   :13.6   Min.   :13.5  
##  1st Qu.:18.4       1st Qu.:18.4   1st Qu.:18.2   1st Qu.:17.9  
##  Median :19.3       Median :19.3   Median :19.6   Median :19.2  
##  Mean   :19.8       Mean   :19.8   Mean   :19.8   Mean   :19.6  
##  3rd Qu.:21.4       3rd Qu.:21.4   3rd Qu.:21.2   3rd Qu.:21.8  
##  Max.   :24.0       Max.   :24.2   Max.   :24.3   Max.   :23.7  
##                                                                 
##       Day5           Day7           Day9          Day10     
##  Min.   :13.1   Min.   :12.0   Min.   :16.1   Min.   :16.1  
##  1st Qu.:17.8   1st Qu.:18.4   1st Qu.:18.7   1st Qu.:18.6  
##  Median :19.5   Median :19.8   Median :19.9   Median :20.1  
##  Mean   :19.5   Mean   :20.0   Mean   :20.2   Mean   :20.2  
##  3rd Qu.:21.5   3rd Qu.:22.2   3rd Qu.:21.2   3rd Qu.:21.6  
##  Max.   :23.9   Max.   :24.2   Max.   :27.5   Max.   :24.3  
##                                NA's   :1      NA's   :1     
##      Day14          Day16          Day22          Day31     
##  Min.   :16.4   Min.   :16.3   Min.   :17.3   Min.   :18.4  
##  1st Qu.:19.4   1st Qu.:19.3   1st Qu.:20.4   1st Qu.:20.3  
##  Median :20.9   Median :21.0   Median :21.9   Median :22.4  
##  Mean   :21.0   Mean   :20.9   Mean   :21.9   Mean   :22.1  
##  3rd Qu.:22.9   3rd Qu.:22.5   3rd Qu.:23.6   3rd Qu.:23.7  
##  Max.   :26.6   Max.   :25.0   Max.   :26.2   Max.   :26.5  
##  NA's   :1      NA's   :1      NA's   :1      NA's   :1     
##      Day32      IP_clinda_day_41 VPI_infection_day_42     Day43     
##  Min.   :18.5   Min.   :18.7     Min.   :18.3         Min.   :16.0  
##  1st Qu.:20.2   1st Qu.:20.4     1st Qu.:20.2         1st Qu.:19.9  
##  Median :22.5   Median :23.0     Median :22.5         Median :22.6  
##  Mean   :22.3   Mean   :22.4     Mean   :22.3         Mean   :21.9  
##  3rd Qu.:23.9   3rd Qu.:24.5     3rd Qu.:24.0         3rd Qu.:23.8  
##  Max.   :27.0   Max.   :26.8     Max.   :27.0         Max.   :26.7  
##  NA's   :1      NA's   :1        NA's   :1            NA's   :1     
##      Day44        pBLweight        CFUDay1            CFUDay4        
##  Min.   :15.8   Min.   : 81.4   Min.   :       0   Min.   :1.00e+02  
##  1st Qu.:19.6   1st Qu.: 85.4   1st Qu.:     100   1st Qu.:1.00e+02  
##  Median :20.1   Median : 89.6   Median : 2900050   Median :5.00e+05  
##  Mean   :20.4   Mean   : 92.0   Mean   :16418794   Mean   :1.94e+07  
##  3rd Qu.:21.4   3rd Qu.: 98.7   3rd Qu.:23000000   3rd Qu.:3.50e+07  
##  Max.   :24.5   Max.   :105.5   Max.   :85000000   Max.   :1.03e+08  
##  NA's   :1      NA's   :1                                            
##     CFUDay7            CFUDay10           CFUDay16          CFUDay22      
##  Min.   :     100   Min.   :     100   Min.   :    100   Min.   :    100  
##  1st Qu.:     100   1st Qu.:     100   1st Qu.:    100   1st Qu.:    100  
##  Median :     100   Median :     100   Median :    100   Median :    100  
##  Mean   : 6056309   Mean   : 1643169   Mean   : 825558   Mean   : 218784  
##  3rd Qu.: 4425000   3rd Qu.:  220000   3rd Qu.: 295050   3rd Qu.:   5050  
##  Max.   :56000000   Max.   :31000000   Max.   :9000000   Max.   :2800000  
##                     NA's   :3          NA's   :1         NA's   :1        
##     CFUDay31         CFUDay40        infection_status
##  Min.   :   100   Min.   :    100   cleared  : 9     
##  1st Qu.:   100   1st Qu.:    100   colonized: 8     
##  Median :   100   Median :    100   control  :14     
##  Mean   : 17558   Mean   : 165881   NA's     : 1     
##  3rd Qu.:  4050   3rd Qu.: 101550                    
##  Max.   :290000   Max.   :3200000                    
##  NA's   :1        NA's   :1
```
###Figure 8. Discussion and Data Visualization Modifications
The purpose of figure 8 is to demonstrate that adaptive immunity does not play a role in *C. diff* infection clearance. This figure shows the CFU (mean +/-SEM) for each cage of RAG or WT mice over time after infection with *C. diff* strain 630 or mock infection. There are 6 total cages, each cage has between 4 and 8 mice, and cages house either WT or RAG mice. 

These numbers are small, and I think that displaying the mean+/- SEM colonization by cage takes away from the interpretation of this data. Visually it looks as though colonization is dependent on cage effects.  This is likely true, but it is not important for the purpose of this figure. Displaying the data by cage takes the emphasis of the figure away from the main point, which is that there is variation in colonization outcomes that is consistent across RAG vs WT mice.   

I think that the data would be expressed better as individual mice since the
numbers are small.  Additionally separate symbols for RAG vs WT genotype visually demonstrate that these genotypes do not impact colonization. Furthermore, a distinction between mock and actual infection using a line at the limit of detection will allow easier and more clear interpretation of these data.

Also, a chi-square test will provide a stronger argument that clearance of infection does not depend on adaptive immunity.  

![plot of chunk unnamed-chunk-2](./README_files/figure-html/unnamed-chunk-2.png) 

```
## Warning: Chi-squared approximation may be incorrect
```

```
## 
## 	Pearson's Chi-squared test
## 
## data:  mouseinf_geno$infection_status and mouseinf_geno$genotype
## X-squared = 0.1075, df = 2, p-value = 0.9477
```
###Figure 9. Discussion and Data Visualization Modifications
In figure 9, Jhansi grouped WT and RAG mice by their *C. diff* status at the time of VPI infection (naive, previously 630 colonized and cleared infection, 630 colonized), and plotted this against the percentage of baseline weight (a measure of mouse illness) post VPI infection. 

This figure demonstrates that mice that are infected by a less virulent strain at the time of infection are protected from CDI illness once infected by a lethal strain, and that adaptive immunity does not modify this effect.

This figure was created in Prism and there are some issues with how the data is displayed that detract from its impact.  Frist, healthy mice should lose the least amount of weight. In this graph, colonized mice have the highest percent of baseline weight (ie their weight has not changed), but it looks as though there is a larger effect in this group because that bar is the highest. Second, this graph is supposed to be a box and whisker plot, however it looks like a bar chart with error bars and many outliers because the first quartile hits the x axis.  These issues have inspired me to display this figure differently in a way that is easier to understand. 

Also, I am not a fan of asterisk and bars across graphs to display significance because I think that they are distracting and lead the viewer to pay attention to the asterisk rather than the data itself. I have decided to perform pariwise t-tests to compare these groups of mice and then report the statistical test and p-value in a subtitle. 

![plot of chunk unnamed-chunk-3](./README_files/figure-html/unnamed-chunk-3.png) 

```
## $cleared
##          mean   quantile.0%  quantile.25%  quantile.50%  quantile.75% 
##         91.32         84.33         89.63         91.67         92.31 
## quantile.100%        median            sd 
##         97.99         91.67          3.78 
## 
## $colonized
##          mean   quantile.0%  quantile.25%  quantile.50%  quantile.75% 
##        99.311        81.443        98.974       101.269       104.097 
## quantile.100%        median            sd 
##       105.291       101.269         7.673 
## 
## $control
##          mean   quantile.0%  quantile.25%  quantile.50%  quantile.75% 
##        88.172        82.927        84.515        85.848        87.638 
## quantile.100%        median            sd 
##       105.473        85.848         6.922
```

```
## 
## 	Pairwise comparisons using t tests with pooled SD 
## 
## data:  mousecfu$pBLweight and mousecfu$infection_status 
## 
##           cleared colonized
## colonized 0.0318  -        
## control   0.2598  0.0015   
## 
## P value adjustment method: holm
```
##Discussion
In this project, I have replicated analysis of Jhansi's mouse colonization data.  I have performed statistical tests and created new graphics that display the data in a way that I think is easier to understand.  

Jhansi and I discussed this project as well as the new plots that I created. We both thought it was interesting that I, as someone who had never before seen this data, thought different things should be emphasized than she did as the person who actually performed the experiments and collected the data. 

Additionally, this project taught me that it is difficult to analyze someone elses dataset when you have no idea what variables mean and how it is organized.  Working on this has given me ideas on how to better anotate and store (such as keeping variables in long format vs wide format) my own data so that it will be easier to analyze and to share with collaborators.   
 
##Sources
1. [CDC.gov *C. diff*](http://www.cdc.gov/HAI/organisms/cdiff/Cdiff_infect.html)

