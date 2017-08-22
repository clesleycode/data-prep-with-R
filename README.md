
## Setup

This guide was written in R 3.2.3.


### R and R Studio

Install [R](https://www.r-project.org/) and [R Studio](https://www.rstudio.com/products/rstudio/download/).

Next, to install the R packages, cd into your workspace, and enter the following, very simple, command into your bash: 

```
R
```

This will prompt a session in R! From here, you can install any needed packages. For the sake of this tutorial, enter the following into your terminal R session:

```
install.packages("dplyr")
install.packages("downloader")
```


## Introduction

We've gone over Data Acquisition as of now, so we know how to <i>get</i> our data. But once you have the data, it might not be in the best shape. You might have scraped a bunch of data from a website, but need it in the form of a dataframe to work with it in an easier manner. This process is called data preparation - preparing your data in a format that's easiest to form with.

### Overview

<b> Data Acquisition: </b> Reading and writing with a variety of file formats and databases. <br>
<b> Preparation: </b> Cleaning, munging, combining, normalizing, reshaping, slicing and dicing, and transforming data for analysis. <br>
<b> Transformation: </b> Applying mathematical and statistical operations to groups of data sets to derive new data sets. For example, aggregating a large table by group variables. <br>
<b> Modeling and computation: </b> Connecting your data to statistical models, machine learning algorithms, or other computational tools <br>
<b> Presentation: </b> Creating interactive or static graphical visualizations or textual summaries <br>


## Data Merging

If you encounter two different datasets that contain the same type of information, you might consider merging them for your analyses.  

Let's go through an example containing student data. `d1` contains 5 of the samples and `d2` contains 2 of them: 


```R
d1 <- read.csv("./names_original.csv")
print(d1)
```

First.Name Last.Name
1     Lesley   Cordero
2       Ojas     Sathe
3      Helen      Chen
4        Eli  Epperson
5      Jacob Greenberg



```R
d2 <- read.csv("./names_add.csv")
print(d2)
```

Warning message in read.table(file = file, header = header, sep = sep, quote = quote, :
“incomplete final line found by readTableHeader on './names_add.csv'”

First.Name Last.Name
1     Martin     Perez
2      Menna   Elsayed


Instead of working with two separate datasets, it's much easier to simply merge, so we do this with the `merge()` function:


```R
result <- merge(d1, d2, all=TRUE)
print(result)
```

First.Name Last.Name
1        Eli  Epperson
2      Helen      Chen
3      Jacob Greenberg
4     Lesley   Cordero
5       Ojas     Sathe
6     Martin     Perez
7      Menna   Elsayed


Now, you might be asking what will happen if one of the datasets has more columns than other - will they still be allowed to merge? Let's try this example with another dataset:


```R
d3 <- read.csv("./names_extra.csv")
print(d3)
```

Warning message in read.table(file = file, header = header, sep = sep, quote = quote, :
“incomplete final line found by readTableHeader on './names_extra.csv'”

First.Name Last.Name                  Major
1     Martin     Perez Mechanical Engineering
2      Menna   Elsayed              Sociology


If we use the same `merge` function, we get:


```R
result1 <- merge(d1, d3, all=TRUE)
print(result1)
```

First.Name Last.Name                  Major
1        Eli  Epperson                   <NA>
2      Helen      Chen                   <NA>
3      Jacob Greenberg                   <NA>
4     Lesley   Cordero                   <NA>
5       Ojas     Sathe                   <NA>
6     Martin     Perez Mechanical Engineering
7      Menna   Elsayed              Sociology


Notice the `NA` values - these are undefined values indicating there wasn't any data to be displayed. R will simply fill in the missing data for each sample where it's unavailable.

Now, how do we merge two datasets with differing columns? Well, let's take a look at our datasets:


```R
h1 <- read.csv("./housing.csv", stringsAsFactors=FALSE)
print(h1)
```

Dorm           Name
1 East Campus     Helen Chen
2    Broadway  Danielle Jing
3     Shapiro   Craig Rhodes
4        Watt Lesley Cordero
5 East Campus   Martin Perez
6    Broadway  Menna Elsayed
7     Wallach  Will Essilfie



```R
h2 <- read.csv("./dorms.csv", stringsAsFactors=FALSE)
print(h2)
```

Dorm Street   Cost
1    Broadway  114th   9000
2     Shapiro  115th   9500
3        Watt  113th  10500
4 East Campus  116th 11,000
5     Wallach  114th   9500


With the `merge()` function in pandas, we can specify which column to merge on and what kind of join to specify. By default merge does an 'inner' join, but here we set it to a left join:


```R
house <- merge(h1, h2, by="Dorm")
print(house)
```

Dorm           Name Street   Cost
1    Broadway  Danielle Jing  114th   9000
2    Broadway  Menna Elsayed  114th   9000
3 East Campus     Helen Chen  116th 11,000
4 East Campus   Martin Perez  116th 11,000
5     Shapiro   Craig Rhodes  115th   9500
6     Wallach  Will Essilfie  114th   9500
7        Watt Lesley Cordero  113th  10500


### Dropping Columns

After completing the different operations we've reviewed so far, you'll find that our `house` dataframe now has 4 columns. But what if we figured out that the street isn't so important? Can we just get rid of this column? Turns out we can. 


```R
house1 <- subset(house, select=-c(Street))
print(house1)
```

Dorm           Name   Cost
1    Broadway  Danielle Jing   9000
2    Broadway  Menna Elsayed   9000
3 East Campus     Helen Chen 11,000
4 East Campus   Martin Perez 11,000
5     Shapiro   Craig Rhodes   9500
6     Wallach  Will Essilfie   9500
7        Watt Lesley Cordero  10500


### Changing values

When it comes to changing data in a DataFrame, many times the task is large and non-trivial. For example, what if the price of the East Campus dorm changed to $12,000? Would we really want to go through each column and change these by hand? Or even change them with for loops and indexing? Nah. Luckily, R gives us a short hand way of easily doing this. 
house1$Cost[which(house1$Dorm=="East Campus")] <- 12000
Now, let's break this down. Let's start with the innermost portion, which provides us the Dorm values for each row. 


```R
print(house1$Dorm)
```

[1] "Broadway"    "Broadway"    "East Campus" "East Campus" "Shapiro"    
[6] "Wallach"     "Watt"       


Now, let's add the second part of that innermost statement: 


```R
print(house1$Dorm=="East Campus")
```

[1] FALSE FALSE  TRUE  TRUE FALSE FALSE FALSE


Now, we're testing for equality. This statement provides a vector of booleans that indicate whether or not the dorm is equal to the string "East Campus". 

When we add the `which` function around this statement, R is now selecting which rows output the value `TRUE` in the vector. In this case, it's rows 3 and 4. 


```R
print(which(house1$Dorm=="East Campus"))
```

[1] 3 4


Since we now wrap the statement with house$Cost, it outputs back the cost values associated with the `East Campus`. 


```R
print(house1$Cost[which(house1$Dorm=="East Campus")])
```


<ol class=list-inline>
<li>'11,000'</li>
<li>'11,000'</li>
</ol>



And finally, we reset these rows to the changed value we want of 12,000. 


```R
house1$Cost[which(house1$Dorm=="East Campus")] <- 12000
print(house1)
```

Dorm           Name  Cost
1    Broadway  Danielle Jing  9000
2    Broadway  Menna Elsayed  9000
3 East Campus     Helen Chen 12000
4 East Campus   Martin Perez 12000
5     Shapiro   Craig Rhodes  9500
6     Wallach  Will Essilfie  9500
7        Watt Lesley Cordero 10500



```R

```
