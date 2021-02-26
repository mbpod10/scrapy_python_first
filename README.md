# SCRAPY BASIC TEMPLATE

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


![image of it](https://i.imgur.com/qHy6Pnn.png "pagination")

Once we have the link to the href, in this case it is `response.xpath("//a[@class='page-link']/@href").get()`, we are going to set it to a variable and then create an if statement that says, that exists, then parse the page once again like this:

```py
next_page = response.xpath("//a[@class='page-link']/@href").get()
        if next_page:
            yield scrapy.Request(url=next_page, callback=self.parse)
```

so the whole spider file would look like:

```py
class GlassesDealsSpider(scrapy.Spider):
    name = 'glasses_deals'
    allowed_domains = ['www.glassesshop.com']
    start_urls = ['http://www.glassesshop.com/bestsellers/']

    def parse(self, response):
        rows = response.xpath(
            "//div[@class='col-12 pb-5 mb-lg-3 col-lg-4 product-list-row text-center product-list-item']")

        for row in rows:
            product_image = row.xpath(".//div[2]/a/img/@data-src[1]").get()
            product_link = row.xpath(
                ".//div[@class='p-title-block']/div/div/div/div[@class='p-title']/a/@href").get()
            product_name = ' '.join(row.xpath(
                ".//div[@class='p-title-block']/div/div/div/div[@class='p-title']/a/text()").get().split())
            product_price = row.xpath(
                ".//div[@class='p-title-block']/div[@class='mt-3']/div/div[2]/div/div/span/text()").get()

            yield {
                'product_image': product_image,
                'product_link': product_link,
                'product_name': product_name,
                'product_price': product_price

            }

        next_page = response.xpath("//a[@class='page-link']/@href").get()
        if next_page:
            yield scrapy.Request(url=next_page, callback=self.parse)

```

## User Agent

type into google `My User Agent`, this will give you an agent to your machine. Some website don't allow bots, so this user agent will be sent with request headers which will spoof the website and make it seem as though an actual person is accessing the webpage. 

My User Agent = `Mozilla/5.0 (Macintosh; Intel Mac OS X 11_0_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.192 Safari/537.36`

We will create a function called `start_requests` which will include our start_url as well as our user-agent. This function has to be called `start_requests`

```py
def start_requests(self):
        yield scrapy.Request(url='http://www.glassesshop.com/bestsellers/',
                             callback=self.parse,
                             headers={'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 11_0_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.192 Safari/537.36'})
```

we need to include this within our pagination as well like this:

```py
next_page = response.xpath("//a[@class='page-link']/@href").get()
        if next_page:
            yield scrapy.Request(url=next_page, callback=self.parse,
                                 headers={'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 11_0_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.192 Safari/537.36'})
```

The whole file will look like this:
```py
class GlassesDealsSpider(scrapy.Spider):
    name = 'glasses_deals'
    allowed_domains = ['www.glassesshop.com']   

    def start_requests(self):
        yield scrapy.Request(url='http://www.glassesshop.com/bestsellers/',
                             callback=self.parse,
                             headers={'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 11_0_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.192 Safari/537.36'})

    def parse(self, response):
        rows = response.xpath(
            "//div[@class='col-12 pb-5 mb-lg-3 col-lg-4 product-list-row text-center product-list-item']")

        for row in rows:
            product_image = row.xpath(".//div[2]/a/img/@data-src[1]").get()
            product_link = row.xpath(
                ".//div[@class='p-title-block']/div/div/div/div[@class='p-title']/a/@href").get()
            product_name = ' '.join(row.xpath(
                ".//div[@class='p-title-block']/div/div/div/div[@class='p-title']/a/text()").get().split())
            product_price = row.xpath(
                ".//div[@class='p-title-block']/div[@class='mt-3']/div/div[2]/div/div/span/text()").get()

            yield {
                'product_image': product_image,
                'product_link': product_link,
                'product_name': product_name,
                'product_price': product_price
            }

        next_page = response.xpath("//a[@class='page-link']/@href").get()
        if next_page:
            yield scrapy.Request(url=next_page, callback=self.parse,
                                 headers={'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 11_0_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.192 Safari/537.36'})

```

Execute in the console with `scrapy crawl __file_name__`

# SCRAPY CRAWL SPIDER TEMPLATE

COMMAND LINE:

```
conda activate virtual_workspace
scrapy startproject imdb
cd imdb
scrapy genspider -t crawl best_movies imbd.com
```

```py
import scrapy
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule


class BestMoviesSpider(CrawlSpider):
    name = 'best_movies'
    allowed_domains = ['imdb.com']

    user_agent = 'Mozilla/5.0 (Macintosh; Intel Mac OS X 11_0_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.192 Safari/537.36'

    def start_requests(self):
        yield scrapy.Request(url='https://www.imdb.com/chart/top/', headers={"User-Agent": self.user_agent})

    rules = (
        Rule(LinkExtractor(
          # Think of the restrict_xpath as the for loop we used in the previous project but use the //a to go to the different pages
            restrict_xpaths="//td[@class='titleColumn']/a"), callback='parse_item', follow=True, process_request='set_user_agent'),
    )

    def set_user_agent(self, request):
        request.headers['User-Agent'] = self.user_agent
        return request

    def parse_item(self, response):
        yield {
            'title': response.xpath("//div[@class='title_wrapper']/h1/text()").get(),
            'year': response.xpath("//span[@id='titleYear']/a/text()").get(),
            'duration': response.xpath("normalize-space(//time[1]/text())").get(),
            'rating': response.xpath("//div[@class='ratingValue']/strong/span/text()").get(),
            'movie_url': response.url,
            'User-Agent': response.request.headers['User-Agent']
        }

```

## PAGINATION

to paginate, we need to add another rule to the rules tuple, simply find the next link and add the xpath to the rules
```py
rules = (
        Rule(LinkExtractor(
            restrict_xpaths="//article[@class='product_pod']/h3/a"), callback='parse_item', follow=True),
        Rule(LinkExtractor(
             restrict_xpaths="//li[@class='next']/a"
             ))
    )
```

### FIX UTF ENCODING
in `settings.py` add `FEED_EXPORT_ENCODING = 'utf-8'`

### FIX SPACES IN OUTPUTS
add a normalize-space
```py
'duration': response.xpath("normalize-space(//time[1]/text())").get(),
```
### FINAL OUTPUT

```py
# -*- coding: utf-8 -*-
import scrapy
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule


class BookDealsSpider(CrawlSpider):
    name = 'book_deals'
    allowed_domains = ['books.toscrape.com']
    # start_urls = ['https://books.toscrape.com/']

    user_agent = 'Mozilla/5.0 (Macintosh; Intel Mac OS X 11_0_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.192 Safari/537.36'

    def start_requests(self):
        yield scrapy.Request(url='https://books.toscrape.com/', headers={"User-Agent": self.user_agent})

    rules = (
        Rule(LinkExtractor(
            restrict_xpaths="//article[@class='product_pod']/h3/a"), callback='parse_item', follow=True, process_request='set_user_agent'),
        Rule(LinkExtractor(
             restrict_xpaths="//li[@class='next']/a"), follow=True, process_request='set_user_agent')
    )

    def set_user_agent(self, request):
        request.headers['User-Agent'] = self.user_agent
        return request

    def parse_item(self, response):
        book_name = response.xpath("//h1[1]/text()").get()
        price = response.xpath("//p[@class='price_color']/text()").get()

        yield {
            'book_name': book_name,
            'price': price,
        }

```

## PIPELINES

In the scrapy project folder, go to `pipelines.py`

```py
# -*- coding: utf-8 -*-

# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://doc.scrapy.org/en/latest/topics/item-pipeline.html

import logging
class BooksPipeline(object):

    def open_spider(self, spider):
        logging.warning("SPIDER OPENED FROM PIPELINE")

    def close_spider(self, spider):
        logging.warning("SPIDER CLOSED FROM PIPELINE")

    def process_item(self, item, spider):
        return item

```

Now go to `settings.py` and uncomment the `ITEM_PIPELINES` dictionary
```py
ITEM_PIPELINES = {
    'books.pipelines.BooksPipeline': 300,
}
```

```
# Beginning of output
2021-02-26 14:09:03 [root] WARNING: SPIDER OPENED FROM PIPELINE
# End of output
2021-02-26 14:10:57 [root] WARNING: SPIDER CLOSED FROM PIPELINE
```

## Storing DataSet to MongoDB

- go to https://www.mongodb.com/cloud
- sign in 
- create a new project and name it whatever
- select create new cluster
- select create cluster (this can take 7-10 minutes)
- go to terminal prompt and make sure your virtual environment is active
```
  conda activate virtual_workspace
  conda install pymongo dnspython
```
- go to Database Access, and add a user with password; you need this password in pymongo string
- go to clusters and then NETWORK ACCESS and enter 0.0.0.0/0 which will whitelist the database.
- then go to connect in clusters, 
  - click 'Connect your application' 
  - driver as Python
  - version as 3.6 or later
  - then copy the string template and add it to your `pipeline.py` file om pymongo.MongoClient("")
- then go to the `pipelines.py` file
  
```py
import logging
import pymongo

class MongodbPipeline(object):
    # Name of spider and collection in mongodb
    collection_name = "book_deals"
    # 
    def open_spider(self, spider):
        self.client = pymongo.MongoClient(
            "mongodb+srv://brock:123@cluster0.rvyih.mongodb.net/myFirstDatabase?retryWrites=true&w=majority")
        self.db = self.client["BOOKS"]

    def close_spider(self, spider):
        self.client.close()

    def process_item(self, item, spider):
        self.db[self.collection_name].insert(item)
        return item

  ```
now, go to the `settings.py` file and add this to the ITEM_PIPELINES dictionary

```py
ITEM_PIPELINES = {
    'books.pipelines.MongodbPipeline': 300
}
```
now run `scrapy crawl book_deals` then go to mongo => clusters => collections => BOOKS => BOOKS.book_deals

## SQLITE 3

```py
class SQLitePipeline(object):

    def open_spider(self, spider):
        self.connection = sqlite3.connect("books.db")
        self.c = self.connection.cursor()
        self.c.execute('''
            CREATE TABLE book_deals (
              book_name TEXT,
              price TEXT
            )
        ''')
        self.connection.commit()

    def close_spider(self, spider):
        self.connection.close()

    def process_item(self, item, spider):
        self.c.execute('''
            INSERT INTO book_deals (book_name, price) VALUES(?,?)
        ''', (
            item.get('book_name'),
            item.get('price')
        ))
        self.connection.commit()
        return item
```
```py
ITEM_PIPELINES = {    
    'books.pipelines.SQLitePipeline': 300
}
```
