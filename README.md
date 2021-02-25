# SCRAPY

`CMD + SHIFT + P` then select `python:select interpreter` select `virtual_workspace`

### Create New Project

Activate ENV

```
scrapy startproject worldometer
cd worldometer
scrapy genspider countries www.worldometers.info/world-population/population-by-country
```

When pasting url into the terminal, make sure that you delete the `http://` and the `/`

```python
# -*- coding: utf-8 -*-
import scrapy


class CountriesSpider(scrapy.Spider):
    name = 'countries'
    #BEFORE
    allowed_domains = ['www.worldometers.info/world-population/population-by-country']
    start_urls = ['http://www.worldometers.info/world-population/population-by-country/']
    #AFTER remove after domain name and add a 's' to http
    allowed_domains = ['www.worldometers.info/']
    start_urls = ['https://www.worldometers.info/world-population/population-by-country/']

    def parse(self, response):
        pass
```

install ipython

```
conda install ipython
```

### Open Scrapy Shell & Text Data GET

- Open Shell

```
scrapy shell
```

Do the following `In` commands and make sure you get the correct responses

```s
In [1]: fetch("https://www.worldometers.info/world-population/population-by-country/")
2020-10-16 13:27:53 [scrapy.core.engine] INFO: Spider opened
2020-10-16 13:27:54 [scrapy.core.engine] DEBUG: Crawled (404) <GET https://www.worldometers.info/robots.txt> (referer: None)
2020-10-16 13:27:54 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.worldometers.info/world-population/population-by-country/> (referer: None)
In [2]: r = scrapy.Request(url="https://www.worldometers.info/world-population/population-by-country/")


In [3]: fetch(r)

2020-10-16 13:33:27 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.worldometers.info/world-population/population-by-country/> (referer: None)
In [4]: response.body

In [6]: view(response)
Out[6]: True

In [7]: title=response.xpath("//h1")

In [8]: title
Out[8]: [<Selector xpath='//h1' data='<h1>Countries in the world by populat...'>]

In [10]: title=response.xpath("//h1/text()")

In [11]: title
Out[11]: [<Selector xpath='//h1/text()' data='Countries in the world by population ...'>]

In [12]: title.get()
Out[12]: 'Countries in the world by population (2020)'

In [13]: title_css = response.css("h1::text")

In [14]: title_css
Out[14]: [<Selector xpath='descendant-or-self::h1/text()' data='Countries in the world by population ...'>]

In [15]: title_css.get()
Out[15]: 'Countries in the world by population (2020)'

In [16]: countries = response.xpath("//td/a/text()").getall()

In [17]: countries
Out[17]:
['China',
 'India',
 'United States',
 'Indonesia',
...
]

In [18]: countries_css = response.css("td a::text").getall()

In [19]: countries_css
Out[19]:
['China',
 'India',
 'United States',
 'Indonesia',
 ...
]
```

We are now getting the data, we can now modify the `countries.py` file

```python
# -*- coding: utf-8 -*-
import scrapy


class CountriesSpider(scrapy.Spider):
    name = 'countries'
    allowed_domains = ['www.worldometers.info/']
    start_urls = [
        'https://www.worldometers.info/world-population/population-by-country/']

    def parse(self, response):
        title = response.xpath("//h1/text()").get()
        countries = response.xpath("//td/a/text()").getall()

        yield{
            'title': title,
            'countries': countries
        }

```

Now we can run the file to make a spider. At the same level as `scrapy.cfg` file run where countries is the name of the genspider we made earlier:

```
scrapy crawl countries
```

In the terminal, you should see a return object that includes `title` and a list called `countries`

### X Path

- Get all <div> tags

```
//div
```

- Get div tag with class name 'intro'

```
//div[@class='intro']
```

- Get p elements inside div with class name 'into'

```
//div[@class='intro']/p
```

- Get p elements inside div with class name 'into' AND 'outro'

```
//div[@class='intro' or @class='outro']/p
```

- Get TEXT inside p elements inside div with class name 'into' AND 'outro'

```
//div[@class='intro' or @class='outro']/p/text()
```

- get href attributes

```
//a/@href
```

- get href attributes that start with http and not https

```
//a[starts-with(@href, 'https')]
```

- get href attributes that start with fr

```
//a[ends-with(@href, 'fr')]
```

- get href attribute that contains word google

```
//a[contains(@href, 'google')]
```

- get text that contains the word france (inside html tags)

```
//a[contains(text(), 'France')]
```

- get based on first position

```
ul//[@id='items']/li[1]
```

- get based on first four elements

```
ul//[@id='items']/li[1 or 4]
```

- get based on position 1-4

```
ul//[@id='items']/li[position() = 1 or position() = 4]
```

- get based on first and last element

```
ul//[@id='items']/li[position() = 1 or position() = last()]
```

Get parent

```
//p[@id='unique']/parent::node()
```

Get all ancestors

```
//p[@id='unique']/ancestor::node()
```

# Get Stuff

```
scrapy shell "https://www.worldometers.info/world-population/population-by-country/"
countries = response.xpath("//td/a")
```

We will use this output in our `counties.py`

```python
# -*- coding: utf-8 -*-
import scrapy


class CountriesSpider(scrapy.Spider):
    name = 'countries'
    allowed_domains = ['worldometers.info/']
    start_urls = [
        'https://www.worldometers.info/world-population/population-by-country/']

    def parse(self, response):
        countries = response.xpath("//td/a")

        for country in countries:
            name = country.xpath(".//text()").get()
            link = country.xpath(".//@href").get()

            yield {
                'country_name': name,
                'country_link': link
            }

```

Get A Reponse From Each URL inside <a>

```python
# -*- coding: utf-8 -*-
import scrapy


class CountriesSpider(scrapy.Spider):
    name = 'countries'
    allowed_domains = ['worldometers.info']
    start_urls = [
        'https://www.worldometers.info/world-population/population-by-country/']

    def parse(self, response):
        countries = response.xpath("//td/a")
        for country in countries:
            name = country.xpath(".//text()").get()
            link = country.xpath(".//@href").get()

            # absolute_url = f"https://www.worldometers.info{link}"
            # absolute_url = response.urljoin(link)

            # yield scrapy.Request(url=absolute_url)
            yield response.follow(url=link)

```

run `scrapy crawl countries`

- Now we want to scrap each response we get from the response we just made.
  - In other words, we want to get ALL the href links on the main page, go into those links then get more data from each of those individual pages.

```python
# -*- coding: utf-8 -*-
import scrapy
import logging


class CountriesSpider(scrapy.Spider):
    name = 'countries'
    allowed_domains = ['worldometers.info']
    start_urls = [
        'https://www.worldometers.info/world-population/population-by-country/']

    def parse(self, response):
        countries = response.xpath("//td/a")
        for country in countries:
            name = country.xpath(".//text()").get()
            link = country.xpath(".//@href").get()

            yield response.follow(url=link, callback=self.parse_country)

    def parse_country(self, response):
        rows = response.xpath(
            "(//table[@class='table table-striped table-bordered table-hover table-condensed table-list'])[1]/tbody/tr")
        for row in rows:
            year = row.xpath(".//td[1]/text()").get()
            population = row.xpath(".//td[2]/strong/text()").get()

            yield {
                'year': year,
                'population': population
            }

```

Now we have the year and population in those years. Now we need the name of those rows
Notice below that there is no name in the yield response

```
{'year': '1970', 'population': '55,982,144'}
2020-10-19 11:43:21 [scrapy.core.scraper] DEBUG: Scraped from <200 https://www.worldometers.info/world-population/pakistan-population/>
```

```py
# -*- coding: utf-8 -*-
import scrapy
import logging


class CountriesSpider(scrapy.Spider):
    name = 'countries'
    allowed_domains = ['worldometers.info']
    start_urls = [
        'https://www.worldometers.info/world-population/population-by-country/']

    def parse(self, response):
        countries = response.xpath("//td/a")
        for country in countries:
            name = country.xpath(".//text()").get()
            link = country.xpath(".//@href").get()

            yield response.follow(url=link, callback=self.parse_country, meta={'country_name': name})

    def parse_country(self, response):
        name = response.request.meta['country_name']
        rows = response.xpath(
            "(//table[@class='table table-striped table-bordered table-hover table-condensed table-list'])[1]/tbody/tr")
        for row in rows:
            year = row.xpath(".//td[1]/text()").get()
            population = row.xpath(".//td[2]/strong/text()").get()

            yield {
                'country_name': name,
                'year': year,
                'population': population
            }
```

```
2020-10-19 11:55:09 [scrapy.core.scraper] DEBUG: Scraped from <200 https://www.worldometers.info/world-population/philippines-population/>
{'country_name': 'Philippines', 'year': '2005', 'population': '86,326,250'}
```

## Make the DataSet

- JSON: ` scrapy crawl countries -o population_dataset.json`

- CSV: `scrapy crawl countries -o population_dataset.csv`

- XML: `scrapy crawl countries -o population_dataset.xml`

https://www.cigabuy.com/consumer-electronics-c-56_75-pg-1.html

consumer-electronics-c-56_75-pg-1


## Pagination
The act of going to a different web page (like clicking next in a list) and receiving data from other pages

 - This is from a different project

First, we received the data that wished to get at the initial endpoint, now we cant to inspect the <b>NEXT or 2 ect</b> button that would allow the webpage to paginate. 
