---
layout: tutorial
title: How to Process Data with Apache Hive
tutorial-id: 110
tutorial-series: Basic Development
tutorial-version: hdp-2.4.0
intro-page: true
components: [ ambari, hive ]
---


# How to Process Data with Apache Hive

### Introduction

In this tutorial, we will use the [Ambari](http://hortonworks.com/hadoop/ambari/) HDFS file view to store massive data files of baseball statistics. We will implement [Hive](http://hortonworks.com/hadoop/hive/) queries to analyze, process and filter that data. 

## Pre-Requisites
*  Downloaded and Installed latest [Hortonworks Sandbox](http://hortonworks.com/products/hortonworks-sandbox/#install)
*  [Learning the Ropes of the Hortonworks Sandbox](http://hortonworks.com/hadoop-tutorial/learning-the-ropes-of-the-hortonworks-sandbox/)
*  Allow yourself around one hour to complete this tutorial

## Outline
- [Hive](#hive)
- [Hive or Pig?](#hive-or-pig)
- [Our Data Processing Task](#our-data-processing-task)
- [Step 1: Download The Data](#download-the-data)
- [Step 2: Upload The Data Files](#upload-the-data-files)
- [Step 3: Start the Hive View](#start-the-hive-view)
- [Further Reading](#further-reading)
- [summary](#summary)

## Hive <a id="hive"></a>

Hive is a component of [Hortonworks Data Platform](http://hortonworks.com/hdp/)(HDP). Hive provides a SQL-like interface to data stored in HDP. In the previous tutorial,
we used Pig, which is a scripting language with a focus on dataflows. Hive provides a database query interface to Apache Hadoop.

## Hive or Pig? <a id="hive-or-pig"></a>

People often ask why do [Pig](http://hortonworks.com/hadoop/pig/) and [Hive](http://hortonworks.com/hadoop/hive/) exist when they seem to do much of the same thing. Hive because of its SQL like query language is 
often used as the interface to an Apache Hadoop based data warehouse. Hive is considered friendlier and more familiar to users who are 
used to using SQL for querying data. Pig fits in through its data flow strengths where it takes on the tasks of bringing data into Apache 
Hadoop and working with it to get it into the form for querying. A good overview of how this works is in Alan Gates posting on the Yahoo 
Developer blog titled [Pig and Hive at Yahoo!](https://developer.yahoo.com/blogs/hadoop/pig-hive-yahoo-464.html) From a technical point 
of view, both Pig and Hive are feature complete, so you can do tasks in either tool. However, you will find one tool or the other will be 
preferred by the different groups that have to use Apache Hadoop. The good part is they have a choice and both tools work together.

### Our Data Processing Task <a id="our-data-processing-task"></a>

We are going to do the same data processing task as we just did with Pig in the previous tutorial. We have several files of baseball 
statistics and we are going to bring them into Hive and do some simple computing with them. We are going to find the player with the 
highest runs for each year. This file has all the statistics from 1871–2011 and contains more that 90,000 rows. Once we have the highest 
runs we will extend the script to translate a player id field into the first and last names of the players.

### Step 1: Download The Data <a id="download-the-data"></a>

The data files we are using come from the site [SeanLahman.com](http://www.seanlahman.com/). You can download the following data file,
[lahman591-csv.zip](http://seanlahman.com/files/database/lahman591-csv.zip). Once you have the file you will need to unzip it into a 
directory. We will be uploading just the `Master.csv` and `Batting.csv` files from the dataset.

### Step 2: Upload The Data Files <a id="upload-the-data-files"></a>

We start by selecting the HDFS Files view from the Off-canvas menu at the top. The `HDFS Files view` shows you the files in the HDP file
store. In this case, the file store resides in the Hortonworks Sandbox VM.

![HDFS File View Icon Image](/assets/hello-hdp/hdfs_files_view_hello_hdp_lab1.png)

Navigate to `/user/maria_dev` and click on the **Upload** button.

![HDFS_maria_dev_Folder_Image](/assets/how-to-process-data-with-apache-hive/user_maria_dev_hdfs_process_data_hive.png)

Clicking on browse will open a dialog box. Navigate to where you stored theBatting.csv file on your local disk and select `Batting.csv.` 
Do the same thing for `Master.csv.` When you are done you will see there are two files in your directory.

![batting and master csv uploaded Image](/assets/how-to-process-data-with-apache-hive/stored_two_files_process_data_hive.png)

### Step 3: Start the Hive View <a id="start-the-hive-view"></a>

Lets open the `Hive View`by clicking on the Hive button in the top bar as previously when we selected the HDFS Files view. The Hive view provides a user interface to the Hive data warehouse system for Hadoop.

![Hive View Icon from HDFS warehouse](/assets/how-to-process-data-with-apache-hive/start_hive_view_process_data_hive.png)


#### 3.1 Explore The Hive User Interface

On right is a `query editor`. A query may span multiple lines. At the bottom, there are buttons to `Execute` the query, `Explain` the query, `Save` the query with a name and to open a new Worksheet window for another query.

![hive ui](/assets/how-to-process-data-with-apache-hive/hive_user_interface_process_data_hive.png)


##### Hive and Pig Data Model Differences

Before we get started let’s take a look at how `Pig and Hive` data models differ. In the case of Pig all data objects exist and are operated on in the script. Once the script is complete all data objects are deleted unless you stored them. In the case of Hive we are operating on the Apache Hadoop data store. Any query you make, table that you create, data that you copy persists from query to query. You can think of Hive as providing a data workbench where you can examine, modify and manipulate the data in Apache Hadoop. So when we perform our data processing task we will execute it one query or line at a time. Once a line successfully executes you can look at the data objects to verify if the last operation did what you expected. All your data is live, compared to Pig, where data objects only exist inside the script unless they are copied out to storage. This kind of flexibility is Hive’s strength. You can solve problems bit by bit and change your mind on what to do next depending on what you find.

#### 3.2 Create Table temp_batting

The first task we will do is create a table to hold the data. We will type the query into the `composition area` on the right handside. Once you have typed in the query hit the `Execute` button at the bottom.

~~~sql
create table temp_batting (col_value STRING);
~~~

![temp_batting_query](/assets/how-to-process-data-with-apache-hive/temp_batting_table_process_data_hive.png)

> **Hint:** press `CTRL` + `Space` for autocompletion

The query does not return any results because at this point we just created an empty table and we have not copied any data in it.

Once the query has executed we can refresh the `Database Explorer` at the left of the composition area and when folding down the default database we will see we have a new table called `temp_batting`.

![temp_batting_table_default_database](/assets/how-to-process-data-with-apache-hive/database_explorer_tempbatting_process_data_hive.png)

Clicking on the `icon` next to the table name a new Worksheets opens, which loads `sample data` from this table. We see the table is empty right now. This is a good example of the interactive feel you get with using Hive.

![load_sample_data_temp_batting_empty](/assets/how-to-process-data-with-apache-hive/icon_temp_batting_process_data_hive.png)

The next line of code will load the data file `Batting.csv` into the table `temp_batting`.

![load_battingcsv_into_temp_batting](/assets/how-to-process-data-with-apache-hive/load_data_tempbatting_process_data_hive.png)


#### 3.3 Create Query to Populate Hive Table temp_batting with Batting.csv Data

The complete query looks like this.

~~~sql
LOAD DATA INPATH '/user/maria_dev/Batting.csv' OVERWRITE INTO TABLE temp_batting;
~~~

After executing the query we can look at the Tables again and when we browse the data for `temp_batting` we see that the data has been read in. Note Hive consumed the data file `Batting.csv` during this step. If you look in the `File Browser` you will see Batting.csv is no longer there.

![temp_batting_sample_data_has_data](/assets/how-to-process-data-with-apache-hive/verify_tempbatting_read_data_process_data_hive.png)


#### 3.4 Create Table batting

Now that we have read the data in we can start working with it. The next thing we want to do extract the data. So first we will type in a query to create a new table called `batting` to hold the data. That table will have three columns for `player_id, year and the number of runs`.

~~~sql
create table batting (player_id STRING, year INT, runs INT);
~~~

![batting_query](/assets/how-to-process-data-with-apache-hive/create_batting_table_process_data_hive.png)


#### 3.5 Create Query to Extract Data from temp_batting and Store It to batting

Then we extract the data we want from `temp_batting` and copy it into `batting`. We will do this with a regexp pattern. To do this we are going to build up a multi-line query. The first line of the query create the table `batting`. The three regexp_extract calls are going to extract the `player_id, year and run` fields from the table temp_batting. When you are done typing the query it will look like this. Be careful as there are no spaces in the regular expression pattern.

~~~sql
insert overwrite table batting  
SELECT  
  regexp_extract(col_value, '^(?:([^,]*)\,?){1}', 1) player_id,  
  regexp_extract(col_value, '^(?:([^,]*)\,?){2}', 1) year,  
  regexp_extract(col_value, '^(?:([^,]*)\,?){9}', 1) run  
from temp_batting;
~~~

![filter_batting_player_year_run_query](/assets/how-to-process-data-with-apache-hive/overwrite_batting_table_process_data_hive.png)

Execute the query and look at the `batting table`. You should see data that looks like this.

![load_sample_batting_table_data](/assets/how-to-process-data-with-apache-hive/view_batting_table_data_process_data_hive.png)


#### 3.6 Create Query to Filter The Data (year, runs)

Now we have the data fields we want. The next step is to `group` the data by year so we can find the highest score for each `year`. This query first groups all the records by year and then selects the `player with the highest runs` from each year.

~~~sql
SELECT year, max(runs) FROM batting GROUP BY year;
~~~

The results of the query look like this.

![group_year_highest_runs](/assets/how-to-process-data-with-apache-hive/filter_data_years_runs_process_data_hive.png)


#### 3.7 Create Query to Filter The Data (year, player, runs)

Now we need to go back and get the `player_id(s)` so we know who the player(s) was. We know that for a given year we can use the runs to find the player(s) for that year. So we can take the previous query and join it with the `batting records` to get the final table.

~~~sql
SELECT a.year, a.player_id, a.runs from batting a  
JOIN (SELECT year, max(runs) runs FROM batting GROUP BY year ) b  
ON (a.year = b.year AND a.runs = b.runs);
~~~

The resulting data looks like:

![year_playerid_runs_data_table](/assets/how-to-process-data-with-apache-hive/filter_data_player_runs_year_process_data_hive.png)

So now we have our results. As described earlier we solved this problem using Hive step by step. At any time we were free to look around at the data, decide we needed to do another task and come back. At all times the data is live and accessible to us.

## Summary <a id="summary"></a>

Congratulations on completing this tutorial! We just learned how to upload data into HDFS Files View and create hive queries to manipulate data. Let's review all the queries that were utilized in this tutorial: **create**, **load**, **insert**, **select**, **from**, **group by**, **join** and **on**. With these queries, we created a table _temp_batting_ to store the data. We created another table _batting_, so we can overwrite that table with extracted data from the _temp_batting_ table we created earlier. Finally, created queries to filter the data to have the result show the best player runs each year.

## Further Reading <a id="further-reading"></a>
- [Apache Hive](http://hortonworks.com/hadoop/hive/)
- [Hive Tutorials](http://hortonworks.com/hadoop/hive/#tutorials)
- [Hive Language Manual](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL)
 
