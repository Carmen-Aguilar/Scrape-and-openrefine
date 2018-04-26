# How I scraped several websites and cleaned the results in OpenRefine

For a story about the olive oil industry, I wanted to know the products sold in the UK by brand and price. There is no “free/public” list, so I made one. 

I scraped that information from the main retailers using python. I mainly used quickcode, but I repeated the process using Jupyter notebook. 

As an example, here is the code that I used on one website. 

```
#!/usr/bin/env python

import scraperwiki
import requests
from lxml import html

web = requests.get("https://groceries.morrisons.com/webshop/getSearchProducts.do?clearTabs=yes&isFreshSearch=true&chosenSuggestionPosition=0&entry=olive+oil", verify=False)
root = html.fromstring(web.content)
print root

rows = root.cssselect("div.fop-content-wrapper")
print rows
dataset = {}
index = 0
for row in rows:
    index = index+1
    dataset['index'] = index
    Titles = row.cssselect("h4")
    print 'scraping', Titles
    dataset['Titles'] = Titles[0].text
    Prices = row.cssselect("span.fop-unit-price")
    dataset['Prices'] = Prices[0].text
    print 'scraping2', Prices
    Links = row.cssselect("a")
    dataset['Links'] = Links[0].attrib['href']
    scraperwiki.sql.save(['index'], dataset)
    
```
 
And <a href="https://github.com/Carmen-Aguilar/Scrape-and-openrefine/blob/master/Webscraping.ipynb">here is the step by step using Jupyter Notebook</a>. In this case, I did a loop in the url as well. 

## Moving to OpenRefine

I combined the six files where I had been storing the information (one for each retailer) in Open Refine, and I made sure to click on the option “Worksheet to import” to generate a new column with the name of each document. I also unticked the option “Storing blank rows”. 

My files were called “name of the retailer.csv”, so I used this column to get rid of the “.csv” and keep only the name. I split the column (Edit column/Split into several columns) by the dot and deleted the column with only “csv” values. I guess I could have also done it by replacing “.csv” with nothing. 

![screenshot1](/screenshot1.png)

I realised that I wasn’t really accurate with my scraping, and I didn’t store the information using the same names for the columns in all the six files. I had, then, several columns for the same information. One called ‘Title’, another ‘title’, another ‘titles’. 

So, besides making a mental note for next scraping, I combined all of them into one single column called “product_name”. How? Click on “Edit column/Add column based on this column” and using the GREL expression. 

In my case the expression was:
```
if(isBlank(cells["Names"].value), " ", cells["Names"].value) + if(isBlank(cells["Titles"].value), " ", cells["Titles"].value) + if(isBlank(cells["titles"].value), " ", cells["titles"].value) + if(isBlank(cells["name"].value), " ", cells["name"].value)
```
The <a href="https://groups.google.com/forum/#!topic/openrefine/O6QF_KKaMw0">conversation in this group</a> helped me.
 
After proving there were no blanks rows, I repeated the process with the columns of the prices. I realised that you can use the same column with “Edit cells”, instead of creating a new one. 

## Cleaning

When scraping some pages, I got more products which were not related to the products I was interested in. For example, I got sardines in olive oil when I was looking for (real) olive oil. 

I cleaned it in a kind of manually way. I select “Text filter” and looked for the word “oil”. In the blue box, I clicked on “invert”, and I got the products without that word. I deleted them. But I had still more products which weren’t olive oil. I repeated the process with the word ‘olive’.

![screenshot2](/screenshot2.png)

Filtering again by words like ‘butter’, ‘anchovy’, ‘fillet’, ‘hair’, ‘snack’…. and up to 20 options, and without inverting this time, I got my final list. 

I also completed part of the url that I had grabbed. In some cases, I didn’t store the “www.nameoftheretailer.co.uk”, but just everything after that. In “Edit cells/Transform” and using GREL, I pasted the first part between quotation marks, followed by “+” and value. 

 ![screenshot3](/screenshot3.png)

But, before pasting, clean all the white spaces you can have in the cells. Otherwise, you will have “firstpartofurl”space”secondparturl” and that does not work. I proved it. 

## If/or…

My next step was adding a new column with the country of origin of each brand. In other words, it was telling to OpenRefine “if you see this OR this OR this…. Write this word”. 

Sounds easy, but it took me a while to complete the formula. Here is just part of it:
```
if(or(value.contains("Waitrose"),value.contains("ASDA"),value.contains("Tesco"),value.contains("Sainsbury"),value.contains("Morrisons”),value.contains(“Carapelli”),value.contains(“Farchioni”),value.contains(“Casolare”),value.contains(“Napolina”)),”Own brand", if(or(value.contains("Bertolli"),value.contains("Filippo")), "Italian", null))
```

And don’t copy and paste directly to OpenRefine. I, first, wrote it in a Notepad, but the code didn’t work when I pasted in OpenRefine. It took me another while to see that the quotation marks were different in the Notepad to the one which OpenRefine reads.

## Changing GREL for Jython

Finally, I cleaned the price column. I had several types of prices:
£number/100ml
Numberp/100 ml
Weirdcharacter£number/1lt
…

And all the possible combinations. 

The first step was splitting the column by “/”. The second, deleting the “p” character with GREL expression: replace(value, “p”, “”). And then, I changed the values to numbers. 

I didn’t do the same with the £ symbol, because I had to convert pence into pounds and I needed something to differentiate them.

So, before getting read of the pound symbol, I changed pence to pounds. “Easy,” said one of my colleagues who was waiting for me to go to the pub. 

But he left without me. 

“Value/100” didn’t work. Nor even putting brackets or other characters. Happily I found <a href="https://github.com/OpenRefine/OpenRefine/issues/350">this conversation</a> where they pointed out changing from GREL to Jython. 

 ![screenshot4](/screenshot4.png)

It worked and I went to the pub. 
