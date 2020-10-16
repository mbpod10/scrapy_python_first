# SCRAPY

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

### Open Scrapy Shell

- Open Shell

```
scrapy shell
```

- See If We Get A Response

```
fetch("https://www.worldometers.info/world-population/population-by-country/")
```

- Make Request Into A Object Variable

```
r = scrapy.Request(url="https://www.worldometers.info/world-population/population-by-country/")
```

- Fetch

```
fetch(r)
```

- Get The HTML

```
response.body
```

- Do this

```s
2020-10-16 12:45:30 [asyncio] DEBUG: Using selector: KqueueSelector
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
```
