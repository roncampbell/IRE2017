R is an open-source programming language that is widely used by scientists and, increasingly, journalists to analyze and visualize data. The core program, known as "base R", is supplemented by more than 12,000 specialized packages. Think of these packages as tools like the corkscrew, awl and file on a Swiss Army knife -- really useful in special situations.

You can install a package at any time. But in order to use a package during a session, you must, in effect, hit the "on" switch. Here's an example using one wildly popular R package:

<code>install.packages("ggplot2")</code>      # adds package to computer

<code>library(ggplot2)</code>                 # adds package to session

(In R, place a # before comments. The program does not process anything on the same line after the #-sign.)  

If you're just getting started with R, here are my recommendations for packages that you should have: tidyverse (a collection of packages, including ggplot2, from the prolific Hadley Wickham), data.table, stringr, forcats and lubridate.

Before we get started, here are a few rules of the R road: 

Capitalization and punctuation matter. You might have noticed that the syntax for install.packages() included quote marks while the syntax for library() did not. Mix that up and you will get an error. Similarly, if you find yourself dealing with a table called "DataTable" (no quote marks), do not call it datatable or Datatable or DATATABLE. You will get an error. R is unforgiving about spelling, punctuation and capitalization. It is, as I told the first R class I ever taught, "the copy editor from hell."

Second, R uses a lot of variables. It names them with an assignment operator which looks like this:  <-    
You can create it by clicking on the left arrow (found on the shift key above the comma on most keyboards) and then the hyphen. Or you can spare your wrists a lot of effort by learning a shortcut. In Windows the shortcut is <code>Alt -</code>. In Mac the shortcut is <code>Option -</code>. 

Now let's crunch some data! In this introductory class we'll be using base R, so we won't have to use the library() command to install  packages. We will, however, need to import data. 

We'll begin by launching R Studio. R Studio has four panes or windows. Depending upon how your copy is set up (it can be customized), you'll see the Console where you'll do most of your work on lower left, the Source on upper left, Environment and History on upper right and Plots and Packages on lower right. Go to the Session menu and Set Working Directory. I typically set the Working Directory to the same folder where I keep my data; it makes data importing a little easier.

R can import many file types, but the easiest and most flexible is a CSV or "comma-separated variable". If you've worked with Excel or database programs, you've already encountered CSVs. If not, the name explains most but not all of their mysteries. CSV files typically consist of files with columns separated by, yes, commas; if a column contains internal commas then that column should have some demarcation such as quote marks. 

We will be working with two Medicare files on Arizona hospitals, ReadmissionsAndDeaths.csv and PaymentAndValue.csv

The exact syntax for the import command will depend on your working directory and the location of these files. For demo purposes, I set my working directory to the exact location of the files:

> AZ_readmit <- read.csv("./ReadmissionsAndDeaths.csv", header=TRUE, colClasses=c(rep("character",19)))

Let's walk through what we did here: First we assigned a name to the table or in R-speak "data frame"; it's now AZ_readmit. We used the base R function read.csv to go to the working directory and read a CSV file. We specified that there is a header. As for that "colClasses" thing, well, that's complicated. There are 19 columns and the very first column is an ID with (shudder) leading zeroes. Left alone, R would have imported that ID as a number, minus the zero. 

How did I know about the leading zeroes? I looked first in a text editor. This takes almost no time and saves an endless amount of grief.

One other thing: That c() after colClasses is something you will come across a lot in R. It stands for "concatenate" or "combine". In this case it wasn't strictly necessary since there was only one column class; but it's a good habit. 

Now that the data is imported, let's interview it. 

> dim(AZ_readmit)         # displays dimensions of the data frame in rows and columns

[1] 1134   19

> str(AZ_readmit)         # displays dimensions plus for each variable the data type and four samples

> head(AZ_readmit)        # displays first six rows in Console

> View(AZ_readmit)        # displays data frame in Source pane

What is this data all about? Well, the CSV was called ReadmissionsAndDeaths, which sounds promising. And there are two columns called MeasureName and MeasureID. We can get a quick idea of the contents with the table() command which creates a frequency table:

> table(AZ_readmit$MeasureID)

       MORT_30_AMI       MORT_30_CABG       MORT_30_COPD         MORT_30_HF         MORT_30_PN 
                81                 81                 81                 81                 81 
       MORT_30_STK       READM_30_AMI      READM_30_CABG      READM_30_COPD        READM_30_HF 
                81                 81                 81                 81                 81 
 
Notice the syntax: We name the table, a $-sign and the column. We get back a series of measures, some beginning with "MORT" (for "mortality", others with "READM" (for "readmission"). These are measures for 81 Arizona hospitals; they measure how often patients died or were readmitted within 30 days after discharge for conditions like acute myocardial infarction (AMI or heart attack), cardio-arterial bypass graft (CABG), chronic obstructive pulmonary disorder (COPD), heart failure, pneumonia and stroke.
 
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

The aggregate() function lets us compare two or more variables. We begin with a continuous variable (Score), separated by a tilde from a categorical variable (MeasureID), followed by the data frame, and then the function, in this case the mean. In English we want to know the mean Score for each Measure ID. Other functions we could use include median, sum (not relevant here) and length (R-speak for count). 


The scores, by the way, are not death rates; they are "risk-standardized mortality rates" that make possible comparisons among very different hospitals. Bottom line: the lower, the better. 

Next we'll zero in on heart attack deaths. To do that, we'll take a subset of the data. We'll need to grab every case where the MeasureID is MORT_30_AMI. But we also want to eliminate hospitals where a footnote indicates there are no results or too few results.

> AZ_AMI <- subset(AZ_readmit, MeasureID=="MORT_30_AMI" & Footnote=="")

Again let's dissect the command: We assign the new file (AZ_AMI), call the subset() command, the file we want to subset (AZ_readmit) and then name the columns we want to use; we want everything in AZ_readmit where the MeasureID is "MORT_30_AMI" and where the Footnote is empty.

There had been 1,134 rows in our original data frame, and in the frequency table above we had 81 rows for MORT_30_AMI. But after eliminating hospitals with footnotes we wind up with just 45 hospitals in our new data frame.

Now let's take a close look:

> summary(AZ_AMI$Score)

   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
   
  11.30   13.40   13.90   13.99   14.60   16.10 

The summary() function is really handy, but we can get a more granular look by using the quantile() function:

> quantile(AZ_AMI$Score, c(0.1, 0.25, 0.5, 0.75, 0.9))

  10%   25%   50%   75%   90% 
  
12.88 13.40 13.90 14.60 15.16 

Notice that we're using c-for-combine operator!

If you want to visualize this, base R has some simple tools. Packages such as ggplot2 can do much more.

 > hist(AZ_AMI$Score)

 (https://github.com/roncampbell/IRE2017/blob/images/AZ_AMI%24Score_hist.png)

This produces a histogram, similar to a column or bar chart, showing the distribution of values of AMI scores. 

Let's import our second file, PaymentAndValue.csv; this lists average Medicare payments to Arizona hospitals for selected types of medical care. We want to see if there's a relationship between heart attack-related payment rates and mortality scores. To do that we'll have to tweak the payment table and then merge it with AZ_AMI.

> AZ_payment <- read.csv("./PaymentAndValue.csv", header=TRUE, colClasses=c(rep("character",17)))

> View(AZ_payment)

There's something odd about the Payment column - not just the many rows with a "Not Available" label, but the payments are formatted with $-signs and commas. If you try to change this column to a number using the as.numeric() method I showed you earlier, you will get nothing but NA's. (Trust me, I tested this.) The reason is that R treats the $-sign and everything that follows it as a character string. So we have to use a little magic to extract the $-sign and the comma, converting what's left to a number.

> AZ_payment$Payment <- as.numeric(gsub("[\\$,]", "", AZ_payment$Payment))

Warning message:
NAs introduced by coercion 

This is a bit of wizardry known as a "regular expression". The "gsub" is a global substitution; it says to look everywhere in the column Payment for a $-sign and a comma ($,) and to replace them with nothing (""). Once we have done that, working from inside the parentheses to outside, we convert the column to numeric. 

Now we can make a data frame of payments for heart attacks (AMI's).

> AMI_Pay <- subset(AZ_payment, PaymentMeasureID=="PAYM_30_AMI" & Footnote=="")

Next we combine AZ_AMI and AMI_Pay. The two data frames have several fields in common, but the most fool-proof way to merge them is by using the ProviderID. People can always misspell or abbreviate a hospital name or address; not much chance of that happening with a six-digit ID number.

> AMI_Pay_Deaths <- merge(AZ_AMI, AMI_Pay, by="ProviderID")

The syntax is simple: We assign a name for the new data frame, use the  command merge(), followed by the two data frames that we are merging, the key word "by=" and then the column on which we are merging. That's it. The one down side is that we get all the columns in both tables with lots of redundancies. 

We want to know if hospitals with the lowest mortality scores get paid more than lower-performing hospitals. There are several ways to answer that question. But if the answer is obvious, there's an easy way to **see** it. It's called a scatter plot, an arrangement of dots on a horizontal-vertical plane. All we have to do is plot, say, mortality scores on one axis and payments on the other axis and see if the dots form a pattern.

> plot(AMI_Pay_Deaths$Score, AMI_Pay_Deaths$Payment)

(https://github.com/roncampbell/IRE2017/blob/images/Pay_deaths_score.png)

Um, no. If there's a pattern here, I can't see it.

Where our eyes fail, perhaps a statistical test will succeed. Let's try a linear regression. We want to see how Score responds to the independent variable, Payment. So Score will appear on the left side of the tilde. 

> az_model <- lm(Score ~ Payment, data=AMI_Pay_Deaths)

> summary(az_model)

> Call:
> lm(formula = Score ~ Payment, data = AMI_Pay_Deaths)

> Residuals:
>      Min       1Q   Median       3Q      Max 
> -2.38980 -0.70170  0.02236  0.49484  2.41430 
> 
> Coefficients:
>              Estimate Std. Error t value Pr(>|t|)    
> (Intercept) 1.133e+01  2.540e+00    4.46 5.81e-05 
> Payment     1.138e-04  1.084e-04    1.05    0.299    
> 
> Signif. codes:  0 ‘ ’ 0.001 ‘ ’ 0.01 ‘ ’ 0.05 ‘.’ 0.1 ‘ ’ 1
> 
> Residual standard error: 0.9836 on 43 degrees of freedom
> Multiple R-squared:  0.02501,	Adjusted R-squared:  0.002337 
> F-statistic: 1.103 on 1 and 43 DF,  p-value: 0.2995

There's a lot of detail here, but perhaps the most telling is the Adjusted R-squared value. R-squared is the ratio of explained variation (between Score and Payment in this case) and total variation. It's always between 0 and 1. The very low R-squared tells us that the regression model explains almost none of the variance in the Score - Payment regression line. 

So there isn't a relationship between the money Medicare pays hospitals for heart attack care and the 30-day mortality rate. But sometimes in journalism it pays to know right away which stories just are not there.
