---
layout: post
title: Assessing SAT Scores for New York Schools
subtitle: Exploratory data analysis of SAT scores across high schools
cover-img: /assets/img/sat-img.jpg
tags: [SAT, exam, EDA]
---

The **SAT (Scholastic Aptitude Test)** is a standardized test widely used for college admissions in the United States. The test is divided into 3 sections, each of which has a total score of 800. High schools are often ranked by their average SAT scores, and high SAT scores are considered a sign of how good a school district is.

This project utilizes different datasets taken from the [NYC OpenData initiative](https://opendata.cityofnewyork.us/). The main idea here was to combine data from multiple associated data sources and perform exploratory data analysis on it to understand:

1. Are schools differentiated by SAT scores?
2. The schools which are doing best and the ones which are doing worse, what are the key differentiating factors between these schools?
3. Do demographics really play a role in SAT scores? If yes, then how and which demographic categories are doing better than others  

### Step - 1: Data pull and cleaning

For purpose of this analysis, we will be taking multiple open-source data sets available, perform some cleaning, and consolidate them into one dataset having one row per school, which would then allow us to do the rest of Exploratory Data Analysis pretty easily. As I would be using the NYC OpenData APIs to pull in the data, I have added the links to APIs for downloading the data directly into pandas dataframes. 

Before we start with the datasets, I would like to take a moment and explain how the data hierarchy works in these datasets. I did some research and found out that New York is divided into districts (denoted by numbers like 01,02, etc.), which are then further divided into boroughs, which then have schools. Thus, each school is identified by its **DBN (District Borough Number)** which looks something like _"01M015"_.

The datasets that we would be using in this project are:

- **Sat Scores** - SAT Scores by sections for each school. Download JSON [here](https://data.cityofnewyork.us/resource/f9bf-2cp4.json)
- **Enrollment** - Average attendance & Number of students enrolled at a district level. Download JSON [here](https://data.cityofnewyork.us/resource/7z8d-msnt.json)
- **High Schools** - A plethora of details about every school such as location, address, website, etc. Download JSON [here](https://data.cityofnewyork.us/resource/n3p6-zve2.json)
- **Class Size** - Details about school class sizes & education programs. Download JSON [here](https://data.cityofnewyork.us/resource/urz7-pzb3.json)
- **Advanced Placement** - Summary of students taking advanced placement exams by school. Download JSON [here](https://data.cityofnewyork.us/resource/itfs-ms3e.json)
- **Graduation** - Graduation related stats for each school by cohorts. Download JSON [here](https://data.cityofnewyork.us/resource/vh2h-md7a.json)
- **Demographics** - Student demographics information for each school. Download JSON [here](https://data.cityofnewyork.us/resource/ihfw-zy9j.json)
- **Districts** - District coordinates and shape polygons. You will need to download this data in GeoJSON format in order to use it for plotting. Download them from [here](https://data.cityofnewyork.us/Education/School-Districts/r8nu-ymqj)
- **Math test results** - Math test results for every school. Download JSON [here](https://data.cityofnewyork.us/resource/jufi-gzgp.json)
- **Survey Results** - Survey responses of students, teachers, and parents for each school providing overview of their perception of school quality. This is not available through APIs and has to be downloaded from [here](https://data.cityofnewyork.us/Education/2011-NYC-School-Survey/mnz3-dyi8)

Having gotten enough context, let's get started with this project!!

First, we will import the required libraries using the below code:
```python
#Importing all relevant libraries
import numpy as np
import pandas as pd
import os
import folium # For plotting chloropleth charts
from folium import plugins
```

Now, let's read in all the different datasets that I described above:
```python
#Using Department of Education APIs to read in all relevant data
sat_results = pd.read_json("https://data.cityofnewyork.us/resource/f9bf-2cp4.json")
enrollment = pd.read_json("https://data.cityofnewyork.us/resource/7z8d-msnt.json")
high_schools = pd.read_json("https://data.cityofnewyork.us/resource/n3p6-zve2.json")
class_size = pd.read_json("https://data.cityofnewyork.us/resource/urz7-pzb3.json")
ap2010 = pd.read_json("https://data.cityofnewyork.us/resource/itfs-ms3e.json")
graduation = pd.read_json("https://data.cityofnewyork.us/resource/vh2h-md7a.json")
demographics = pd.read_json("https://data.cityofnewyork.us/resource/ihfw-zy9j.json")
math_results = pd.read_json("https://data.cityofnewyork.us/resource/jufi-gzgp.json")
survey_1 = pd.read_csv("masterfile11_gened_final.txt",delimiter='\t',encoding='windows-1252')
survey_2 = pd.read_csv("masterfile11_d75_final.txt",delimiter='\t',encoding='windows-1252')
```

Having the data in place, its always good to check samples for each data set to understand it better
```python
#Browsing the datasets one by one
sat_results.head()
```

<div class="table-container">
<table>
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dbn</th>
      <th>num_of_sat_test_takers</th>
      <th>sat_critical_reading_avg_score</th>
      <th>sat_math_avg_score</th>
      <th>sat_writing_avg_score</th>
      <th>school_name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>01M292</td>
      <td>29</td>
      <td>355</td>
      <td>404</td>
      <td>363</td>
      <td>HENRY STREET SCHOOL FOR INTERNATIONAL STUDIES</td>
    </tr>
    <tr>
      <th>1</th>
      <td>01M448</td>
      <td>91</td>
      <td>383</td>
      <td>423</td>
      <td>366</td>
      <td>UNIVERSITY NEIGHBORHOOD HIGH SCHOOL</td>
    </tr>
    <tr>
      <th>2</th>
      <td>01M450</td>
      <td>70</td>
      <td>377</td>
      <td>402</td>
      <td>370</td>
      <td>EAST SIDE COMMUNITY SCHOOL</td>
    </tr>
    <tr>
      <th>3</th>
      <td>01M458</td>
      <td>7</td>
      <td>414</td>
      <td>401</td>
      <td>359</td>
      <td>FORSYTH SATELLITE ACADEMY</td>
    </tr>
    <tr>
      <th>4</th>
      <td>01M509</td>
      <td>44</td>
      <td>390</td>
      <td>433</td>
      <td>384</td>
      <td>MARTA VALLE HIGH SCHOOL</td>
    </tr>
  </tbody>
</table>
</div>




```python
enrollment.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>district</th>
      <th>ytd_attendance_avg_</th>
      <th>ytd_enrollment_avg_</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>DISTRICT 01</td>
      <td>91.18</td>
      <td>12367</td>
    </tr>
    <tr>
      <th>1</th>
      <td>DISTRICT 02</td>
      <td>89.01</td>
      <td>60823</td>
    </tr>
    <tr>
      <th>2</th>
      <td>DISTRICT 03</td>
      <td>89.28</td>
      <td>21962</td>
    </tr>
    <tr>
      <th>3</th>
      <td>DISTRICT 04</td>
      <td>91.13</td>
      <td>14252</td>
    </tr>
    <tr>
      <th>4</th>
      <td>DISTRICT 05</td>
      <td>89.08</td>
      <td>13170</td>
    </tr>
  </tbody>
</table>
</div>




```python
high_schools.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>:@computed_region_92fq_4b7q</th>
      <th>:@computed_region_efsh_h5xi</th>
      <th>:@computed_region_f5dn_yrer</th>
      <th>:@computed_region_sbqj_enih</th>
      <th>:@computed_region_yeji_bk3q</th>
      <th>addtl_info1</th>
      <th>addtl_info2</th>
      <th>advancedplacement_courses</th>
      <th>bbl</th>
      <th>bin</th>
      <th>...</th>
      <th>school_name</th>
      <th>school_sports</th>
      <th>school_type</th>
      <th>se_services</th>
      <th>start_time</th>
      <th>state_code</th>
      <th>subway</th>
      <th>total_students</th>
      <th>website</th>
      <th>zip</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>47</td>
      <td>20529.0</td>
      <td>51</td>
      <td>59</td>
      <td>3</td>
      <td>Uniform Required: plain white collared shirt, ...</td>
      <td>Extended Day Program, Student Summer Orientati...</td>
      <td>Calculus AB, English Language and Composition,...</td>
      <td>4.157360e+09</td>
      <td>4300730.0</td>
      <td>...</td>
      <td>Frederick Douglass Academy VI High School</td>
      <td>Step Team, Modern Dance, Hip Hop Dance</td>
      <td>NaN</td>
      <td>This school will provide students with disabil...</td>
      <td>2020-04-28 07:45:00</td>
      <td>NY</td>
      <td>A to Beach 25th St-Wavecrest</td>
      <td>412.0</td>
      <td>http://schools.nyc.gov/schoolportals/27/Q260</td>
      <td>11691</td>
    </tr>
    <tr>
      <th>1</th>
      <td>45</td>
      <td>17616.0</td>
      <td>21</td>
      <td>35</td>
      <td>2</td>
      <td>Our school requires completion of a Common Cor...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>3.068830e+09</td>
      <td>3186454.0</td>
      <td>...</td>
      <td>Life Academy High School for Film and Music</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>This school will provide students with disabil...</td>
      <td>2020-04-28 08:15:00</td>
      <td>NY</td>
      <td>D to 25th Ave ; N to Ave U ; N to Gravesend - ...</td>
      <td>260.0</td>
      <td>http://schools.nyc.gov/schoolportals/21/K559</td>
      <td>11214</td>
    </tr>
    <tr>
      <th>2</th>
      <td>49</td>
      <td>18181.0</td>
      <td>69</td>
      <td>52</td>
      <td>2</td>
      <td>Dress Code Required: solid white shirt/blouse,...</td>
      <td>Student Summer Orientation, Weekend Program of...</td>
      <td>English Language and Composition, United State...</td>
      <td>3.016160e+09</td>
      <td>3393805.0</td>
      <td>...</td>
      <td>Frederick Douglass Academy IV Secondary School</td>
      <td>Basketball Team</td>
      <td>NaN</td>
      <td>This school will provide students with disabil...</td>
      <td>2020-04-28 08:00:00</td>
      <td>NY</td>
      <td>J to Kosciusko St ; M, Z to Myrtle Ave</td>
      <td>155.0</td>
      <td>http://schools.nyc.gov/schoolportals/16/K393</td>
      <td>11221</td>
    </tr>
    <tr>
      <th>3</th>
      <td>31</td>
      <td>11611.0</td>
      <td>58</td>
      <td>26</td>
      <td>5</td>
      <td>All students are individually programmed (base...</td>
      <td>Extended Day Program</td>
      <td>Art History, English Language and Composition,...</td>
      <td>2.036040e+09</td>
      <td>2022205.0</td>
      <td>...</td>
      <td>Pablo Neruda Academy</td>
      <td>Baseball, Basketball, Flag Football, Soccer, S...</td>
      <td>NaN</td>
      <td>This school will provide students with disabil...</td>
      <td>2020-04-28 08:00:00</td>
      <td>NY</td>
      <td>N/A</td>
      <td>335.0</td>
      <td>www.pablonerudaacademy.org</td>
      <td>10473</td>
    </tr>
    <tr>
      <th>4</th>
      <td>19</td>
      <td>12420.0</td>
      <td>20</td>
      <td>12</td>
      <td>4</td>
      <td>Chancellor’s Arts Endorsed Diploma</td>
      <td>NaN</td>
      <td>Art History, Biology, Calculus AB, Calculus BC...</td>
      <td>1.011560e+09</td>
      <td>1030341.0</td>
      <td>...</td>
      <td>Fiorello H. LaGuardia High School of Music &amp; A...</td>
      <td>NaN</td>
      <td>Specialized School</td>
      <td>This school will provide students with disabil...</td>
      <td>2020-04-28 08:00:00</td>
      <td>NY</td>
      <td>1 to 66th St - Lincoln Center ; 2, 3 to 72nd S...</td>
      <td>2730.0</td>
      <td>www.laguardiahs.org</td>
      <td>10023</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 69 columns</p>
</div>




```python
class_size.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>average_class_size</th>
      <th>borough</th>
      <th>core_course_ms_core_and_9_12_only_</th>
      <th>core_subject_ms_core_and_9_12_only_</th>
      <th>csd</th>
      <th>data_source</th>
      <th>grade_</th>
      <th>number_of_sections</th>
      <th>number_of_students_seats_filled</th>
      <th>program_type</th>
      <th>school_code</th>
      <th>school_name</th>
      <th>schoolwide_pupil_teacher_ratio</th>
      <th>service_category_k_9_only_</th>
      <th>size_of_largest_class</th>
      <th>size_of_smallest_class</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>19.0</td>
      <td>M</td>
      <td>-</td>
      <td>-</td>
      <td>1</td>
      <td>ATS</td>
      <td>0K</td>
      <td>1.0</td>
      <td>19.0</td>
      <td>GEN ED</td>
      <td>M015</td>
      <td>P.S. 015 Roberto Clemente</td>
      <td>NaN</td>
      <td>-</td>
      <td>19.0</td>
      <td>19.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>21.0</td>
      <td>M</td>
      <td>-</td>
      <td>-</td>
      <td>1</td>
      <td>ATS</td>
      <td>0K</td>
      <td>1.0</td>
      <td>21.0</td>
      <td>CTT</td>
      <td>M015</td>
      <td>P.S. 015 Roberto Clemente</td>
      <td>NaN</td>
      <td>-</td>
      <td>21.0</td>
      <td>21.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>17.0</td>
      <td>M</td>
      <td>-</td>
      <td>-</td>
      <td>1</td>
      <td>ATS</td>
      <td>01</td>
      <td>1.0</td>
      <td>17.0</td>
      <td>GEN ED</td>
      <td>M015</td>
      <td>P.S. 015 Roberto Clemente</td>
      <td>NaN</td>
      <td>-</td>
      <td>17.0</td>
      <td>17.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>17.0</td>
      <td>M</td>
      <td>-</td>
      <td>-</td>
      <td>1</td>
      <td>ATS</td>
      <td>01</td>
      <td>1.0</td>
      <td>17.0</td>
      <td>CTT</td>
      <td>M015</td>
      <td>P.S. 015 Roberto Clemente</td>
      <td>NaN</td>
      <td>-</td>
      <td>17.0</td>
      <td>17.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>15.0</td>
      <td>M</td>
      <td>-</td>
      <td>-</td>
      <td>1</td>
      <td>ATS</td>
      <td>02</td>
      <td>1.0</td>
      <td>15.0</td>
      <td>GEN ED</td>
      <td>M015</td>
      <td>P.S. 015 Roberto Clemente</td>
      <td>NaN</td>
      <td>-</td>
      <td>15.0</td>
      <td>15.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
ap2010.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ap_test_takers_</th>
      <th>dbn</th>
      <th>number_of_exams_with_scores_3_4_or_5</th>
      <th>schoolname</th>
      <th>total_exams_taken</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>39.0</td>
      <td>01M448</td>
      <td>10.0</td>
      <td>UNIVERSITY NEIGHBORHOOD H.S.</td>
      <td>49.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>19.0</td>
      <td>01M450</td>
      <td>NaN</td>
      <td>EAST SIDE COMMUNITY HS</td>
      <td>21.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>24.0</td>
      <td>01M515</td>
      <td>24.0</td>
      <td>LOWER EASTSIDE PREP</td>
      <td>26.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>255.0</td>
      <td>01M539</td>
      <td>191.0</td>
      <td>NEW EXPLORATIONS SCI,TECH,MATH</td>
      <td>377.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NaN</td>
      <td>02M296</td>
      <td>NaN</td>
      <td>High School of Hospitality Management</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
graduation.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>advanced_regents_n</th>
      <th>advanced_regents_of_cohort</th>
      <th>advanced_regents_of_grads</th>
      <th>cohort</th>
      <th>dbn</th>
      <th>demographic</th>
      <th>dropped_out_n</th>
      <th>dropped_out_of_cohort</th>
      <th>local_n</th>
      <th>local_of_cohort</th>
      <th>...</th>
      <th>regents_w_o_advanced_of_grads</th>
      <th>school_name</th>
      <th>still_enrolled_n</th>
      <th>still_enrolled_of_cohort</th>
      <th>total_cohort</th>
      <th>total_grads_n</th>
      <th>total_grads_of_cohort</th>
      <th>total_regents_n</th>
      <th>total_regents_of_cohort</th>
      <th>total_regents_of_grads</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>s</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2003</td>
      <td>01M292</td>
      <td>Total Cohort</td>
      <td>s</td>
      <td>NaN</td>
      <td>s</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>HENRY STREET SCHOOL FOR INTERNATIONAL</td>
      <td>s</td>
      <td>NaN</td>
      <td>5</td>
      <td>s</td>
      <td>NaN</td>
      <td>s</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2004</td>
      <td>01M292</td>
      <td>Total Cohort</td>
      <td>3</td>
      <td>5.5</td>
      <td>20</td>
      <td>36.4</td>
      <td>...</td>
      <td>45.9</td>
      <td>HENRY STREET SCHOOL FOR INTERNATIONAL</td>
      <td>15</td>
      <td>27.3</td>
      <td>55</td>
      <td>37</td>
      <td>67.3</td>
      <td>17</td>
      <td>30.9</td>
      <td>45.9</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2005</td>
      <td>01M292</td>
      <td>Total Cohort</td>
      <td>9</td>
      <td>14.1</td>
      <td>16</td>
      <td>25.0</td>
      <td>...</td>
      <td>62.8</td>
      <td>HENRY STREET SCHOOL FOR INTERNATIONAL</td>
      <td>9</td>
      <td>14.1</td>
      <td>64</td>
      <td>43</td>
      <td>67.2</td>
      <td>27</td>
      <td>42.2</td>
      <td>62.8</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2006</td>
      <td>01M292</td>
      <td>Total Cohort</td>
      <td>11</td>
      <td>14.1</td>
      <td>7</td>
      <td>9.0</td>
      <td>...</td>
      <td>83.7</td>
      <td>HENRY STREET SCHOOL FOR INTERNATIONAL</td>
      <td>16</td>
      <td>20.5</td>
      <td>78</td>
      <td>43</td>
      <td>55.1</td>
      <td>36</td>
      <td>46.2</td>
      <td>83.7</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2006 Aug</td>
      <td>01M292</td>
      <td>Total Cohort</td>
      <td>11</td>
      <td>14.1</td>
      <td>7</td>
      <td>9.0</td>
      <td>...</td>
      <td>84.1</td>
      <td>HENRY STREET SCHOOL FOR INTERNATIONAL</td>
      <td>15</td>
      <td>19.2</td>
      <td>78</td>
      <td>44</td>
      <td>56.4</td>
      <td>37</td>
      <td>47.4</td>
      <td>84.1</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 23 columns</p>
</div>




```python
demographics.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>asian_num</th>
      <th>asian_per</th>
      <th>black_num</th>
      <th>black_per</th>
      <th>ctt_num</th>
      <th>dbn</th>
      <th>ell_num</th>
      <th>ell_percent</th>
      <th>female_num</th>
      <th>female_per</th>
      <th>...</th>
      <th>male_per</th>
      <th>name</th>
      <th>prek</th>
      <th>schoolyear</th>
      <th>selfcontained_num</th>
      <th>sped_num</th>
      <th>sped_percent</th>
      <th>total_enrollment</th>
      <th>white_num</th>
      <th>white_per</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10</td>
      <td>3.6</td>
      <td>74</td>
      <td>26.3</td>
      <td>25</td>
      <td>01M015</td>
      <td>36</td>
      <td>12.8</td>
      <td>123.0</td>
      <td>43.8</td>
      <td>...</td>
      <td>56.2</td>
      <td>P.S. 015 ROBERTO CLEMENTE</td>
      <td>15</td>
      <td>20052006</td>
      <td>9</td>
      <td>57.0</td>
      <td>20.3</td>
      <td>281</td>
      <td>5</td>
      <td>1.8</td>
    </tr>
    <tr>
      <th>1</th>
      <td>18</td>
      <td>7.4</td>
      <td>68</td>
      <td>28.0</td>
      <td>19</td>
      <td>01M015</td>
      <td>38</td>
      <td>15.6</td>
      <td>103.0</td>
      <td>42.4</td>
      <td>...</td>
      <td>57.6</td>
      <td>P.S. 015 ROBERTO CLEMENTE</td>
      <td>15</td>
      <td>20062007</td>
      <td>15</td>
      <td>55.0</td>
      <td>22.6</td>
      <td>243</td>
      <td>4</td>
      <td>1.6</td>
    </tr>
    <tr>
      <th>2</th>
      <td>16</td>
      <td>6.1</td>
      <td>77</td>
      <td>29.5</td>
      <td>20</td>
      <td>01M015</td>
      <td>52</td>
      <td>19.9</td>
      <td>118.0</td>
      <td>45.2</td>
      <td>...</td>
      <td>54.8</td>
      <td>P.S. 015 ROBERTO CLEMENTE</td>
      <td>18</td>
      <td>20072008</td>
      <td>14</td>
      <td>60.0</td>
      <td>23.0</td>
      <td>261</td>
      <td>7</td>
      <td>2.7</td>
    </tr>
    <tr>
      <th>3</th>
      <td>16</td>
      <td>6.3</td>
      <td>75</td>
      <td>29.8</td>
      <td>21</td>
      <td>01M015</td>
      <td>48</td>
      <td>19.0</td>
      <td>103.0</td>
      <td>40.9</td>
      <td>...</td>
      <td>59.1</td>
      <td>P.S. 015 ROBERTO CLEMENTE</td>
      <td>17</td>
      <td>20082009</td>
      <td>17</td>
      <td>62.0</td>
      <td>24.6</td>
      <td>252</td>
      <td>7</td>
      <td>2.8</td>
    </tr>
    <tr>
      <th>4</th>
      <td>16</td>
      <td>7.7</td>
      <td>67</td>
      <td>32.2</td>
      <td>14</td>
      <td>01M015</td>
      <td>40</td>
      <td>19.2</td>
      <td>84.0</td>
      <td>40.4</td>
      <td>...</td>
      <td>59.6</td>
      <td>P.S. 015 ROBERTO CLEMENTE</td>
      <td>16</td>
      <td>20092010</td>
      <td>14</td>
      <td>46.0</td>
      <td>22.1</td>
      <td>208</td>
      <td>6</td>
      <td>2.9</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 38 columns</p>
</div>

```python
math_results.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>category</th>
      <th>dbn</th>
      <th>grade</th>
      <th>level_1_1</th>
      <th>level_1_2</th>
      <th>level_2_1</th>
      <th>level_2_2</th>
      <th>level_3_1</th>
      <th>level_3_2</th>
      <th>level_3_4_1</th>
      <th>level_3_4_2</th>
      <th>level_4_1</th>
      <th>level_4_2</th>
      <th>mean_scale_score</th>
      <th>number_tested</th>
      <th>year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>All Students</td>
      <td>01M015</td>
      <td>3</td>
      <td>2.0</td>
      <td>5.1</td>
      <td>11.0</td>
      <td>28.2</td>
      <td>20.0</td>
      <td>51.3</td>
      <td>26.0</td>
      <td>66.7</td>
      <td>6.0</td>
      <td>15.4</td>
      <td>667.0</td>
      <td>39</td>
      <td>2006</td>
    </tr>
    <tr>
      <th>1</th>
      <td>All Students</td>
      <td>01M015</td>
      <td>3</td>
      <td>2.0</td>
      <td>6.5</td>
      <td>3.0</td>
      <td>9.7</td>
      <td>22.0</td>
      <td>71.0</td>
      <td>26.0</td>
      <td>83.9</td>
      <td>4.0</td>
      <td>12.9</td>
      <td>672.0</td>
      <td>31</td>
      <td>2007</td>
    </tr>
    <tr>
      <th>2</th>
      <td>All Students</td>
      <td>01M015</td>
      <td>3</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>6.0</td>
      <td>16.2</td>
      <td>29.0</td>
      <td>78.4</td>
      <td>31.0</td>
      <td>83.8</td>
      <td>2.0</td>
      <td>5.4</td>
      <td>668.0</td>
      <td>37</td>
      <td>2008</td>
    </tr>
    <tr>
      <th>3</th>
      <td>All Students</td>
      <td>01M015</td>
      <td>3</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>4.0</td>
      <td>12.1</td>
      <td>28.0</td>
      <td>84.8</td>
      <td>29.0</td>
      <td>87.9</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>668.0</td>
      <td>33</td>
      <td>2009</td>
    </tr>
    <tr>
      <th>4</th>
      <td>All Students</td>
      <td>01M015</td>
      <td>3</td>
      <td>6.0</td>
      <td>23.1</td>
      <td>12.0</td>
      <td>46.2</td>
      <td>6.0</td>
      <td>23.1</td>
      <td>8.0</td>
      <td>30.8</td>
      <td>2.0</td>
      <td>7.7</td>
      <td>677.0</td>
      <td>26</td>
      <td>2010</td>
    </tr>
  </tbody>
</table>
</div>




```python
survey_1.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dbn</th>
      <th>bn</th>
      <th>schoolname</th>
      <th>d75</th>
      <th>studentssurveyed</th>
      <th>highschool</th>
      <th>schooltype</th>
      <th>rr_s</th>
      <th>rr_t</th>
      <th>rr_p</th>
      <th>...</th>
      <th>s_N_q14e_3</th>
      <th>s_N_q14e_4</th>
      <th>s_N_q14f_1</th>
      <th>s_N_q14f_2</th>
      <th>s_N_q14f_3</th>
      <th>s_N_q14f_4</th>
      <th>s_N_q14g_1</th>
      <th>s_N_q14g_2</th>
      <th>s_N_q14g_3</th>
      <th>s_N_q14g_4</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>01M015</td>
      <td>M015</td>
      <td>P.S. 015 Roberto Clemente</td>
      <td>0</td>
      <td>No</td>
      <td>0.0</td>
      <td>Elementary School</td>
      <td>NaN</td>
      <td>88</td>
      <td>60</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>01M019</td>
      <td>M019</td>
      <td>P.S. 019 Asher Levy</td>
      <td>0</td>
      <td>No</td>
      <td>0.0</td>
      <td>Elementary School</td>
      <td>NaN</td>
      <td>100</td>
      <td>60</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>01M020</td>
      <td>M020</td>
      <td>P.S. 020 Anna Silver</td>
      <td>0</td>
      <td>No</td>
      <td>0.0</td>
      <td>Elementary School</td>
      <td>NaN</td>
      <td>88</td>
      <td>73</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>01M034</td>
      <td>M034</td>
      <td>P.S. 034 Franklin D. Roosevelt</td>
      <td>0</td>
      <td>Yes</td>
      <td>0.0</td>
      <td>Elementary / Middle School</td>
      <td>89.0</td>
      <td>73</td>
      <td>50</td>
      <td>...</td>
      <td>20.0</td>
      <td>16.0</td>
      <td>23.0</td>
      <td>54.0</td>
      <td>33.0</td>
      <td>29.0</td>
      <td>31.0</td>
      <td>46.0</td>
      <td>16.0</td>
      <td>8.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>01M063</td>
      <td>M063</td>
      <td>P.S. 063 William McKinley</td>
      <td>0</td>
      <td>No</td>
      <td>0.0</td>
      <td>Elementary School</td>
      <td>NaN</td>
      <td>100</td>
      <td>60</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 1942 columns</p>
</div>




```python
survey_2.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dbn</th>
      <th>bn</th>
      <th>schoolname</th>
      <th>d75</th>
      <th>studentssurveyed</th>
      <th>highschool</th>
      <th>schooltype</th>
      <th>rr_s</th>
      <th>rr_t</th>
      <th>rr_p</th>
      <th>...</th>
      <th>s_q14_2</th>
      <th>s_q14_3</th>
      <th>s_q14_4</th>
      <th>s_q14_5</th>
      <th>s_q14_6</th>
      <th>s_q14_7</th>
      <th>s_q14_8</th>
      <th>s_q14_9</th>
      <th>s_q14_10</th>
      <th>s_q14_11</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>75K004</td>
      <td>K004</td>
      <td>P.S. K004</td>
      <td>1</td>
      <td>Yes</td>
      <td>0.0</td>
      <td>District 75 Special Education</td>
      <td>38.0</td>
      <td>90</td>
      <td>72</td>
      <td>...</td>
      <td>29.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>75K036</td>
      <td>K036</td>
      <td>P.S. 36</td>
      <td>1</td>
      <td>Yes</td>
      <td>NaN</td>
      <td>District 75 Special Education</td>
      <td>70.0</td>
      <td>69</td>
      <td>44</td>
      <td>...</td>
      <td>20.0</td>
      <td>27.0</td>
      <td>19.0</td>
      <td>9.0</td>
      <td>2.0</td>
      <td>6.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>75K053</td>
      <td>K053</td>
      <td>P.S. K053</td>
      <td>1</td>
      <td>Yes</td>
      <td>NaN</td>
      <td>District 75 Special Education</td>
      <td>94.0</td>
      <td>97</td>
      <td>53</td>
      <td>...</td>
      <td>14.0</td>
      <td>12.0</td>
      <td>12.0</td>
      <td>10.0</td>
      <td>21.0</td>
      <td>13.0</td>
      <td>11.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>75K077</td>
      <td>K077</td>
      <td>P.S. K077</td>
      <td>1</td>
      <td>Yes</td>
      <td>NaN</td>
      <td>District 75 Special Education</td>
      <td>95.0</td>
      <td>65</td>
      <td>55</td>
      <td>...</td>
      <td>14.0</td>
      <td>14.0</td>
      <td>7.0</td>
      <td>11.0</td>
      <td>16.0</td>
      <td>10.0</td>
      <td>6.0</td>
      <td>4.0</td>
      <td>7.0</td>
      <td>7.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>75K140</td>
      <td>K140</td>
      <td>P.S. K140</td>
      <td>1</td>
      <td>Yes</td>
      <td>0.0</td>
      <td>District 75 Special Education</td>
      <td>77.0</td>
      <td>70</td>
      <td>42</td>
      <td>...</td>
      <td>35.0</td>
      <td>34.0</td>
      <td>17.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 1773 columns</p>
</div>



**Things we can start noticing from previews:**
    1. DBN is one common field that is present in majority of the datasets. We can combine these datasets on DBN
    2. All datasets do not have one unique row for each school
    3. All schools listed in these datasets are not high-schools
    4. We can choose the important fields from survey data and combine the two survey datasets into one

Okay, so having said that we need to create a DBN field in the class_size dataset. Looking at what DBN looks like, we can see it is a combination of district, borough, and school code. Fragments of dbn are already lying around in class_size, which we need to concatenate into one.

```python
class_size["dbn"] = class_size.apply(lambda x: "{0:02d}{1}".format(x["csd"], x["school_code"]), axis=1)
```


```python
class_size['dbn'].head(10)
```




    0    01M015
    1    01M015
    2    01M015
    3    01M015
    4    01M015
    5    01M015
    6    01M015
    7    01M015
    8    01M015
    9    01M015
    Name: dbn, dtype: object



**Perfect! This looks like a dbn code now.**

Let's now combine the two survey datasets and only keep the fields seeming important


```python
survey = pd.concat([survey_1,survey_2],axis=0)
```

```python
survey_fields = ["dbn", "rr_s", "rr_t", "rr_p", "N_s", "N_t", "N_p", "saf_p_11", "com_p_11", "eng_p_11", "aca_p_11", "saf_t_11", "com_t_11", "eng_t_10", "aca_t_11", "saf_s_11", "com_s_11", "eng_s_11", "aca_s_11", "saf_tot_11", "com_tot_11", "eng_tot_11", "aca_tot_11"]
survey = survey.loc[:,survey_fields]
```

Before joining the datasets, let's reduce datasets to 1 row per school dbn. Starting with class_size


```python
#Only the grade 9-12 is relevant to our analysis 
class_size['grade_'].unique()
```




    array(['0K', '01', '02', '03', '04', '05', '0K-09', nan, '06', '07', '08',
           'MS Core', '09-12'], dtype=object)




```python
#Majority of the grade 9-12 programs are general education
#Here CTT stands for Collabortive Team Teaching where both Gened and Special Ed teachers teach together in the class
#For now, let's keep only the gened program
class_size[class_size['grade_']=='09-12'].groupby('program_type').count()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>average_class_size</th>
      <th>borough</th>
      <th>core_course_ms_core_and_9_12_only_</th>
      <th>core_subject_ms_core_and_9_12_only_</th>
      <th>csd</th>
      <th>data_source</th>
      <th>grade_</th>
      <th>number_of_sections</th>
      <th>number_of_students_seats_filled</th>
      <th>school_code</th>
      <th>school_name</th>
      <th>schoolwide_pupil_teacher_ratio</th>
      <th>service_category_k_9_only_</th>
      <th>size_of_largest_class</th>
      <th>size_of_smallest_class</th>
      <th>dbn</th>
    </tr>
    <tr>
      <th>program_type</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>CTT</th>
      <td>51</td>
      <td>51</td>
      <td>51</td>
      <td>51</td>
      <td>51</td>
      <td>51</td>
      <td>51</td>
      <td>51</td>
      <td>51</td>
      <td>51</td>
      <td>51</td>
      <td>0</td>
      <td>51</td>
      <td>51</td>
      <td>51</td>
      <td>51</td>
    </tr>
    <tr>
      <th>GEN ED</th>
      <td>149</td>
      <td>149</td>
      <td>149</td>
      <td>149</td>
      <td>149</td>
      <td>149</td>
      <td>149</td>
      <td>149</td>
      <td>149</td>
      <td>149</td>
      <td>149</td>
      <td>0</td>
      <td>149</td>
      <td>149</td>
      <td>149</td>
      <td>149</td>
    </tr>
    <tr>
      <th>SPEC ED</th>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>0</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
      <td>16</td>
    </tr>
  </tbody>
</table>
</div>




```python
class_size = class_size[class_size["grade_"] == "09-12"]
class_size = class_size[class_size["program_type"] == "GEN ED"]
class_size = class_size.groupby("dbn").agg(np.mean)
class_size.reset_index(inplace=True)
class_size.rename(columns = {'grade_':'grade'}, inplace = True)
```

Moving on to demographics


```python
#Let's keep only the latest year
demographics['schoolyear'].unique()
```




    array([20052006, 20062007, 20072008, 20082009, 20092010, 20102011,
           20112012], dtype=int64)




```python
demographics = demographics[demographics['schoolyear']==20112012]
```


```python
#Ensuring there are no duplicate dbn rows
len(demographics)-demographics['dbn'].nunique()
```




    0



Moving on to math results


```python
#Taking results for the highest grade
math_results['grade'].unique()
```




    array(['3', '4', '5', '6', 'All Grades', '7', '8'], dtype=object)




```python
#Taking results for the latest year
math_results['year'].unique()
```




    array([2006, 2007, 2008, 2009, 2010, 2011], dtype=int64)




```python
math_results = math_results[(math_results['grade']=='8') & (math_results['year']==2011)]
```


```python
#Ensuring there are no duplicate dbn rows
len(math_results)-math_results['dbn'].nunique()
```




    0



Moving on to graduation


```python
#Taking the latest 2006 cohort
graduation['cohort'].unique()
```




    array(['2003', '2004', '2005', '2006', '2006 Aug', '2001', '2002'],
          dtype=object)




```python
#Taking the overall cohort
graduation['demographic'].unique()
```




    array(['Total Cohort', 'Asian', 'Male'], dtype=object)




```python
graduation = graduation[(graduation['cohort']=='2006') & (graduation['demographic']=='Total Cohort')]
```


```python
len(graduation) - graduation['dbn'].nunique()
```




    0




```python
#This one is already reduced
len(high_schools) - high_schools['dbn'].nunique()
```




    0




```python
#This one requires fixing
len(ap2010) - ap2010['dbn'].nunique()
```




    1




```python
ap2010 = ap2010.groupby('dbn').agg(np.mean)
ap2010.reset_index(inplace=True)
```


```python
#Checking it is fixed now
len(ap2010) - ap2010['dbn'].nunique()
```




    0




```python
#This one is already reduced
len(survey) - survey['dbn'].nunique()
```




    0




```python
#Ensuring this is reduced
len(sat_results) - sat_results['dbn'].nunique()
```




    0



**Now that we are done cleaning the data sets. Let's proceed to creating few fields which do not exist and combine the datasets.** 

#### There are some entries in sat_results which do not have any score associated. Let's remove them from the analysis as there is no legit way to impute these values


```python
sat_results = sat_results[sat_results['sat_math_avg_score']!='s']
```


```python
#Converting sat scores to numeric valu
cols = ['sat_math_avg_score', 'sat_critical_reading_avg_score', 'sat_writing_avg_score']
for c in cols:
    sat_results[c] = sat_results[c].apply(lambda x: int(x))
```


```python
# Adding in total average SAT score as a new field
sat_results['sat_score'] = sat_results['sat_critical_reading_avg_score']+sat_results['sat_math_avg_score']+sat_results['sat_writing_avg_score']
```


```python
# Extracting latitude and longitude out of the location information provided in high school data
high_schools['Latitude'] = high_schools['location_1'].apply(lambda x: x.get('latitude'))
high_schools['Longitude'] = high_schools['location_1'].apply(lambda x: x.get('longitude'))
```


```python
# Checking the number of unique DBNs in each database
data_list = {'sat_results':sat_results,
             'enrollment':enrollment,
             'high_schools':high_schools,
             'class_size':class_size,
             'ap2010':ap2010,
             'graduation':graduation,
             'demographics':demographics,
             'districts':districts,
             'math_results':math_results,
             'survey':survey}

for k,v in data_list.items():
    print(k)
    try:
        print(v['dbn'].nunique())
    except Exception as e:
        print(e)
```

    sat_results
    421
    enrollment
    'dbn'
    high_schools
    435
    class_size
    15
    ap2010
    257
    graduation
    154
    demographics
    151
    districts
    'dbn'
    math_results
    12
    survey
    1702
    

Hmm, looks like survey leads in number of dbns by a huge margin. 
One join approach can be to use that as base and left join every other table to that. But that will result in a high number of blank rows (Not apt for our analysis)
Let's check how many DBNs we have in common between sat_results and high_schools. Maybe we could utilize that number as the base and fetch information for those from other tables


```python
for k,v in data_list.items():
    print(k)
    try:
        print(len(set(sat_results['dbn']).intersection(v['dbn'])))
    except Exception as e:
        print("Exception {0} caught".format(e))
```

    sat_results
    421
    enrollment
    Exception 'dbn' caught
    high_schools
    339
    class_size
    10
    ap2010
    251
    graduation
    148
    demographics
    67
    districts
    Exception 'dbn' caught
    math_results
    3
    survey
    418
    

From what I observe above, I believe it would be best to take an outer join of sat_results, high_schools, ap2010, and graduation.
Then we can left join other datasets to it.


```python
full = pd.merge(sat_results,high_schools,on='dbn',how='outer')
```


```python
full = pd.merge(full,ap2010,on='dbn',how='outer')
```


```python
full = pd.merge(full,graduation,on='dbn',how='outer')
```


```python
for k,v in data_list.items():
    if k not in ['sat_results','high_schools','ap2010','graduation','enrollment','districts']:
        full = pd.merge(full,v,on='dbn',how='left')
```


```python
np.set_printoptions(threshold=200)
```


```python
cols = full.columns.tolist()
```


```python
full[['ap_test_takers_','number_of_exams_with_scores_3_4_or_5','total_exams_taken']]
```




    ['dbn',
     'num_of_sat_test_takers',
     'sat_critical_reading_avg_score',
     'sat_math_avg_score',
     'sat_writing_avg_score',
     'school_name_x',
     'sat_score',
     ':@computed_region_92fq_4b7q',
     ':@computed_region_efsh_h5xi',
     ':@computed_region_f5dn_yrer',
     ':@computed_region_sbqj_enih',
     ':@computed_region_yeji_bk3q',
     'addtl_info1',
     'addtl_info2',
     'advancedplacement_courses',
     'bbl',
     'bin',
     'boro',
     'building_code',
     'bus',
     'campus_name',
     'census_tract',
     'city',
     'community_board',
     'council_district',
     'ell_programs',
     'end_time',
     'expgrade_span_max',
     'expgrade_span_min',
     'extracurricular_activities',
     'fax_number',
     'grade_span_max',
     'grade_span_min',
     'language_classes',
     'location_1',
     'nta',
     'number_programs',
     'online_ap_courses',
     'online_language_courses',
     'overview_paragraph',
     'partner_cbo',
     'partner_corporate',
     'partner_cultural',
     'partner_financial',
     'partner_highered',
     'partner_hospital',
     'partner_nonprofit',
     'partner_other',
     'phone_number',
     'primary_address_line_1',
     'priority01',
     'priority02',
     'priority03',
     'priority04',
     'priority05',
     'priority06',
     'priority07',
     'priority08',
     'priority09',
     'priority10',
     'program_highlights',
     'psal_sports_boys',
     'psal_sports_coed',
     'psal_sports_girls',
     'school_accessibility_description',
     'school_name_y',
     'school_sports',
     'school_type',
     'se_services',
     'start_time',
     'state_code',
     'subway',
     'total_students',
     'website',
     'zip',
     'Latitude',
     'Longitude',
     'ap_test_takers_',
     'number_of_exams_with_scores_3_4_or_5',
     'total_exams_taken',
     'advanced_regents_n',
     'advanced_regents_of_cohort',
     'advanced_regents_of_grads',
     'cohort',
     'demographic',
     'dropped_out_n',
     'dropped_out_of_cohort',
     'local_n',
     'local_of_cohort',
     'local_of_grads',
     'regents_w_o_advanced_n',
     'regents_w_o_advanced_of_cohort',
     'regents_w_o_advanced_of_grads',
     'school_name',
     'still_enrolled_n',
     'still_enrolled_of_cohort',
     'total_cohort',
     'total_grads_n',
     'total_grads_of_cohort',
     'total_regents_n',
     'total_regents_of_cohort',
     'total_regents_of_grads',
     'average_class_size',
     'csd',
     'number_of_sections',
     'number_of_students_seats_filled',
     'schoolwide_pupil_teacher_ratio',
     'size_of_largest_class',
     'size_of_smallest_class',
     'asian_num',
     'asian_per',
     'black_num',
     'black_per',
     'ctt_num',
     'ell_num',
     'ell_percent',
     'female_num',
     'female_per',
     'fl_percent',
     'frl_percent',
     'grade1',
     'grade10',
     'grade11',
     'grade12',
     'grade2',
     'grade3',
     'grade4',
     'grade5',
     'grade6',
     'grade7',
     'grade8',
     'grade9',
     'hispanic_num',
     'hispanic_per',
     'k',
     'male_num',
     'male_per',
     'name',
     'prek',
     'schoolyear',
     'selfcontained_num',
     'sped_num',
     'sped_percent',
     'total_enrollment',
     'white_num',
     'white_per',
     'category',
     'grade',
     'level_1_1',
     'level_1_2',
     'level_2_1',
     'level_2_2',
     'level_3_1',
     'level_3_2',
     'level_3_4_1',
     'level_3_4_2',
     'level_4_1',
     'level_4_2',
     'mean_scale_score',
     'number_tested',
     'year',
     'rr_s',
     'rr_t',
     'rr_p',
     'N_s',
     'N_t',
     'N_p',
     'saf_p_11',
     'com_p_11',
     'eng_p_11',
     'aca_p_11',
     'saf_t_11',
     'com_t_11',
     'eng_t_10',
     'aca_t_11',
     'saf_s_11',
     'com_s_11',
     'eng_s_11',
     'aca_s_11',
     'saf_tot_11',
     'com_tot_11',
     'eng_tot_11',
     'aca_tot_11']




```python
pd.set_option('display.max_columns', None)
pd.set_option('display.max_rows', None)

full.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>sat_critical_reading_avg_score</th>
      <th>sat_math_avg_score</th>
      <th>sat_writing_avg_score</th>
      <th>sat_score</th>
      <th>:@computed_region_92fq_4b7q</th>
      <th>:@computed_region_efsh_h5xi</th>
      <th>:@computed_region_f5dn_yrer</th>
      <th>:@computed_region_sbqj_enih</th>
      <th>:@computed_region_yeji_bk3q</th>
      <th>bbl</th>
      <th>bin</th>
      <th>census_tract</th>
      <th>community_board</th>
      <th>council_district</th>
      <th>expgrade_span_max</th>
      <th>expgrade_span_min</th>
      <th>grade_span_max</th>
      <th>grade_span_min</th>
      <th>number_programs</th>
      <th>total_students</th>
      <th>zip</th>
      <th>ap_test_takers_</th>
      <th>number_of_exams_with_scores_3_4_or_5</th>
      <th>total_exams_taken</th>
      <th>advanced_regents_of_cohort</th>
      <th>advanced_regents_of_grads</th>
      <th>dropped_out_of_cohort</th>
      <th>local_of_cohort</th>
      <th>local_of_grads</th>
      <th>regents_w_o_advanced_of_cohort</th>
      <th>regents_w_o_advanced_of_grads</th>
      <th>still_enrolled_of_cohort</th>
      <th>total_cohort</th>
      <th>total_grads_of_cohort</th>
      <th>total_regents_of_cohort</th>
      <th>total_regents_of_grads</th>
      <th>average_class_size</th>
      <th>csd</th>
      <th>number_of_sections</th>
      <th>number_of_students_seats_filled</th>
      <th>schoolwide_pupil_teacher_ratio</th>
      <th>size_of_largest_class</th>
      <th>size_of_smallest_class</th>
      <th>asian_num</th>
      <th>asian_per</th>
      <th>black_num</th>
      <th>black_per</th>
      <th>ell_percent</th>
      <th>female_num</th>
      <th>female_per</th>
      <th>frl_percent</th>
      <th>hispanic_num</th>
      <th>hispanic_per</th>
      <th>male_num</th>
      <th>male_per</th>
      <th>schoolyear</th>
      <th>sped_num</th>
      <th>sped_percent</th>
      <th>total_enrollment</th>
      <th>white_num</th>
      <th>white_per</th>
      <th>level_1_1</th>
      <th>level_1_2</th>
      <th>level_2_1</th>
      <th>level_2_2</th>
      <th>level_3_1</th>
      <th>level_3_2</th>
      <th>level_3_4_1</th>
      <th>level_3_4_2</th>
      <th>level_4_1</th>
      <th>level_4_2</th>
      <th>mean_scale_score</th>
      <th>number_tested</th>
      <th>year</th>
      <th>rr_s</th>
      <th>rr_t</th>
      <th>rr_p</th>
      <th>N_s</th>
      <th>N_t</th>
      <th>N_p</th>
      <th>saf_p_11</th>
      <th>com_p_11</th>
      <th>eng_p_11</th>
      <th>aca_p_11</th>
      <th>saf_t_11</th>
      <th>com_t_11</th>
      <th>eng_t_10</th>
      <th>aca_t_11</th>
      <th>saf_s_11</th>
      <th>com_s_11</th>
      <th>eng_s_11</th>
      <th>aca_s_11</th>
      <th>saf_tot_11</th>
      <th>com_tot_11</th>
      <th>eng_tot_11</th>
      <th>aca_tot_11</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>421.000000</td>
      <td>421.000000</td>
      <td>421.000000</td>
      <td>421.000000</td>
      <td>435.000000</td>
      <td>434.000000</td>
      <td>435.000000</td>
      <td>435.000000</td>
      <td>435.000000</td>
      <td>4.310000e+02</td>
      <td>4.310000e+02</td>
      <td>432.000000</td>
      <td>432.000000</td>
      <td>432.000000</td>
      <td>33.000000</td>
      <td>31.000000</td>
      <td>435.000000</td>
      <td>432.000000</td>
      <td>435.000000</td>
      <td>426.000000</td>
      <td>435.000000</td>
      <td>233.000000</td>
      <td>151.000000</td>
      <td>233.000000</td>
      <td>147.000000</td>
      <td>147.000000</td>
      <td>147.000000</td>
      <td>147.000000</td>
      <td>147.000000</td>
      <td>147.000000</td>
      <td>147.000000</td>
      <td>147.000000</td>
      <td>154.000000</td>
      <td>147.000000</td>
      <td>147.000000</td>
      <td>147.000000</td>
      <td>10.000000</td>
      <td>10.000000</td>
      <td>10.000000</td>
      <td>10.000000</td>
      <td>0.0</td>
      <td>10.000000</td>
      <td>10.000000</td>
      <td>77.000000</td>
      <td>77.000000</td>
      <td>77.000000</td>
      <td>77.000000</td>
      <td>77.000000</td>
      <td>77.000000</td>
      <td>77.000000</td>
      <td>77.000000</td>
      <td>77.000000</td>
      <td>77.000000</td>
      <td>77.000000</td>
      <td>77.000000</td>
      <td>77.0</td>
      <td>77.000000</td>
      <td>77.000000</td>
      <td>77.000000</td>
      <td>77.000000</td>
      <td>77.000000</td>
      <td>3.000000</td>
      <td>3.000000</td>
      <td>3.000000</td>
      <td>3.000000</td>
      <td>3.000000</td>
      <td>3.000000</td>
      <td>3.000000</td>
      <td>3.000000</td>
      <td>3.000000</td>
      <td>3.000000</td>
      <td>3.000000</td>
      <td>3.000000</td>
      <td>3.0</td>
      <td>466.000000</td>
      <td>466.000000</td>
      <td>466.000000</td>
      <td>462.000000</td>
      <td>466.000000</td>
      <td>466.00000</td>
      <td>466.000000</td>
      <td>466.000000</td>
      <td>466.000000</td>
      <td>466.000000</td>
      <td>466.000000</td>
      <td>466.000000</td>
      <td>0.0</td>
      <td>466.000000</td>
      <td>462.000000</td>
      <td>462.000000</td>
      <td>462.000000</td>
      <td>462.000000</td>
      <td>466.000000</td>
      <td>466.000000</td>
      <td>466.000000</td>
      <td>466.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>400.850356</td>
      <td>413.368171</td>
      <td>393.985748</td>
      <td>1208.204276</td>
      <td>28.832184</td>
      <td>14232.228111</td>
      <td>38.372414</td>
      <td>36.949425</td>
      <td>3.452874</td>
      <td>2.510921e+09</td>
      <td>2.581724e+06</td>
      <td>3988.340278</td>
      <td>6.787037</td>
      <td>21.856481</td>
      <td>12.363636</td>
      <td>8.516129</td>
      <td>11.894253</td>
      <td>8.442130</td>
      <td>1.708046</td>
      <td>703.842723</td>
      <td>10726.096552</td>
      <td>118.879828</td>
      <td>143.615894</td>
      <td>180.532189</td>
      <td>10.892517</td>
      <td>13.720408</td>
      <td>10.261224</td>
      <td>13.964626</td>
      <td>25.677551</td>
      <td>38.980272</td>
      <td>60.600680</td>
      <td>22.377551</td>
      <td>141.480519</td>
      <td>63.834014</td>
      <td>49.873469</td>
      <td>74.322449</td>
      <td>21.978294</td>
      <td>1.100000</td>
      <td>4.219450</td>
      <td>94.212627</td>
      <td>NaN</td>
      <td>25.314741</td>
      <td>18.493347</td>
      <td>102.870130</td>
      <td>14.372727</td>
      <td>134.051948</td>
      <td>25.749351</td>
      <td>14.679221</td>
      <td>303.987013</td>
      <td>53.032468</td>
      <td>63.042857</td>
      <td>247.311688</td>
      <td>47.036364</td>
      <td>258.857143</td>
      <td>46.967532</td>
      <td>20112012.0</td>
      <td>67.142857</td>
      <td>13.380519</td>
      <td>562.844156</td>
      <td>72.181818</td>
      <td>11.576623</td>
      <td>5.666667</td>
      <td>11.400000</td>
      <td>16.666667</td>
      <td>31.766667</td>
      <td>18.666667</td>
      <td>26.300000</td>
      <td>59.666667</td>
      <td>56.800000</td>
      <td>41.000000</td>
      <td>30.500000</td>
      <td>682.333333</td>
      <td>82.000000</td>
      <td>2011.0</td>
      <td>78.637339</td>
      <td>82.854077</td>
      <td>37.718884</td>
      <td>521.766234</td>
      <td>35.912017</td>
      <td>215.54721</td>
      <td>8.210730</td>
      <td>7.657082</td>
      <td>7.545708</td>
      <td>7.848712</td>
      <td>7.166738</td>
      <td>6.545494</td>
      <td>NaN</td>
      <td>7.534120</td>
      <td>6.698485</td>
      <td>6.161255</td>
      <td>6.697186</td>
      <td>7.420563</td>
      <td>7.355579</td>
      <td>6.784120</td>
      <td>7.087554</td>
      <td>7.601717</td>
    </tr>
    <tr>
      <th>std</th>
      <td>56.802783</td>
      <td>64.684655</td>
      <td>58.635109</td>
      <td>174.536123</td>
      <td>14.209488</td>
      <td>3529.272774</td>
      <td>19.345374</td>
      <td>21.107520</td>
      <td>1.212249</td>
      <td>1.136455e+09</td>
      <td>1.189750e+06</td>
      <td>15582.415491</td>
      <td>4.367781</td>
      <td>14.321718</td>
      <td>0.783349</td>
      <td>1.121635</td>
      <td>0.421583</td>
      <td>1.164591</td>
      <td>1.564233</td>
      <td>775.870436</td>
      <td>537.927907</td>
      <td>221.714680</td>
      <td>373.153407</td>
      <td>390.482085</td>
      <td>18.809995</td>
      <td>20.401413</td>
      <td>7.375343</td>
      <td>10.885436</td>
      <td>20.669807</td>
      <td>18.765469</td>
      <td>21.497118</td>
      <td>15.739418</td>
      <td>147.702989</td>
      <td>20.403057</td>
      <td>24.180810</td>
      <td>20.669807</td>
      <td>2.444940</td>
      <td>0.316228</td>
      <td>2.652025</td>
      <td>59.142968</td>
      <td>NaN</td>
      <td>3.844544</td>
      <td>2.155014</td>
      <td>281.673353</td>
      <td>18.974405</td>
      <td>133.705086</td>
      <td>12.155025</td>
      <td>23.955448</td>
      <td>284.349049</td>
      <td>13.248917</td>
      <td>19.888468</td>
      <td>221.455429</td>
      <td>18.881946</td>
      <td>266.700685</td>
      <td>13.248917</td>
      <td>0.0</td>
      <td>59.658174</td>
      <td>8.219778</td>
      <td>508.919730</td>
      <td>135.710773</td>
      <td>14.941939</td>
      <td>8.144528</td>
      <td>16.724832</td>
      <td>13.576941</td>
      <td>27.157749</td>
      <td>10.408330</td>
      <td>19.754493</td>
      <td>71.143048</td>
      <td>40.605541</td>
      <td>67.549981</td>
      <td>46.159831</td>
      <td>37.872593</td>
      <td>52.048055</td>
      <td>0.0</td>
      <td>16.752320</td>
      <td>17.309832</td>
      <td>20.409052</td>
      <td>619.422243</td>
      <td>35.912360</td>
      <td>261.22329</td>
      <td>0.645791</td>
      <td>0.566448</td>
      <td>0.500744</td>
      <td>0.561343</td>
      <td>0.942149</td>
      <td>1.209475</td>
      <td>NaN</td>
      <td>0.996734</td>
      <td>0.786807</td>
      <td>0.608109</td>
      <td>0.641051</td>
      <td>0.532572</td>
      <td>0.687727</td>
      <td>0.612842</td>
      <td>0.606183</td>
      <td>0.574772</td>
    </tr>
    <tr>
      <th>min</th>
      <td>279.000000</td>
      <td>312.000000</td>
      <td>286.000000</td>
      <td>887.000000</td>
      <td>1.000000</td>
      <td>10090.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000160e+09</td>
      <td>1.000811e+06</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>12.000000</td>
      <td>6.000000</td>
      <td>9.000000</td>
      <td>6.000000</td>
      <td>1.000000</td>
      <td>50.000000</td>
      <td>10001.000000</td>
      <td>6.000000</td>
      <td>6.000000</td>
      <td>6.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>10.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>16.072727</td>
      <td>1.000000</td>
      <td>1.200000</td>
      <td>26.818182</td>
      <td>NaN</td>
      <td>17.090909</td>
      <td>15.090909</td>
      <td>1.000000</td>
      <td>0.200000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>33.000000</td>
      <td>25.100000</td>
      <td>15.800000</td>
      <td>14.000000</td>
      <td>2.400000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>20112012.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>68.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>0.700000</td>
      <td>7.000000</td>
      <td>14.300000</td>
      <td>9.000000</td>
      <td>18.400000</td>
      <td>2.000000</td>
      <td>3.600000</td>
      <td>650.000000</td>
      <td>49.000000</td>
      <td>2011.0</td>
      <td>0.000000</td>
      <td>14.000000</td>
      <td>2.000000</td>
      <td>39.000000</td>
      <td>2.000000</td>
      <td>4.00000</td>
      <td>5.700000</td>
      <td>5.800000</td>
      <td>5.700000</td>
      <td>6.200000</td>
      <td>3.900000</td>
      <td>1.900000</td>
      <td>NaN</td>
      <td>3.600000</td>
      <td>4.900000</td>
      <td>5.000000</td>
      <td>5.400000</td>
      <td>6.100000</td>
      <td>4.900000</td>
      <td>4.500000</td>
      <td>5.000000</td>
      <td>5.600000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>368.000000</td>
      <td>371.000000</td>
      <td>360.000000</td>
      <td>1102.000000</td>
      <td>17.000000</td>
      <td>11606.000000</td>
      <td>22.000000</td>
      <td>23.000000</td>
      <td>2.000000</td>
      <td>2.023060e+09</td>
      <td>2.000992e+06</td>
      <td>118.000000</td>
      <td>3.000000</td>
      <td>10.000000</td>
      <td>12.000000</td>
      <td>9.000000</td>
      <td>12.000000</td>
      <td>9.000000</td>
      <td>1.000000</td>
      <td>349.000000</td>
      <td>10301.500000</td>
      <td>24.000000</td>
      <td>14.000000</td>
      <td>30.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>4.000000</td>
      <td>5.900000</td>
      <td>10.950000</td>
      <td>25.950000</td>
      <td>49.500000</td>
      <td>12.800000</td>
      <td>74.000000</td>
      <td>49.800000</td>
      <td>31.900000</td>
      <td>61.900000</td>
      <td>21.345312</td>
      <td>1.000000</td>
      <td>2.784375</td>
      <td>59.231250</td>
      <td>NaN</td>
      <td>23.612500</td>
      <td>17.437500</td>
      <td>13.000000</td>
      <td>3.800000</td>
      <td>53.000000</td>
      <td>15.600000</td>
      <td>2.100000</td>
      <td>136.000000</td>
      <td>45.500000</td>
      <td>54.600000</td>
      <td>111.000000</td>
      <td>38.800000</td>
      <td>130.000000</td>
      <td>40.800000</td>
      <td>20112012.0</td>
      <td>19.000000</td>
      <td>6.900000</td>
      <td>256.000000</td>
      <td>9.000000</td>
      <td>2.400000</td>
      <td>1.000000</td>
      <td>1.800000</td>
      <td>12.500000</td>
      <td>22.150000</td>
      <td>14.500000</td>
      <td>14.900000</td>
      <td>19.000000</td>
      <td>35.550000</td>
      <td>2.000000</td>
      <td>3.850000</td>
      <td>661.500000</td>
      <td>52.000000</td>
      <td>2011.0</td>
      <td>71.000000</td>
      <td>73.250000</td>
      <td>22.000000</td>
      <td>228.000000</td>
      <td>18.000000</td>
      <td>78.25000</td>
      <td>7.800000</td>
      <td>7.300000</td>
      <td>7.200000</td>
      <td>7.400000</td>
      <td>6.500000</td>
      <td>5.700000</td>
      <td>NaN</td>
      <td>6.900000</td>
      <td>6.100000</td>
      <td>5.700000</td>
      <td>6.200000</td>
      <td>7.000000</td>
      <td>6.900000</td>
      <td>6.400000</td>
      <td>6.700000</td>
      <td>7.200000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>391.000000</td>
      <td>395.000000</td>
      <td>381.000000</td>
      <td>1169.000000</td>
      <td>32.000000</td>
      <td>13093.000000</td>
      <td>39.000000</td>
      <td>34.000000</td>
      <td>4.000000</td>
      <td>2.053680e+09</td>
      <td>2.116159e+06</td>
      <td>251.500000</td>
      <td>6.000000</td>
      <td>18.000000</td>
      <td>12.000000</td>
      <td>9.000000</td>
      <td>12.000000</td>
      <td>9.000000</td>
      <td>1.000000</td>
      <td>460.500000</td>
      <td>10472.000000</td>
      <td>46.000000</td>
      <td>37.000000</td>
      <td>65.000000</td>
      <td>3.300000</td>
      <td>5.200000</td>
      <td>9.300000</td>
      <td>13.500000</td>
      <td>22.900000</td>
      <td>40.200000</td>
      <td>62.500000</td>
      <td>18.500000</td>
      <td>96.000000</td>
      <td>64.900000</td>
      <td>49.600000</td>
      <td>77.100000</td>
      <td>22.397768</td>
      <td>1.000000</td>
      <td>3.500000</td>
      <td>78.821429</td>
      <td>NaN</td>
      <td>25.660714</td>
      <td>18.593750</td>
      <td>29.000000</td>
      <td>6.300000</td>
      <td>98.000000</td>
      <td>27.700000</td>
      <td>6.100000</td>
      <td>222.000000</td>
      <td>52.000000</td>
      <td>68.300000</td>
      <td>181.000000</td>
      <td>49.700000</td>
      <td>190.000000</td>
      <td>48.000000</td>
      <td>20112012.0</td>
      <td>57.000000</td>
      <td>14.100000</td>
      <td>429.000000</td>
      <td>20.000000</td>
      <td>4.800000</td>
      <td>2.000000</td>
      <td>3.600000</td>
      <td>24.000000</td>
      <td>43.600000</td>
      <td>22.000000</td>
      <td>15.500000</td>
      <td>29.000000</td>
      <td>52.700000</td>
      <td>2.000000</td>
      <td>4.100000</td>
      <td>673.000000</td>
      <td>55.000000</td>
      <td>2011.0</td>
      <td>83.000000</td>
      <td>88.000000</td>
      <td>34.500000</td>
      <td>337.500000</td>
      <td>25.000000</td>
      <td>154.50000</td>
      <td>8.300000</td>
      <td>7.700000</td>
      <td>7.500000</td>
      <td>7.800000</td>
      <td>7.200000</td>
      <td>6.600000</td>
      <td>NaN</td>
      <td>7.550000</td>
      <td>6.600000</td>
      <td>6.000000</td>
      <td>6.600000</td>
      <td>7.400000</td>
      <td>7.300000</td>
      <td>6.700000</td>
      <td>7.000000</td>
      <td>7.600000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>416.000000</td>
      <td>437.000000</td>
      <td>411.000000</td>
      <td>1257.000000</td>
      <td>40.000000</td>
      <td>16865.750000</td>
      <td>53.500000</td>
      <td>54.000000</td>
      <td>5.000000</td>
      <td>3.068830e+09</td>
      <td>3.330710e+06</td>
      <td>510.250000</td>
      <td>10.000000</td>
      <td>34.000000</td>
      <td>12.000000</td>
      <td>9.000000</td>
      <td>12.000000</td>
      <td>9.000000</td>
      <td>2.000000</td>
      <td>622.000000</td>
      <td>11217.000000</td>
      <td>114.000000</td>
      <td>106.000000</td>
      <td>149.000000</td>
      <td>11.500000</td>
      <td>16.950000</td>
      <td>16.100000</td>
      <td>18.450000</td>
      <td>38.100000</td>
      <td>51.650000</td>
      <td>72.500000</td>
      <td>30.050000</td>
      <td>136.750000</td>
      <td>77.700000</td>
      <td>64.500000</td>
      <td>89.050000</td>
      <td>22.969118</td>
      <td>1.000000</td>
      <td>5.334559</td>
      <td>124.760110</td>
      <td>NaN</td>
      <td>27.283482</td>
      <td>19.455263</td>
      <td>102.000000</td>
      <td>16.000000</td>
      <td>164.000000</td>
      <td>34.400000</td>
      <td>10.900000</td>
      <td>354.000000</td>
      <td>59.200000</td>
      <td>75.500000</td>
      <td>277.000000</td>
      <td>59.200000</td>
      <td>278.000000</td>
      <td>54.500000</td>
      <td>20112012.0</td>
      <td>86.000000</td>
      <td>18.800000</td>
      <td>605.000000</td>
      <td>67.000000</td>
      <td>12.600000</td>
      <td>8.500000</td>
      <td>17.100000</td>
      <td>24.500000</td>
      <td>47.300000</td>
      <td>24.500000</td>
      <td>32.300000</td>
      <td>85.000000</td>
      <td>76.000000</td>
      <td>60.500000</td>
      <td>43.950000</td>
      <td>698.500000</td>
      <td>98.500000</td>
      <td>2011.0</td>
      <td>91.000000</td>
      <td>96.000000</td>
      <td>49.000000</td>
      <td>481.750000</td>
      <td>35.000000</td>
      <td>254.75000</td>
      <td>8.600000</td>
      <td>8.000000</td>
      <td>7.875000</td>
      <td>8.200000</td>
      <td>7.800000</td>
      <td>7.400000</td>
      <td>NaN</td>
      <td>8.300000</td>
      <td>7.200000</td>
      <td>6.500000</td>
      <td>7.100000</td>
      <td>7.700000</td>
      <td>7.800000</td>
      <td>7.200000</td>
      <td>7.500000</td>
      <td>8.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>679.000000</td>
      <td>735.000000</td>
      <td>682.000000</td>
      <td>2096.000000</td>
      <td>51.000000</td>
      <td>24669.000000</td>
      <td>71.000000</td>
      <td>77.000000</td>
      <td>5.000000</td>
      <td>5.066130e+09</td>
      <td>5.149609e+06</td>
      <td>138501.000000</td>
      <td>18.000000</td>
      <td>51.000000</td>
      <td>14.000000</td>
      <td>9.000000</td>
      <td>12.000000</td>
      <td>9.000000</td>
      <td>10.000000</td>
      <td>5458.000000</td>
      <td>11694.000000</td>
      <td>2117.000000</td>
      <td>2687.000000</td>
      <td>3692.000000</td>
      <td>97.800000</td>
      <td>99.500000</td>
      <td>26.600000</td>
      <td>76.400000</td>
      <td>100.000000</td>
      <td>96.400000</td>
      <td>100.000000</td>
      <td>74.000000</td>
      <td>1092.000000</td>
      <td>100.000000</td>
      <td>100.000000</td>
      <td>100.000000</td>
      <td>25.510526</td>
      <td>2.000000</td>
      <td>10.250000</td>
      <td>214.166667</td>
      <td>NaN</td>
      <td>31.210526</td>
      <td>22.600000</td>
      <td>2377.000000</td>
      <td>89.500000</td>
      <td>828.000000</td>
      <td>47.900000</td>
      <td>94.900000</td>
      <td>1483.000000</td>
      <td>100.000000</td>
      <td>97.500000</td>
      <td>965.000000</td>
      <td>99.800000</td>
      <td>1956.000000</td>
      <td>74.900000</td>
      <td>20112012.0</td>
      <td>292.000000</td>
      <td>32.900000</td>
      <td>3297.000000</td>
      <td>780.000000</td>
      <td>63.700000</td>
      <td>15.000000</td>
      <td>30.600000</td>
      <td>25.000000</td>
      <td>51.000000</td>
      <td>27.000000</td>
      <td>49.100000</td>
      <td>141.000000</td>
      <td>99.300000</td>
      <td>119.000000</td>
      <td>83.800000</td>
      <td>724.000000</td>
      <td>142.000000</td>
      <td>2011.0</td>
      <td>100.000000</td>
      <td>100.000000</td>
      <td>94.000000</td>
      <td>4768.000000</td>
      <td>244.000000</td>
      <td>2804.00000</td>
      <td>10.000000</td>
      <td>9.600000</td>
      <td>9.700000</td>
      <td>10.000000</td>
      <td>9.600000</td>
      <td>9.000000</td>
      <td>NaN</td>
      <td>9.800000</td>
      <td>9.200000</td>
      <td>9.300000</td>
      <td>9.500000</td>
      <td>9.500000</td>
      <td>9.500000</td>
      <td>8.700000</td>
      <td>9.200000</td>
      <td>9.600000</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Adding a field for school districs
full["school_dist"] = full["dbn"].apply(lambda x: x[:2])
full.fillna(full.mean(),inplace=True)
```

**It would be beneficial to understand the correlation between sat_score and other columns to understand what fields influence sat_score the most**


```python
# Computing correlations between Sat_score and other columns
full.corr()['sat_score'].sort_values(ascending=False)
```




    sat_score                               1.000000e+00
    sat_writing_avg_score                   9.810159e-01
    sat_critical_reading_avg_score          9.747583e-01
    sat_math_avg_score                      9.530106e-01
    ap_test_takers_                         5.103909e-01
    total_exams_taken                       5.017174e-01
    number_of_exams_with_scores_3_4_or_5    4.535005e-01
    N_p                                     4.281443e-01
    advanced_regents_of_cohort              4.253570e-01
    N_s                                     4.221618e-01
    advanced_regents_of_grads               3.924787e-01
    total_regents_of_cohort                 3.873194e-01
    total_students                          3.845674e-01
    white_num                               3.777941e-01
    total_grads_of_cohort                   3.403242e-01
    white_per                               3.328702e-01
    N_t                                     2.940519e-01
    saf_t_11                                2.757786e-01
    rr_s                                    2.736355e-01
    asian_num                               2.729442e-01
    total_regents_of_grads                  2.697793e-01
    aca_s_11                                2.669456e-01
    saf_tot_11                              2.608540e-01
    saf_s_11                                2.569001e-01
    male_num                                2.017470e-01
    total_enrollment                        2.013726e-01
    asian_per                               1.906019e-01
    number_of_sections                      1.726816e-01
    female_num                              1.711856e-01
    number_of_students_seats_filled         1.708587e-01
    total_cohort                            1.619401e-01
    aca_tot_11                              1.611355e-01
    eng_s_11                                1.551608e-01
    com_s_11                                1.512602e-01
    aca_t_11                                1.216712e-01
    number_programs                         1.153913e-01
    number_tested                           1.109474e-01
    level_4_1                               1.108240e-01
    level_4_2                               1.107934e-01
    level_3_4_1                             1.104739e-01
    mean_scale_score                        1.072120e-01
    saf_p_11                                1.066251e-01
    level_3_4_2                             1.027085e-01
    rr_p                                    1.021401e-01
    eng_tot_11                              8.574072e-02
    com_t_11                                8.197490e-02
    com_tot_11                              7.766472e-02
    regents_w_o_advanced_of_cohort          7.265046e-02
    female_per                              6.516539e-02
    size_of_largest_class                   4.760754e-02
    bin                                     4.132848e-02
    census_tract                            3.914908e-02
    level_3_1                               3.586515e-02
    average_class_size                      3.438492e-02
    bbl                                     3.429191e-02
    aca_p_11                                3.004570e-02
    eng_p_11                                2.891362e-02
    :@computed_region_sbqj_enih             1.987861e-02
    grade_span_max                          1.830755e-02
    :@computed_region_efsh_h5xi             1.638056e-02
    rr_t                                    1.164827e-02
    expgrade_span_max                       7.847064e-04
    expgrade_span_min                       4.104648e-27
    size_of_smallest_class                 -2.323112e-02
    grade_span_min                         -2.844718e-02
    csd                                    -3.282872e-02
    :@computed_region_f5dn_yrer            -4.702550e-02
    level_3_2                              -4.776999e-02
    community_board                        -5.675188e-02
    black_num                              -6.104478e-02
    zip                                    -6.147304e-02
    male_per                               -6.516539e-02
    :@computed_region_yeji_bk3q            -6.561082e-02
    level_1_2                              -6.972787e-02
    level_1_1                              -7.103664e-02
    :@computed_region_92fq_4b7q            -7.145365e-02
    council_district                       -7.446446e-02
    com_p_11                               -8.715157e-02
    hispanic_num                           -8.974585e-02
    sped_num                               -9.696171e-02
    level_2_2                              -1.105178e-01
    level_2_1                              -1.109453e-01
    regents_w_o_advanced_of_grads          -1.130022e-01
    Latitude                               -1.155862e-01
    Longitude                              -1.288842e-01
    ell_percent                            -1.299355e-01
    local_of_cohort                        -2.226132e-01
    black_per                              -2.295139e-01
    sped_percent                           -2.306069e-01
    still_enrolled_of_cohort               -2.361166e-01
    local_of_grads                         -2.697793e-01
    hispanic_per                           -3.222427e-01
    dropped_out_of_cohort                  -3.397260e-01
    frl_percent                            -3.546909e-01
    schoolwide_pupil_teacher_ratio                   NaN
    schoolyear                                       NaN
    year                                             NaN
    eng_t_10                                         NaN
    Name: sat_score, dtype: float64



Let's take a moment here to note down anything interesting we can find from the above. We can use that later to create an
analysis viewpoint.
 * Total number of students correlated with Sat Scores. Interesting, that would mean than large schools
    perform better than small schools
 * Number of parents, teachers, and students responding to the survey also highly correlates with sat scores
 * FRL (Free or Reduced Lunches) and ELL (English Language Learners) percent correlate strongly negatively with sat score
 * Female_num correlates positively whereas male_num correlates negatively with Sat scores
 * Racial inequality in sat scores can be identified easily from above (white_per, hispanic_per, black_per, asian_per)

## Plotting stage

Now that we have prepared our data, drawn some insights from it as well. Let's now do some exploratory charts and maps
for a better understanding of the problem at hand


```python
# Converting Latitude and Longitude to float values
full['Latitude'] = pd.to_numeric(full['Latitude'],errors='coerce')
full['Longitude'] = pd.to_numeric(full['Longitude'],errors='coerce')
```


```python
schools_map = folium.Map(location=[full['Latitude'].mean(), full['Longitude'].mean()], zoom_start=10)
marker_cluster = plugins.MarkerCluster().add_to(schools_map)
for name, row in full.loc[~full['Latitude'].isnull()].iterrows():
    folium.Marker([row["Latitude"], row["Longitude"]], popup="{0}: {1}".format(row["dbn"], row["school_name"])).add_to(marker_cluster)
schools_map.save('schools.html')
schools_map
```

<div class="map-container">
    <iframe src="/assets/img/schools.html" height="600" width="800" frameborder="0">
    </iframe>
</div>