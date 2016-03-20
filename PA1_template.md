---
title: "Reproducible Research 1"
author: "ecvizie"
date: "March 20, 2016"
output: html_document
---


#Assignment #1 - Reproducible Research
## Loading and preprocessing the data
Data taken from <https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip> on 03/20/2016. We'll use a set without NAs to start so I don't get a headache.
<div class="chunk" id="readdata"><div class="rcode"><div class="source"><pre class="knitr r"><span class="hl std">c</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">read.csv</span><span class="hl std">(</span><span class="hl str">'activity.csv'</span><span class="hl std">)</span>
<span class="hl std">d</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">na.omit</span><span class="hl std">(c)</span> <span class="hl com">#Create a copy with omitted NAs</span>
<span class="hl std">d</span><span class="hl opt">$</span><span class="hl std">date</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">as.Date</span><span class="hl std">(d</span><span class="hl opt">$</span><span class="hl std">date,</span> <span class="hl str">'%Y-%m-%d'</span><span class="hl std">)</span> <span class="hl com">#Convert dates            </span>
<span class="hl std">agg</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">aggregate</span><span class="hl std">(d</span><span class="hl opt">$</span><span class="hl std">steps,</span> <span class="hl kwd">list</span><span class="hl std">(d</span><span class="hl opt">$</span><span class="hl std">date),</span> <span class="hl kwc">FUN</span><span class="hl std">=sum)</span> <span class="hl com">#Aggregate and return total steps per day</span>
<span class="hl kwd">hist</span><span class="hl std">(agg</span><span class="hl opt">$</span><span class="hl std">x,</span> <span class="hl kwc">main</span><span class="hl std">=</span><span class="hl str">&quot;Total # of Steps Taken Per Day&quot;</span><span class="hl std">,</span> <span class="hl kwc">xlab</span><span class="hl std">=</span><span class="hl str">&quot;Steps Taken per Day&quot;</span><span class="hl std">)</span>
</pre></div>
<div class="rimage default"><img src="figure/readdata-1.png" title="plot of chunk readdata" alt="plot of chunk readdata" class="plot" /></div>
</div></div>

##Calculate Summary Data for the Daily Data Set
<div class="chunk" id="calcAvgMed"><div class="rcode"><div class="source"><pre class="knitr r"><span class="hl kwd">summary</span><span class="hl std">(agg</span><span class="hl opt">$</span><span class="hl std">x)</span>
</pre></div>
<div class="output"><pre class="knitr r">##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    8841   10760   10770   13290   21190
</pre></div>
</div></div>

##Time series plot of the average number of steps taken
<div class="chunk" id="timeseries1"><div class="rcode"><div class="source"><pre class="knitr r"><span class="hl com">#Aggregate by interval across all days</span>
<span class="hl std">agginterval</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">aggregate</span><span class="hl std">(d</span><span class="hl opt">$</span><span class="hl std">steps,</span> <span class="hl kwd">list</span><span class="hl std">(d</span><span class="hl opt">$</span><span class="hl std">interval),</span> <span class="hl kwc">FUN</span><span class="hl std">=mean)</span>
<span class="hl com">#Time series plot by interval</span>
<span class="hl kwd">plot</span><span class="hl std">(x</span> <span class="hl opt">~</span> <span class="hl std">Group.1, agginterval,</span> <span class="hl kwc">type</span> <span class="hl std">=</span> <span class="hl str">&quot;l&quot;</span><span class="hl std">,</span>
     <span class="hl kwc">main</span><span class="hl std">=</span><span class="hl str">&quot;Avg Steps Per Interval&quot;</span><span class="hl std">,</span> <span class="hl kwc">xlab</span><span class="hl std">=</span><span class="hl str">'Interval'</span><span class="hl std">,</span> <span class="hl kwc">ylab</span><span class="hl std">=</span><span class="hl str">&quot;Avg # of Steps&quot;</span><span class="hl std">)</span>
</pre></div>
<div class="rimage default"><img src="figure/timeseries1-1.png" title="plot of chunk timeseries1" alt="plot of chunk timeseries1" class="plot" /></div>
</div></div>

##The 5-minute interval that, on average, contains the maximum number of steps
<div class="chunk" id="maxinterval"><div class="rcode"><div class="source"><pre class="knitr r"><span class="hl std">maxint</span><span class="hl kwb">&lt;-</span> <span class="hl std">agginterval[agginterval</span><span class="hl opt">$</span><span class="hl std">x</span><span class="hl opt">==</span><span class="hl kwd">max</span><span class="hl std">(agginterval</span><span class="hl opt">$</span><span class="hl std">x),]</span>
<span class="hl std">maxint</span><span class="hl opt">$</span><span class="hl std">Group.1</span>
</pre></div>
<div class="output"><pre class="knitr r">## [1] 835
</pre></div>
</div></div>

##Code to describe and show a strategy for imputing missing data
We will use the average for that interval to fill the missing data. We accomplish this by merging datasets based on the interval period and then setting the NAs to the constant value.
<div class="chunk" id="missingdata"><div class="rcode"><div class="source"><pre class="knitr r"><span class="hl com">#How many NAs do we have?</span>
<span class="hl kwd">length</span><span class="hl std">(c[</span><span class="hl kwd">is.na</span><span class="hl std">(c)])</span>
</pre></div>
<div class="output"><pre class="knitr r">## [1] 2304
</pre></div>
<div class="source"><pre class="knitr r"><span class="hl com">#use interval average to fill data</span>
<span class="hl std">merged</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">merge</span><span class="hl std">(agginterval, c,</span> <span class="hl kwc">by.x</span><span class="hl std">=</span><span class="hl str">&quot;Group.1&quot;</span><span class="hl std">,</span> <span class="hl kwc">by.y</span> <span class="hl std">=</span> <span class="hl str">&quot;interval&quot;</span><span class="hl std">)</span>
<span class="hl std">merged[</span><span class="hl kwd">is.na</span><span class="hl std">(merged</span><span class="hl opt">$</span><span class="hl std">steps)</span><span class="hl opt">==</span><span class="hl num">TRUE</span><span class="hl std">,]</span><span class="hl opt">$</span><span class="hl std">steps</span> <span class="hl kwb">&lt;-</span> <span class="hl std">merged[</span><span class="hl kwd">is.na</span><span class="hl std">(merged</span><span class="hl opt">$</span><span class="hl std">steps)</span><span class="hl opt">==</span><span class="hl num">TRUE</span><span class="hl std">,]</span><span class="hl opt">$</span><span class="hl std">x</span>
<span class="hl std">merged</span><span class="hl opt">$</span><span class="hl std">date</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">as.Date</span><span class="hl std">(merged</span><span class="hl opt">$</span><span class="hl std">date,</span> <span class="hl str">'%Y-%m-%d'</span><span class="hl std">)</span>
<span class="hl std">merged</span><span class="hl opt">$</span><span class="hl std">dow</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">weekdays</span><span class="hl std">(merged</span><span class="hl opt">$</span><span class="hl std">date,</span> <span class="hl kwc">abbreviate</span> <span class="hl std">=</span> <span class="hl num">TRUE</span><span class="hl std">)</span>
<span class="hl std">merged</span><span class="hl opt">$</span><span class="hl std">wknd</span> <span class="hl kwb">&lt;-</span> <span class="hl std">(merged</span><span class="hl opt">$</span><span class="hl std">dow</span> <span class="hl opt">%in%</span> <span class="hl kwd">c</span><span class="hl std">(</span><span class="hl str">&quot;Sat&quot;</span><span class="hl std">,</span> <span class="hl str">&quot;Sun&quot;</span><span class="hl std">))</span>
<span class="hl std">aggm</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">aggregate</span><span class="hl std">(merged</span><span class="hl opt">$</span><span class="hl std">steps,</span> <span class="hl kwd">list</span><span class="hl std">(merged</span><span class="hl opt">$</span><span class="hl std">date),</span> <span class="hl kwc">FUN</span><span class="hl std">=sum)</span>
<span class="hl kwd">summary</span><span class="hl std">(aggm</span><span class="hl opt">$</span><span class="hl std">x)</span>
</pre></div>
<div class="output"><pre class="knitr r">##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    9819   10770   10770   12810   21190
</pre></div>
<div class="source"><pre class="knitr r"><span class="hl com">#Histogram of Filled Data</span>
<span class="hl kwd">hist</span><span class="hl std">(aggm</span><span class="hl opt">$</span><span class="hl std">x,</span> <span class="hl kwc">main</span><span class="hl std">=</span><span class="hl str">&quot;Total # of Steps Taken Per Day&quot;</span><span class="hl std">,</span> <span class="hl kwc">xlab</span><span class="hl std">=</span><span class="hl str">&quot;Steps Taken per Day&quot;</span><span class="hl std">)</span>
</pre></div>
<div class="rimage default"><img src="figure/missingdata-1.png" title="plot of chunk missingdata" alt="plot of chunk missingdata" class="plot" /></div>
</div></div>

##Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends
Group by the weekend feature and then by interval feature producing an average number of steps.
<div class="chunk" id="weekenddaycompare"><div class="rcode"><div class="source"><pre class="knitr r"><span class="hl std">aggminterval</span> <span class="hl kwb">&lt;-</span> <span class="hl kwd">aggregate</span><span class="hl std">(merged</span><span class="hl opt">$</span><span class="hl std">steps,</span> <span class="hl kwd">list</span><span class="hl std">(merged</span><span class="hl opt">$</span><span class="hl std">wknd, merged</span><span class="hl opt">$</span><span class="hl std">Group.1),</span> <span class="hl kwc">FUN</span><span class="hl std">=mean)</span>

<span class="hl kwd">par</span><span class="hl std">(</span><span class="hl kwc">mfrow</span><span class="hl std">=</span><span class="hl kwd">c</span><span class="hl std">(</span><span class="hl num">2</span><span class="hl std">,</span><span class="hl num">1</span><span class="hl std">))</span>
<span class="hl kwd">plot</span><span class="hl std">(x</span> <span class="hl opt">~</span> <span class="hl std">Group.2, aggminterval[aggminterval</span><span class="hl opt">$</span><span class="hl std">Group.1</span><span class="hl opt">==</span><span class="hl num">TRUE</span><span class="hl std">,],</span> <span class="hl kwc">type</span> <span class="hl std">=</span> <span class="hl str">&quot;l&quot;</span><span class="hl std">,</span>
     <span class="hl kwc">main</span><span class="hl std">=</span><span class="hl str">&quot;Steps taken per day (Weekend)&quot;</span><span class="hl std">,</span> <span class="hl kwc">xlab</span><span class="hl std">=</span><span class="hl str">&quot;&quot;</span><span class="hl std">,</span> <span class="hl kwc">ylab</span><span class="hl std">=</span><span class="hl str">&quot;# of Steps&quot;</span><span class="hl std">)</span>
<span class="hl kwd">plot</span><span class="hl std">(x</span> <span class="hl opt">~</span> <span class="hl std">Group.2, aggminterval[aggminterval</span><span class="hl opt">$</span><span class="hl std">Group.1</span><span class="hl opt">==</span><span class="hl num">FALSE</span><span class="hl std">,],</span> <span class="hl kwc">type</span> <span class="hl std">=</span> <span class="hl str">&quot;l&quot;</span><span class="hl std">,</span>
     <span class="hl kwc">main</span><span class="hl std">=</span><span class="hl str">&quot;Steps taken per day (Weekday)&quot;</span><span class="hl std">,</span> <span class="hl kwc">xlab</span><span class="hl std">=</span><span class="hl str">&quot;Interval&quot;</span><span class="hl std">,</span><span class="hl kwc">ylab</span><span class="hl std">=</span><span class="hl str">&quot;# of Steps&quot;</span><span class="hl std">)</span>
</pre></div>
<div class="rimage default"><img src="figure/weekenddaycompare-1.png" title="plot of chunk weekenddaycompare" alt="plot of chunk weekenddaycompare" class="plot" /></div>
</div></div>
