---
output: pdf_document
---
output: html_document
---
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
```{r echo=FALSE}
echo=FALSE
#reading in cfu and weights datafile
mousecfu<-read.table(file="cleaned-up_mouse_data.txt", header=T)
summary(mousecfu)
```
###Figure 8. Discussion and Data Visualization Modifications
The purpose of figure 8 is to demonstrate that adaptive immunity does not play a role in *C. diff* infection clearance. This figure shows the CFU (mean +/-SEM) for each cage of RAG or WT mice over time after infection with *C. diff* strain 630 or mock infection. There are 6 total cages, each cage has between 4 and 8 mice, and cages house either WT or RAG mice. 

These numbers are small, and I think that displaying the mean+/- SEM colonization by cage takes away from the interpretation of this data. Visually it looks as though colonization is dependent on cage effects.  This is likely true, but it is not important for the purpose of this figure. Displaying the data by cage takes the emphasis of the figure away from the main point, which is that there is variation in colonization outcomes that is consistent across RAG vs WT mice.   

I think that the data would be expressed better as individual mice since the
numbers are small.  Additionally separate symbols for RAG vs WT genotype visually demonstrate that these genotypes do not impact colonization. Furthermore, a distinction between mock and actual infection using a line at the limit of detection will allow easier and more clear interpretation of these data.

Also, a chi-square test will provide a stronger argument that clearance of infection does not depend on adaptive immunity.  

```{r echo=FALSE}
#reshape data so a new variable called day takes on the value 1,4,7,16,22,31,40 for each mouse
#add each wide variable day to a new variable called cfu corresponding with each day
reshapecfu<-reshape(mousecfu, varying=c("CFUDay1",  "CFUDay4",  "CFUDay7",	"CFUDay10",	"CFUDay16",	"CFUDay22",	"CFUDay31",	"CFUDay40"), v.names="CFU", timevar="Day", times=c(1, 4, 7, 10, 16, 22, 31, 40), new.row.names=1:10000, direction="long")
#sorting by day
attach(reshapecfu)
sortedreshapecfu<-reshapecfu[order(Day),]
detach(reshapecfu)      
#create line graph of all mice by genotype
par(mar=c(6,6,4,4), xpd=T)
plot(sortedreshapecfu[sortedreshapecfu$genotype=="RAG","CFU"]~sortedreshapecfu[sortedreshapecfu$genotype=="RAG","Day"], type="p", col="green", pch=3, log="y", main="630 CFU/g Feces in RAG & WT Mice Over Time", xlab="Day", ylab="", las=1, cex=1, sub="ChiSquare RAG vs. WT Colonization Outcome p=.94")
#adding WT mice
points(sortedreshapecfu[sortedreshapecfu$genotype=="WT","CFU"]~sortedreshapecfu[sortedreshapecfu$genotype=="WT","Day"], type="p", col="blue", pch=1)
#connecting the mice
#1
lines(sortedreshapecfu[sortedreshapecfu$id=="1","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="1", "Day"], type="l", col="grey")
#2
lines(sortedreshapecfu[sortedreshapecfu$id=="2","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="2", "Day"], type="l", col="grey")
#3
lines(sortedreshapecfu[sortedreshapecfu$id=="3","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="3", "Day"], type="l", col="grey")
#4
lines(sortedreshapecfu[sortedreshapecfu$id=="4","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="4", "Day"], type="l", col="grey")
#5
lines(sortedreshapecfu[sortedreshapecfu$id=="5","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="5", "Day"], type="l", col="grey")
#6
lines(sortedreshapecfu[sortedreshapecfu$id=="6","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="6", "Day"], type="l", col="grey")
#7
lines(sortedreshapecfu[sortedreshapecfu$id=="7","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="7", "Day"], type="l", col="grey")
#8
lines(sortedreshapecfu[sortedreshapecfu$id=="8","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="8", "Day"], type="l", col="grey")
#9
lines(sortedreshapecfu[sortedreshapecfu$id=="9","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="9", "Day"], type="l", col="grey")
#10
lines(sortedreshapecfu[sortedreshapecfu$id=="10","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="10", "Day"], type="l", col="grey")
#11
lines(sortedreshapecfu[sortedreshapecfu$id=="11","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="11", "Day"], type="l", col="grey")
#12
lines(sortedreshapecfu[sortedreshapecfu$id=="12","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="12", "Day"], type="l", col="grey")
#13
lines(sortedreshapecfu[sortedreshapecfu$id=="13","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="13", "Day"], type="l", col="grey")
#14
lines(sortedreshapecfu[sortedreshapecfu$id=="14","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="14", "Day"], type="l", col="grey")
#15
lines(sortedreshapecfu[sortedreshapecfu$id=="15","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="15", "Day"], type="l", col="grey")
#16
lines(sortedreshapecfu[sortedreshapecfu$id=="16","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="16", "Day"], type="l", col="grey")
#17
lines(sortedreshapecfu[sortedreshapecfu$id=="17","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="17", "Day"], type="l", col="grey")
#18
lines(sortedreshapecfu[sortedreshapecfu$id=="18","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="18", "Day"], type="l", col="grey")
#19
lines(sortedreshapecfu[sortedreshapecfu$id=="19","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="19", "Day"], type="l", col="grey")
#20
lines(sortedreshapecfu[sortedreshapecfu$id=="20","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="20", "Day"], type="l", col="grey")
#21
lines(sortedreshapecfu[sortedreshapecfu$id=="21","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="21", "Day"], type="l", col="grey")
#22
lines(sortedreshapecfu[sortedreshapecfu$id=="22","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="22", "Day"], type="l", col="grey")
#23
lines(sortedreshapecfu[sortedreshapecfu$id=="23","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="23", "Day"], type="l", col="grey")
#24
lines(sortedreshapecfu[sortedreshapecfu$id=="24","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="24", "Day"], type="l", col="grey")
#25
lines(sortedreshapecfu[sortedreshapecfu$id=="25","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="25", "Day"], type="l", col="grey")
#26
lines(sortedreshapecfu[sortedreshapecfu$id=="26","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="26", "Day"], type="l", col="grey")
#27
lines(sortedreshapecfu[sortedreshapecfu$id=="27","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="27", "Day"], type="l", col="grey")
#28
lines(sortedreshapecfu[sortedreshapecfu$id=="28","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="28", "Day"], type="l", col="grey")
#29
lines(sortedreshapecfu[sortedreshapecfu$id=="29","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="29", "Day"], type="l", col="grey")
#30
lines(sortedreshapecfu[sortedreshapecfu$id=="30","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="30", "Day"], type="l", col="grey")
#31
lines(sortedreshapecfu[sortedreshapecfu$id=="31","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="31", "Day"], type="l", col="grey")
#32
lines(sortedreshapecfu[sortedreshapecfu$id=="32","CFU"]~sortedreshapecfu[sortedreshapecfu$id=="32", "Day"], type="l", col="grey")
#adding control
lines(sortedreshapecfu[sortedreshapecfu$treatment=="c","CFU"]~sortedreshapecfu[sortedreshapecfu$treatment=="c","Day"], type="l", col="black", lwd=5)
#adding legend
legend("topright", c("RAG","WT", "Mock"), pch=c(3,1,15), col=c("green","blue", "black"), cex=1, bty="n")
#chi square test for infection outcome by genotype
#reading in compressed table for equal variable lengths
mouseinf_geno<-read.table(file="geno_infstatus.txt", header=T)
ragchisq<-chisq.test(mouseinf_geno$infection_status, mouseinf_geno$genotype)
ragchisq

```
###Figure 9. Discussion and Data Visualization Modifications
In figure 9, Jhansi grouped WT and RAG mice by their *C. diff* status at the time of VPI infection (naive, previously 630 colonized and cleared infection, 630 colonized), and plotted this against the percentage of baseline weight (a measure of mouse illness) post VPI infection. 

This figure demonstrates that mice that are infected by a less virulent strain at the time of infection are protected from CDI illness once infected by a lethal strain, and that adaptive immunity does not modify this effect.

This figure was created in Prism and there are some issues with how the data is displayed that detract from its impact.  Frist, healthy mice should lose the least amount of weight. In this graph, colonized mice have the highest percent of baseline weight (ie their weight has not changed), but it looks as though there is a larger effect in this group because that bar is the highest. Second, this graph is supposed to be a box and whisker plot, however it looks like a bar chart with error bars and many outliers because the first quartile hits the x axis.  These issues have inspired me to display this figure differently in a way that is easier to understand. 

Also, I am not a fan of asterisk and bars across graphs to display significance because I think that they are distracting and lead the viewer to pay attention to the asterisk rather than the data itself. I have decided to perform pariwise t-tests to compare these groups of mice and then report the statistical test and p-value in a subtitle. 

```{r echo=FALSE}

#create strip chart with individual dots as RAG mice
par(mar=c(6,6,4,4), xpd=T)
stripchart(mousecfu[mousecfu$genotype=="RAG", "pBLweight"]~mousecfu[mousecfu$genotype=="RAG","infection_status"], method ="jitter", vertical=F, pch=c(1), col="green", xlab="% Baseline Weight", main="Weight Change By Colonization Status at VPI Infection", las=1, cex=1.5, sub="Pairwise t-test: Colonized vs. Control P<.05, Colonized vs. Cleared P<.05", font.sub=3)
#adding WT mice
stripchart(mousecfu[mousecfu$genotype=="WT", "pBLweight"]~mousecfu[mousecfu$genotype=="WT","infection_status"], method ="jitter", vertical=F, pch=c(1), col="blue", cex=1.5, add=T)
#adding median lines to plot
#median control
points(85.847797,2.992193, col="black", pch="|", cex=2)
#median colonized
points(101.269055,1.991561, col="black", pch="|", cex=2)
#median cleared
points(91.666667,1.002984, col="black", pch="|", cex=2)
#adding IQR
#making IQR coordinate points for each grou
#control IQR 
points( 84.514663, 2.992193, col="black", pch= "l", cex=1.5, lwd=2)
points(87.637788, 2.992193, col="black", pch="l", cex=1.5)
#colonized IQR
points( 98.974400, 1.991561, col="black", pch= "l", cex=1.5, lwd=2)
points(104.097494, 1.991561, col="black", pch="l", cex=1.5, lwd=2)
#cleared IQR vectors
points( 89.629630, 1.002984, col="black", pch= "l", cex=1.5, lwd=2)
points(92.307692, 1.002984, col="black", pch="l", cex=1.5, lwd=2)
#adding legend

legend("bottom", c("RAG","WT"), pch=1, col=c("green","blue"),horiz=T, xpd=T, bty="n", cex=1, inset=c(0,1))

#this is how I got the coordinates for above IQR and medians
#writing function to find median mean and IQR of each pre VPI type
summarystats<-function(x) {
 c( mean=mean(x), quantile=quantile(x), median=median(x), sd=sd(x))
}
sumstatsweight<-tapply(mousecfu$pBLweight, mousecfu$infection_status, summarystats)
sumstatsweight  

#do pairwise t tests comparing naive vs cleared vs colonized
pairwise.t.test(mousecfu$pBLweight, mousecfu$infection_status)
#add significance to strip chart



```
##Discussion
In this project, I have replicated analysis of Jhansi's mouse colonization data.  I have performed statistical tests and created new graphics that display the data in a way that I think is easier to understand.  

Jhansi and I discussed this project as well as the new plots that I created. We both thought it was interesting that I, as someone who had never before seen this data, thought different things should be emphasized than she did as the person who actually performed the experiments and collected the data. 

Additionally, this project taught me that it is difficult to analyze someone elses dataset when you have no idea what variables mean and how it is organized.  Working on this has given me ideas on how to better anotate and store (such as keeping variables in long format vs wide format) my own data so that it will be easier to analyze and to share with collaborators.   
 
##Sources
1. [CDC.gov *C. diff*](http://www.cdc.gov/HAI/organisms/cdiff/Cdiff_infect.html)

