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
view(response)
title = response.xpath("//h1")
title
// Out[8]: [<Selector xpath='//h1' data='<h1>Countries in the world by populat...'>]
```
