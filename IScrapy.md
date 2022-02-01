scrapy
===

[scrapy官网](https://scrapy.org/)

> scrapy是一个python的爬虫框架，负责网页的爬取、分析等。不支持任务调度、渲染

### install

```sh
sudo apt install python3
pip3 install scrapy
```

### 简单例子

```python
import scrapy

class BlogSpider(scrapy.Spider):
    name = 'blogspider'
    start_urls = ['https://blog.scrapinghub.com']

    def parse(self, response):
        for title in response.css('.post-header>h2'):
            yield {'title': title.css('a ::text').get()}

        for next_page in response.css('a.next-posts-link'):
            yield response.follow(next_page, self.parse)
```

```sh
scrapy runspider myspider.py
```

### projects 模板

> scrapy提供自动创建模板的方式，通过创建的模板可以快速方便的开发爬虫逻辑

```
scrapy startproject spinder_pig .
```

创建完成后，会形成如下目录结构

```sh
scrapy.cfg
spinder_pig/
    __init__.py
    items.py
    middlewares.py
    pipelines.py
    settings.py
    spiders/
        __init__.py
        spider1.py
        spider2.py
        ...
```

#### scrapy.cfg

```ini
[settings]
default = spinder_pig.settings

[deploy]
project = spinder_pig

```

`spinder_pig`为对应的项目名称

#### item

在spinder中和pipline传递的数据对象，需要在`items.py`中进行声明

```python
import scrapy
class SpinderPigItems(scrapy.Item):
    url = scrapy.Field()
    title = scrapy.Field()
    content = scrapy.Field()
    date = scrapy.Field()
    spided_time = scrapy.Field()
    site = scrapy.Field()
```

#### piplines

接受从spiders中获取的数据，并进行相关的操作，如存储数据库

```python

def save_item(item):
    values = (item['url'], item['site'], item['title'], item['content'], item['date'], item['spided_time'])
    result = dao.exe_select("select url from spided where url=%s", (item['url'],))
    if len(result) == 0:
        dao.exe_insert("insert into spided (url,site,title,content,date,spided_time) values(%s,%s,%s,%s,%s,%s)", values)
    else:
        print("WARN:%s is in db,so do not insert it" % item['url'])


class SpinderMinePipeline:
    def __init__(self):
        pass

    def close_spider(self, spider):
        dao.close()

    def process_item(self, item, spider):
        save_item(item)
        return item
```



#### settings.py

全局参数配置

```ini
BOT_NAME = 'spinder_pig'
SPIDER_MODULES = ['spinder_pig.spiders']
NEWSPIDER_MODULE = 'spinder_pig.spiders'
ITEM_PIPELINES = {
    'spinder_pig.pipelines.SpinderMinePipeline': 300,
}

# Crawl responsibly by identifying yourself (and your website) on the user-agent
#USER_AGENT = 'spinder_pig (+http://www.yourdomain.com)'
USER_AGENT = 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.62 Safari/537.36'

# Obey robots.txt rules
ROBOTSTXT_OBEY = False

# for Splash
SPLASH_URL = 'http://bjrdc218:8050'
SPIDER_MIDDLEWARES = {
    'scrapy_splash.SplashDeduplicateArgsMiddleware': 100,
}

DOWNLOADER_MIDDLEWARES = {
    'scrapy_splash.SplashCookiesMiddleware': 723,
    'scrapy_splash.SplashMiddleware': 725,
    'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware': 810
}

DUPEFILTER_CLASS = 'scrapy_splash.SplashAwareDupeFilter'
HTTPCACHE_STORAGE = 'scrapy_splash.SplashAwareFSCacheStorage'
```



#### spiders

爬虫逻辑实现代码，该目录下可以包含多个spider，每个spider为一个py文件。

如下为一个spider代码样例片段

```python
class PigSpider(scrapy.Spider):
    name = "spinder_pig"
    start_urls = config.keys()

    def parse(self, response):
        yield from self.__parse_news(response)
        for i in range(1, 10):
            pass

    def start_requests(self):
        for url in self.start_urls:
            if config[url]["use_splash"] is True:
                logger.info("request %s by Splash" % url)
                yield SplashRequest(url, callback=self.parse)
            else:
                logger.info("request %s by scrapy" % url)
                yield scrapy.Request(url=url, callback=self.parse)

    def __parse_news_detail(self, response, news_url):
        logger.info('parse_news_detail:%s', response.url)
        title = self.__remove_tags(response.xpath(config[news_url]["news_title"]).extract())
        content = self.__remove_a(response.xpath(config[news_url]["news_content"]).extract())
        date = self.__remove_tags(response.xpath(config[news_url]["news_date"]).extract())
        item = SpinderPigItems()
        item['site'] = news_url
        item['url'] = response.url if response.url.find("?") < 0 else response.url[0:response.url.index("?")]
        item['title'] = title
        item['content'] = content
        item['date'] = date
        item['spided_time'] = datetime.now().strftime('%Y.%m.%d-%H:%M:%S')
        yield item
...        
```

其中的item代码中产生的item数据会发送到pipeline中

#### middlewares

> The spider middleware is a framework of hooks into Scrapy’s spider processing mechanism where you can plug custom functionality to process the responses that are sent to [Spiders](https://docs.scrapy.org/en/latest/topics/spiders.html#topics-spiders) for processing and to process the requests and items that are generated from spiders.

```ini
SPIDER_MIDDLEWARES = {
    'spinder_pig.middlewares.SpinderRockSpiderMiddleware': 543,
}

# Enable or disable downloader middlewares
# See https://docs.scrapy.org/en/latest/topics/downloader-middleware.html
DOWNLOADER_MIDDLEWARES = {
    'spinder_pig.middlewares.SpinderRockDownloaderMiddleware': 543,
}
```



## splash

[splash官网](https://splash.readthedocs.io/en/stable/index.html)

在通过scrapy进行爬取的时候，经常遇到一些通过js进行页面渲染的情况，scrapy是不支持页面渲染的，为了获取到渲染后的内容，需要通过第三方组件进行。splash就是其中比较好的一种

### install

splash可以通过docker运行，通过如下命令直接运行。

```sh
docker run -d -p 8050:8050 scrapinghub/splash --max-timeout 3600
```

该命令会自动下载splash的镜像并制动运行，可以通过浏览器访问如下地址查看运行状态

```http
http://bjrdc218:8050
```



### curl

可以使用curl来进行简单的测试，相关命令如下.

```sh
curl 'http://bjrdc218:8050/render.html?url=http://clgl.cadc.net.cn:8087/tpub/trkqry/YSC-3707820557-3J82-B8F543F1-9E03-4BB7-AE48-6E16942584B6&baseurl=http://clgl.cadc.net.cn:8087/tpub/trkqry/'
```

返回的内容为已经渲染的页面，但是如果页面上有相对路径的时候可能无法渲染。

### scrapy-splash

splash与scrapy做了整合，可以通过scrapy的架构快速实现splash的渲染。

#### 配置

```ini
# for Splash
SPLASH_URL = 'http://bjrdc218:8050'
SPIDER_MIDDLEWARES = {
    'scrapy_splash.SplashDeduplicateArgsMiddleware': 100,
    'spinder_pig.middlewares.SpinderPigSpiderMiddleware': 544
}

DOWNLOADER_MIDDLEWARES = {
    'scrapy_splash.SplashCookiesMiddleware': 723,
    'scrapy_splash.SplashMiddleware': 725,
    'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware': 810,
    'spinder_pig.middlewares.SpinderPigDownloaderMiddleware': 543
}

DUPEFILTER_CLASS = 'scrapy_splash.SplashAwareDupeFilter'
HTTPCACHE_STORAGE = 'scrapy_splash.SplashAwareFSCacheStorage'
```

#### 代码

```python
import re
import scrapy
from scrapy.linkextractors import LinkExtractor
from scrapy_splash import SplashRequest
from w3lib.html import remove_tags


class Sohu(scrapy.Spider):
    name = 'sohu'
    start_urls = ['https://mp.sohu.com/profile?xpt=c29odXptdHR5MnZ3YmFAc29odS5jb20=']

    # start request
    def start_requests(self):
        for url in self.start_urls:
            yield SplashRequest(url, callback=self.parse)

    def __init__(self):
        self.news_links = LinkExtractor(restrict_xpaths="//div[@class='feed-texts']/h4[@class='feed-title']/a")

    def parse(self, response):
        news_links = self.news_links.extract_links(response);
        # news_links = response.xpath("//div[@class='feed-texts']/h4[@class='feed-title']/a").extract()
        for news in news_links:
            print(news)
            yield scrapy.Request(url=news.url, callback=self.__parse_news_page)

    def __remove_tags(self, content):
        return re.sub(r'[\t\r\n]', '', remove_tags(content))

    def __parse_news_page(self, response):
        title = response.xpath("//div[@class='text-title']/h1/text()").get()
        source = self.__remove_tags(
            "".join(response.xpath("//div[@class='article-info']/span[@class='time']/text()").extract()))
        content = response.xpath("//article[@class='article']").extract()
        print("title:", title)
        print("source", source)
        print("content", content)

```

此种方法能够基本基本的渲染的问题，但是无法获取到通过js赋值,在html中查看不到的内容

### lua

为解决对页面的交互性操作，如

1. 通过鼠标点击获取数据
2. 通过滚动获取内容
3. 通过js获取页面元素内容

如下代码解决一个网站无法通过html代码获取元素内容的情况。

```python
import scrapy
from scrapy_splash import SplashRequest

script = """
function main(splash, args)
    splash.images_enabled = false
    splash:set_user_agent('Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.67 Safari/537.36')
    assert(splash:go(args.url))
    splash:wait(1)
    assert(splash:autoload("http://clgl.cadc.net.cn:8087/tpub/js/jquery-1.11.3.min.js"))
    splash:wait(1)
    local run=splash:evaljs("GetTruckDetail(GetRequest(location.href))")
    local truck_no=splash:evaljs("$('#truck_no').val()")
    local truck_num=splash:evaljs("$('#truck_num').val()")
    local truck_owner=splash:evaljs("$('#truck_owner').val()")
    local owner_telno=splash:evaljs("$('#owner_telno').val()")
    local recorder=splash:evaljs("$('#recorder').val()")
    local valid_date_start=splash:evaljs("$('#valid_date_start').val()")
    local valid_date_end=splash:evaljs("$('#valid_date_end').val()")
    local owner_idcard=splash:evaljs("$('#owner_idcard').val()")
    local incounty=splash:evaljs("$('#incounty').val()")
    return {
        truck_no=truck_no,
        truck_num=truck_num,
        owner_telno=owner_telno,
        truck_owner=truck_owner,
        recorder=recorder,
        valid_date_start=valid_date_start,
        valid_date_end=valid_date_end,
        owner_idcard=owner_idcard,
        incounty=incounty
    }
end
"""


class Cadc(scrapy.Spider):
    name = 'cadc'
    start_urls = ['http://clgl.cadc.net.cn:8087/tpub/trkqry/YSC-3707820557-3J82-B8F543F1-9E03-4BB7-AE48-6E16942584B6',
                  "http://clgl.cadc.net.cn:8888/CarPT/CZ_ChaKanYe/ErWeiMaOpen.aspx?CLNumber=6bKBTkMzN0E2&TableName=Q1pfQ1FUcmFuc3BvcnRTaGFuZ0Rvbmc="]
    def start_requests(self):
        for url in self.start_urls:
            yield SplashRequest(url,
                                callback=self.parse,
                                endpoint='execute',
                                args={
                                    'baseurl':"http://clgl.cadc.net.cn:8087/tpub/trkqry/",
                                    'lua_source': script
                                })

    def parse(self, response):
        truck_num = response.xpath("//input[@id='truck_no']/text()").extract()
        print("GET:%s" % truck_num)
        print("BODY:%s" % response.body)

```

