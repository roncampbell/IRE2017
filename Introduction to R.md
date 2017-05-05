R is an open-source programming language that is widely used by scientists and, increasingly, journalists to analyze and visualize data. The core program, known as "base R", is supplemented by more than 12,000 specialized packages. Think of these packages as tools like the corkscrew, awl and file on a Swiss Army knife -- really useful in special situations.

You can install a package at any time. But in order to use a package during a session, you must, in effect, hit the "on" switch. Here's an example using one wildly popular R package:

<code>install.packages("tidyverse")</code>      # adds package to computer

<code>library(tidyverse)</code>                 # adds package to session

(In R, place a # before comments. R does not process anything on the same line after the #-sign.)  

If you're just getting started with R, I recommend installing tidyverse, which is actually a collection of several packages from the prolific Hadley Wickham including data manipulation tools dplyr, tidyr, stringr and lubridate and graphics tool ggplot2.

Before we get started, here are a few rules of the R road: 

Capitalization and punctuation matter. You might have noticed that the syntax for install.packages() included quote marks while the syntax for library() did not. Mix that up and you will get an error. Similarly, if you find yourself dealing with a table called "DataTable" (no quote marks), do not call it datatable or Datatable or DATATABLE. You will get an error. R is unforgiving about spelling, punctuation and capitalization. It is, as I told the first R class I ever taught, "the copy editor from hell."

Second, R uses a lot of variables. It names them with an assignment operator which looks like this:  <-    
You can create it by clicking on the left arrow (found on the shift key above the comma on most keyboards) and then the hyphen. Or you can spare your wrists a lot of effort by learning a shortcut. In Windows the shortcut is <code>Alt -</code>. In Mac the shortcut is <code>Option -</code>. 

Now let's crunch some data! In this introductory class we'll be using base R, so we won't have to use the library() command to install  packages. We will, however, need to import data. 

We'll begin by launching R Studio. R Studio has four panes or windows. Depending upon how your copy is set up (it can be customized), you'll see the Console where you'll do most of your work on lower left, the Source on upper left, Environment and History on upper right and Plots and Packages on lower right. Go to the Session menu and Set Working Directory. I typically set the Working Directory to the same folder where I keep my data; it makes data importing a little easier.

R can import many file types, but the easiest and most flexible is a CSV or "comma-separated variable". If you've worked with Excel or database programs, you've already encountered CSVs. If not, the name explains most but not all of their mysteries. CSV files typically consist of files with columns separated by, yes, commas; if a column contains internal commas then that column should have some demarcation such as quote marks. 

We will be working with two Medicare files on Arizona hospitals, ReadmissionsAndDeaths.csv and HospitalAcquiredConditions.csv

The exact syntax for the import command will depend on your working directory and the location of these files. For demo purposes, I set my working directory to the exact location of the files:

> AZ_readmit <- read.csv("./ReadmissionsAndDeaths.csv", header=TRUE, colClasses=c(rep("character",19)))

Let's walk through what we did here: First we assigned a name to the table or in R-speak "data frame"; it's now AZ_readmit. We used the base R function read.csv to go to the working directory and read a CSV file. We specified that there is a header. As for that "colClasses" thing, well, that's complicated. There are 19 columns and the very first column is an ID with (shudder) leading zeroes. Left alone, R would have imported that ID as a number, minus the zero. 

How did I know about the leading zeroes? I looked first in a text editor. This takes almost no time and saves an endless amount of grief.

One other thing: That c() after colClasses is something you will come across a lot in R. It stands for "concatenate" or "combine". In this case it wasn't strictly necessary since there was only one column class; but it's a good habit. 

Now that the data is imported, let's interview it. 

> dim(AZ_readmit)         # displays dimensions of the data frame in rows and columns

[1] 1134   19

> str(AZ_readmit)         # displays the structure: dimensions plus for each variable the data type and four samples

> head(AZ_readmit)        # displays first six rows in Console

> View(AZ_readmit)        # displays a scrollable view of the data frame in Source pane

What is this data all about? Well, the CSV was called ReadmissionsAndDeaths, which sounds promising. And there are two columns called MeasureName and MeasureID. We can get a quick idea of the contents with the table() command, which creates a frequency table:

> table(AZ_readmit$MeasureID)

       MORT_30_AMI       MORT_30_CABG       MORT_30_COPD         MORT_30_HF         MORT_30_PN 
                81                 81                 81                 81                 81 
       MORT_30_STK       READM_30_AMI      READM_30_CABG      READM_30_COPD        READM_30_HF 
                81                 81                 81                 81                 81 
 
Notice the syntax: We list the names of the data frame and column separated by a $-sign. We get back a series of measures, some beginning with "MORT" (for "mortality"), others with "READM" (for "readmission"). There are 81, one for each hospital in Arizona; they measure how often patients died or were readmitted within 30 days after discharge for conditions like acute myocardial infarction (AMI or heart attack), cardio-arterial bypass graft (CABG), chronic obstructive pulmonary disorder (COPD), heart failure (HF), pneumonia (PN) and stroke (STK).
 
Right now every column is a character string. And the key "Score" column is a jumble of numbers and "Not Available" entries. But we can fix that.
 
> AZ_readmit$Score <- as.numeric(AZ_readmit$Score)

Warning message:

NAs introduced by coercion  
 
This command converts the Score field from a character string to a number; "NA", short for "Not Available" appears where there is no data. We can deal with NA values easily in R.

Now let's look at all of the measures at once:

> aggregate(Score ~ MeasureID, AZ_readmit, mean)

            MeasureID     Score
1         MORT_30_AMI 13.988889

2        MORT_30_CABG  3.277778

3        MORT_30_COPD  8.274074

4          MORT_30_HF 11.613725 ...

We use the aggregate() function to combine and compare. We begin with a continuous variable (Score), separated by a tilde from a categorical variable (MeasureID), followed by the data frame, and then the function, in this case the mean. In English we want to know the mean Score for each Measure ID. Other functions we could use include median, sum (not relevant here) and length (R-speak for count). 

The scores, by the way, are not death rates; they are "risk-standardized mortality rates", which make possible comparisons among very different hospitals. Bottom line: the lower, the better. 

Next we'll zero in on heart attack deaths. To do that, we'll take a subset of the data. We'll grab every case where the MeasureID is MORT_30_AMI. But we also want to eliminate hospitals where a footnote indicates there are no results or too few results.

> AZ_AMI <- subset(AZ_readmit, MeasureID=="MORT_30_AMI" & Footnote=="")

Again let's dissect the command: We call the subset() command, the file we want to subset (AZ_readmit) and then name the columns we want to use; we want everything in AZ_readmit where the MeasureID is "MORT_30_AMI" and where the Footnote is empty. We assign all of this to a new data frame called AZ_AMI. 

There had been 1,134 rows in our original data frame, and in the frequency table above we had 81 rows for MORT_30_AMI. But after eliminating hospitals with footnotes we wind up with just 45 hospitals in the new data frame AZ_AMI.

Let's take a close look:

> summary(AZ_AMI$Score)

   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
   
  11.30   13.40   13.90   13.99   14.60   16.10 

The summary() function is handy, but we can get a more granular look by using the quantile() function:

> quantile(AZ_AMI$Score, c(0.1, 0.25, 0.5, 0.75, 0.9))

  10%   25%   50%   75%   90% 
  
12.88 13.40 13.90 14.60 15.16 

Notice that we're using c-for-combine operator!

If you want to visualize this, base R has some simple tools. (Packages such as ggplot2 can do much more.)

 > hist(AZ_AMI$Score)

 (https://github.com/roncampbell/IRE2017/blob/images/AZ_AMI%24Score_hist.png)

This command produces a histogram, similar to a column or bar chart, showing the distribution of values of AMI scores. 

Let's import our second file, HospitalAcquiredConditions.csv; this shows how well or poorly hospitals deal with certain bugs such as MRSA that exist almost exclusively inside healthcare facilities. If their HAC (hospital-acquired condition) scores exceed 6.75, hospitals are docked a percentage of their annual Medicare payments. We want to see if there's a relationship between Arizona hospitals' HAC scores and their 30-day heart attack mortality scores. To do that we'll import this table and merge it with AZ_AMI.

> AZ_infections <- read.csv("./HospitalAcquiredConditions.csv", header=TRUE, colClasses=c(rep("character",16)))

> View(AZ_conditions)

We need to convert a few of the columns from string to numeric. By now this is easy.

> AZ_infections$Domain1_Score <- as.numeric(AZ_infections$Domain1_Score)

> AZ_infections$Domain2_Score <- as.numeric(AZ_infections$Domain2_Score)
Warning message:
NAs introduced by coercion 

> AZ_infections$Total_HAC_Score <- as.numeric(AZ_infections$Total_HAC_Score)

Next we combine AZ_AMI and AMI_infections. The two data frames have several fields in common, but the most fool-proof way to merge them is by using the ProviderID. This is a unique number assigned by Medicare to each hospital. People can always misspell or abbreviate a hospital name or address; not much chance of that happening with a six-digit ID number.

> AMI_AMI_infect <- merge(AZ_AMI, AMI_infections, by="ProviderID")

The syntax here is fairly simple: We assign a name for the new data frame, use the command merge(), followed by the names of the two data frames that we are merging, the key word "by=" and then the column on which we are merging in quotes. (If the merger column had been named differently in the two tables, then we would do it like this: by=c("ProviderID", "Provider_ID"), making sure that we match the order of the data frames in the merge statement.) That's it. The one down side is that we get all the columns in both tables, including several that repeat the same information.  

We want to know if low heart-attack mortality and low HAC scores go together. There are several ways to answer that question. But if the answer is obvious, there's an easy way to **see** it. It's called a scatter plot, an arrangement of dots on a horizontal-vertical plane. All we have to do is plot, say, mortality scores on one axis and payments on the other axis and see if the dots form a pattern.

> plot(AZ_AMI_infect$Score, AZ_AMI_infect$Total_HAC_Score)

(https://github.com/roncampbell/IRE2017/blob/images/AMI_infections.png)

Um, no. If there's a pattern here, I can't see it.

Where our eyes fail, perhaps a statistical test will succeed. Let's try a linear regression. We want to see how Score responds to the independent variable, Total_HAC_Score. So Score will appear on the left side of the tilde. 

> infect_model <- lm(Score ~ Total_HAC_Score, data=AZ_AMI_infect)

> summary(infect_model)

> Call:
> lm(formula = Score ~ Total_HAC_Score, data = AZ_AMI_infect)

> Residuals:
>      Min       1Q   Median       3Q      Max 
> -2.3937 -0.8598  0.1684  0.5709  2.8935 
> 
> Coefficients:
>              Estimate Std. Error t value Pr(>|t|)    
> (Intercept) 14.42464    0.64643  22.314   <2e-16 

> Total_HAC_Score -0.04723    0.10931  -0.432    0.668    
> 
> Signif. codes:  0 ‘ ’ 0.001 ‘ ’ 0.01 ‘ ’ 0.05 ‘.’ 0.1 ‘ ’ 1
> 
> Residual standard error: 1.091 on 42 degrees of freedom

> Multiple R-squared:  0.004425,	
Adjusted R-squared:  -0.01928 

> F-statistic: 0.1867 on 1 and 42 DF,  p-value: 0.6679

There's a lot of detail here, but perhaps the most telling is the Adjusted R-squared value. R-squared is the ratio of explained variation (between Score and Total_HAC_Score in this case) and total variation. It's always between 0 and 1. The very low R-squared tells us that the regression model explains almost none of the variance in the Score - Total_HAC_Score regression line. 

So there surprisingly isn't much of a relationship between hospital-acquired conditions and the 30-day mortality for heart attacks. But sometimes in journalism it pays to know right away which stories just are not there.
