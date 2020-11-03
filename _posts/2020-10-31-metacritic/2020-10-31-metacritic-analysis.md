---
title: Metacritic Review Bombing - Correlation Analysis
date: 2020-11-02 12:00:00 -05:00
tags: [python, Google Sheets, pandas, analysis]
description: Breif overview of the code I used to scrape Metacritic for my review bombing analysis
---

# Metacritic Video Game Review Analysis

I was searching for interesting data sets to explore and expand my skill set. My interest in video games led me to Metacritic and created my own dataset([Read how I created and cleaned my data](https://www.juliusj.com/metacritic-data-scrape/)). I scraped data from January 2015 to September 2020. Specifically, I wanted to explore if review bombing is a real phenomenon not isolated to isolated and controversial game releases.

"A **review bomb** is an Internet phenomenon in which large groups of people leave negative user reviews online for a published work, most commonly a video game or a theatrical film, in an attempt to harm the sales or popularity of a product, particularly to draw attention to an issue with the product or its vendor." - Wikipedia 

Data Exploration:

 

I utilized [Pandas Profiling](https://github.com/pandas-profiling/pandas-profiling) to assist with the exploratory phase of my analysis. 

Some Highlights: ([View the full HTML output](https://github.com/JuliusJohnson/Metacritic_Game_Scrapper_2020/blob/master/Analysis/mc_report.html))



*   Playstation 4(PS4) had the most reviews of any platform between 2015 and September 2020
*   The average critic score is ~72/100; the median is 74/100
*   The average user score is ~66/100; the median is 70/100 
*   The number of reviews ranged from a minimum of 4 to a maximum of 9248
*   There is a moderate correlation between userscore and critic score(score)

<div style="text-align:center"><img src="/assets/img/mc-correlations.png" /></div>

We see that there is some variance between userscore and critic score. When split look at correlations by year correlations fall between ~.40(2020) and ~.56(2015). I looked to see if there was any correlation for months and developers but no clear pattern emerged. 

Some insights into review bombing overtly appear when stack ranking the deltas of critic score and user score. From the first 10 rows a strong pattern emerge, sports games:


<table>
  <tr>
   <td><strong>name</strong>
   </td>
   <td><strong>genre</strong>
   </td>
   <td><strong>Delta</strong>
   </td>
  </tr>
  <tr>
   <td>Atomicrops
   </td>
   <td>['Action', 'Shooter', "Shoot-'Em-Up", 'Top-Down']
   </td>
   <td><p style="text-align: right">
78</p>

   </td>
  </tr>
  <tr>
   <td>Tom Clancy's The Division 2: Warlords of New York
   </td>
   <td>['Action', 'Shooter', 'Third-Person', 'Tactical']
   </td>
   <td><p style="text-align: right">
74</p>

   </td>
  </tr>
  <tr>
   <td>Tom Clancy's The Division 2: Warlords of New York
   </td>
   <td>['Action', 'Shooter', 'Third-Person', 'Tactical']
   </td>
   <td><p style="text-align: right">
71</p>

   </td>
  </tr>
  <tr>
   <td>NBA 2K18
   </td>
   <td>['Sports', 'Team', 'Basketball', 'Sim']
   </td>
   <td><p style="text-align: right">
69</p>

   </td>
  </tr>
  <tr>
   <td>FIFA 20
   </td>
   <td>['Sports', 'Team', 'Soccer', 'Sim']
   </td>
   <td><p style="text-align: right">
67</p>

   </td>
  </tr>
  <tr>
   <td>NBA 2K20
   </td>
   <td>['Sports', 'Team', 'Basketball', 'Sim']
   </td>
   <td><p style="text-align: right">
66</p>

   </td>
  </tr>
  <tr>
   <td>FIFA 20
   </td>
   <td>['Sports', 'Team', 'Soccer', 'Sim']
   </td>
   <td><p style="text-align: right">
66</p>

   </td>
  </tr>
  <tr>
   <td>EA SPORTS UFC 4
   </td>
   <td>['Sports', 'Individual', 'Combat', 'Boxing / Martial Arts']
   </td>
   <td><p style="text-align: right">
66</p>

   </td>
  </tr>
  <tr>
   <td>NBA 2K20
   </td>
   <td>['Sports', 'Team', 'Basketball', 'Sim']
   </td>
   <td><p style="text-align: right">
65</p>

   </td>
  </tr>
  <tr>
   <td>Madden NFL 21
   </td>
   <td>['Sports', 'Team', 'Football', 'Sim']
   </td>
   <td><p style="text-align: right">
65</p>

   </td>
  </tr>
</table>


While NLP is out of the scope of this analysis, anecdotally, I saw a similar pattern of reviews of sports games they were overall too iterative. In this case that means only making the minimum yearly roster changes and other changes players ultimately found insignificant. 

**Conclusion:** Review Bombing happens quite frequently if you are a sports game. Between 2015 and September 2020 there were a total of 255(4.51%) games whose score delta was beyond two standard deviations(upper). Distribution of Review Deltas:


<div style="text-align:center"><img src="/assets/img/mc-distribution.png" /></div>

