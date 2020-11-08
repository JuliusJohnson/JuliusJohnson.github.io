---
title: How I Scrapped Data From Metacritic for Analysis
date: 2020-10-31 12:00:00 -05:00
tags: [python, scrapping, pandas, crawling, beautifulsoup]
description: Breif overview of the code I used to scrape Metacritic for my review bombing analysis
---

## How I Scrapped Data From Metacritic for Analysis

My goal is to discover if video game review bombing is real, using data. I decided to use one of the internet’s biggest review aggregators, MetaCritic, to get the data myself. I found older datasets on the internet, but in many cases, they lacked the exact dimensions I wanted to analyze and were dated. 


### [Web Scraper](https://github.com/JuliusJohnson/Metacritic_Game_Scrapper_2020/blob/master/mScrapper.py)(GitHub)

**Initial Setup:**

The Metacritic game’s SRP(Search Result Page) is pretty typical. It can return a paginated list of games filtered by year. The function “webpage” allows me to control the year and “numberPages” determines the number of pages a search result has, allowing me to control how many times to loop the web scraper.


```
data_dict = {'name':[], 'date':[], 'platform': [], 'score':[], 'url':[], 'userscore':[]} # Data Structure

def webpage(pageNum,year): #function that navigates the metacritic SRP(Search Results Pages) based on the page number and the year
   url = 'https://www.metacritic.com/browse/games/score/metascore/year/all/filtered?year_selected='+ str(year) +'&distribution=&sort=desc&view=condensed&page='+ str(pageNum)
   userAgent = {'User-agent': 'Mozilla/5.0'}
   response = requests.get(url, headers=userAgent)
   return response

def numberPages(response): # Helper Function that determines how many pages are in a SRP to know how many times to run scrapper function
   soup = BeautifulSoup(response.text, 'html.parser')
   pages = soup.find_all('li', {"class":"page last_page"})
   pagesCleaned = pages[0].find('a', {"class":"page_num"})
   return (pagesCleaned.text)
```


I created a function to scrape each item in the dictionary (Video Game Name, Release Date, Platform, Critic Score, URL, and userscore. Please feel free to review the [full code for the scrapper on GitHub](https://github.com/JuliusJohnson/Metacritic_Game_Scrapper_2020/blob/master/mScrapper.py).

**Scrapping Execution:**

I scraped data from 2015 to the most recent video game release at the time of scrape. I was careful not to be IP blocked and used a 5-second sleep timer between each year. 


```
def main():
   years = list(range(2015,2021))
   for year in years:
       numPage = (numberPages(webpage(0,year)))
       pages(int(numPage),year)
       time.sleep(5)
       print(year)
   xData = (pd.DataFrame.from_dict(data_dict))
   xData.to_csv('mc.csv')
```



### [Link Crawler:(GitHub)](https://github.com/JuliusJohnson/Metacritic_Game_Scrapper_2020/blob/master/crawler.py)

Because I wanted to modularize the script for this project (partly due to concerns around execution time), I created a separate web crawler that crawls through each of the game URLs and return the developer, number of user ratings, and genre.  

This script takes about 5.5 hours to run. After I scrape the genre, developer, and the number of user reviews from a link, I pause 3.25 seconds before moving on to the next link. Once again, I was careful in not getting IP blocked or throttled.

_Note: “getGenreDev” is the function that contains code for the scrapping element of the link crawler_


```
data['developer'] = ""
data['genre'] = ""
data['no_userreviews'] = ""
index = 0
while index < data.shape[0]:
   print(index) #for troubleshooting
   try:
       data['developer'].loc[index] = (getGenreDev(data['fullurl'][index])[0]) #looks at the row represented by (loc[index]) and sets that equal to the first output of the "getGenreDev" function
       data['genre'].loc[index] = (getGenreDev(data['fullurl'][index])[1])
       data['no_userreviews'].loc[index] = (getGenreDev(data['fullurl'][index])[2])
   except:
       data['developer'].loc[index] = "No Data"
       data['genre'].loc[index] = "No Data"
       data['no_userreviews'].loc[index] = "No Data"
   index += 1
   time.sleep(3.25)   
data.to_csv('metacritic_output.csv')
```



### [Cleaning the Data(GitHub)](https://github.com/JuliusJohnson/Metacritic_Game_Scrapper_2020/blob/master/Analysis/metacritic_cleaning_exploration.ipynb)

After I successfully got all the fields I wanted to analyze, I needed to clean the data. I created a Jupyter Notebook, and with the aid of Pandas Profiling, I made a usable CSV file for analysis.

How I augmented the data can be found in the code below:

Most of the cleaning was creating consistent data types within dimensions and metrics.


<div style="text-align:center"><img src="/assets/img/metacritic-code-dtypes.png" /></div>