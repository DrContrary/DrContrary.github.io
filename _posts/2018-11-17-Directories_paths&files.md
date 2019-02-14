---
title: "Directories, Paths, and Files"
author: "Alyssa"
date: "November 17, 2018"
---

----------------------------

When you are just starting out, issues with getting to your data can get in the way of learning R. These might be the basics, but if you don't have these down you are going to run into issues quickly.  

While there is nothing wrong with asking for help with these fundamental skills, it also doesn't hurt to have a good go-to reference when questions arise.

## Directories  

In computing, the directory is more an abstract concept. You can think of it as the system of organization used by your computer to store your files. More broadly, this is Where you store your files.  

You may be more familiar with the use of "folders", which are really just a visual representation of a directory. You can think of them same way.

**Working Directory**  
This is the current "folder" that your computer is working in. It goes without saying that it is important to know where you are working in relation to the files you are working with.

**Get your current directory**  

{% highlight r %}
getwd()
{% endhighlight %}
This command has no arguments and running just this function will print the current directory that R is working in.  

**Set your working directory**  

{% highlight r %}
setwd("/path/path/path")
{% endhighlight %}
This command will set your working directory and takes the argument of the filepath as a string (put it in quotes).

**List files in your directory**  

{% highlight r %}
list.files()
{% endhighlight %}
Leaving the arguments of this command blank will print the files currently in your working directory. For the full list of possible arguments, see the documentation <https://stat.ethz.ch/R-manual/R-devel/library/base/html/list.files.html> or use the command ?list.files within R to access the help file.

## R Studio Directories  
If you are working in R Studio, there are a few little GUI features that make beginners feel more comfortable. If you would like, you can use these options to set your working directory and view the contents in your file viewer.

![](https://i.imgur.com/9jVVpIg.jpg)


## Paths  

The path refers to the series of nested directories in which your files lie. You can think of this as the directions to find your files within your computer.

**OS Specific Differences **  
In Linux based systems (including Macs) the filepath is written a bit differently than Windows. It is important to pay attention because this can cause your code to not run.

**Windows**

{% highlight r %}
path <- "\path\path\path\directory\file"
{% endhighlight %}

**Linux and R**

{% highlight r %}
path <- "/path/path/path/directory/file"
{% endhighlight %}

The only difference as you might notice is the use of either a forward slash or back slash. If you work on a Linux machine, you won't need to change anything in your filepaths when working in R. However if you work on Windows machine, you will need to change your back slashes to forward slashes when specifying paths.

## Files  
While there are many different methods to read in many types of files, I will only briefly go over a few methods here.

### R Studio Text File Data Import Options
If you are using R studio you have a couple build in GUI options for import. These are more familiar for people just starting out. Over time you will find there are more efficient ways to import your data that keep analysis as reproduceable as possible.

![](https://i.imgur.com/rIRkBwX.jpg?1)  

**Import using Base R Functions**  
Clicking this option brings you to a fileviewer that allows you find your file of interest for import. Once a txt file has been selected a second menu opens which allows you to change some of the options for import. The name of the data will automatically populate with the file name, however this may be changed prior to import. Once you have made any adjustments, clicking import will load the data into your global environment.  

**Import using readr Functions**  
Of the two txt import options under this menu, I think using readr is the better option for several reasons. First, readr is part of the tidyverse and imports dataframes as tibbles. The user interface is also obviously better and you have more control over the import. You can change the data class here as well simply by clicking the dropdown menu on the column name. Under this menu, you can also skip columns for import that aren't needed. Finally, part of the UI is a code preview. This gets you more familiar with importing data this way and can also be copied and pasted into your script.  

### R Studio Excel File Data Import Options
In reality, Excel is a popular spreadsheet program and you will encounter lots of data as Excel files. You can also import this data using the R Studio UI by selecting the third option.

![](https://i.imgur.com/oU23rZo.jpg)  

**Import using read_excel Functions**  
Much like readr to import text files, read_excel is part of the Tidyverse. As such the UI is very similar and the data is imported as a tibble. Really the only difference here is that you can import excel files and even specify the sheet you want to read in.  

### Code Your File Imports   
After a while you will likely find that all the mouse clicks involved in UI based data import are just slowing you down. To improve your workflow, you will want to start coding your data import.  

The first thing you need to do is set your working directory to where your files are.  

**Set your working directory**

{% highlight r %}
path <- "/path/path/path/"
{% endhighlight %}


**Import CSV Files**  


{% highlight r %}
df = read.csv("file_name.csv")
{% endhighlight %}

With this command we are reading in file_name.csv which is located in the current working directory and naming it df.

**Import Text Files with Other Separators**  
If for example your data uses a . or # as a separator you can import it using read.delim. All you have to do is define the separator. In this example we will import a file that uses the .  


{% highlight r %}
df = read.delim("file_name.txt", sep = ".")
{% endhighlight %}

**Import with the Tidyverse**  
The readr and read_excel functions we used in the UI import can also be coded directly. Simply place the functions in your script following the same format show in the example code. Note that the only difference between read_csv (tidyverse) and read.csv (base R) is the underscore or period.  


{% highlight r %}
df = read_excel("file_name.xlsx")
df = read_csv("file_name.csv")
{% endhighlight %}
