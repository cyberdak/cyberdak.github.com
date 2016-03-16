---
layout : post
title : centos 上 折腾 安装 scrapy 的经历
---

准备给xxtv加上段子的模块，所以打算用python做一个抓去段子的爬虫。

#最快的办法是先安装下来的依赖，最后安装python。因为每次安装完都需要重新编译，第一次踩坑是难免的。

第一步是pyenv安装 py 2.7 和py 3.4
我是两个都安装了，防止该用的时候不知道如何挑选

还是和记忆中安装一样，到处是问题

总得来说是下面这么几个问题

1. bz2
2. lxml
3. sqlite3

第一个问题"compressionerror bz2 module is not available",需要安装bz2模块

```
yum install bzip2-devel -y
```

然后，重新编译python

第二个问题,"Could not find function xmlCheckVersion in library libxml2. Is libxml2 installed?"

直接"pip install lxml"依然会提示这个错误。

执行下面的代码
```
yum remove audit
yum install gcc
yum install libxslt-devel libxml2-devel
pip install xml
```

然后重新编译python

再次安装scrapy,这次终于可以安装成功了。。。

但是执行官网的demo代码，又不成功了，提示找不到_sqlite3模块。。

又得再安装了

去sqlite官网找一个最新的源代码，然后下载编译

```
wget http://www.sqlite.org/2016/sqlite-autoconf-3110100.tar.gz
tar zxvf sqlite-autoconf-3110100.tar.gz
cd sqlite-autoconf-3110100
./configure
make
make install
```

然后重新编译python，再安装scrapy。这次终于是运行成功了。

```
[root@iZ28zmja50jZ py]# scrapy runspider myspider.py
2016-03-13 01:07:49 [scrapy] INFO: Scrapy 1.0.5 started (bot: scrapybot)
2016-03-13 01:07:49 [scrapy] INFO: Optional features available: ssl, http11
2016-03-13 01:07:49 [scrapy] INFO: Overridden settings: {}
2016-03-13 01:07:49 [scrapy] INFO: Enabled extensions: CloseSpider, TelnetConsole, LogStats, CoreStats, SpiderState
2016-03-13 01:07:49 [scrapy] INFO: Enabled downloader middlewares: HttpAuthMiddleware, DownloadTimeoutMiddleware, UserAgentMiddleware, RetryMiddleware, DefaultHeadersMiddleware, MetaRefreshMiddleware, HttpCompressionMiddleware, RedirectMiddleware, CookiesMiddleware, ChunkedTransferMiddleware, DownloaderStats
2016-03-13 01:07:49 [scrapy] INFO: Enabled spider middlewares: HttpErrorMiddleware, OffsiteMiddleware, RefererMiddleware, UrlLengthMiddleware, DepthMiddleware
2016-03-13 01:07:49 [scrapy] INFO: Enabled item pipelines: 
2016-03-13 01:07:49 [scrapy] INFO: Spider opened
2016-03-13 01:07:49 [scrapy] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
2016-03-13 01:07:49 [scrapy] DEBUG: Telnet console listening on 127.0.0.1:6023
2016-03-13 01:07:50 [scrapy] DEBUG: Crawled (200) <GET https://blog.scrapinghub.com/> (referer: None)
2016-03-13 01:07:50 [scrapy] DEBUG: Crawled (200) <GET https://blog.scrapinghub.com/category/autoscraping/> (referer: https://blog.scrapinghub.com/)
2016-03-13 01:07:50 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/autoscraping/>
{'title': u'Announcing Portia, the Open Source Visual Web\xa0Scraper! on'}
2016-03-13 01:07:50 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/autoscraping/>
{'title': u'Introducing Dash on'}
2016-03-13 01:07:50 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/autoscraping/>
{'title': u'Spiders activity graphs on'}
2016-03-13 01:07:50 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/autoscraping/>
{'title': u'Autoscraping casts a wider\xa0net on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Crawled (200) <GET https://blog.scrapinghub.com/category/tools/> (referer: https://blog.scrapinghub.com/)
2016-03-13 01:07:51 [scrapy] DEBUG: Crawled (200) <GET https://blog.scrapinghub.com/category/splash/> (referer: https://blog.scrapinghub.com/)
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/tools/>
{'title': u'Scrapy Tips from the Pros: Part\xa01 on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/tools/>
{'title': u'Why we moved to\xa0Slack on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Crawled (200) <GET https://blog.scrapinghub.com/category/scrapy-cloud/> (referer: https://blog.scrapinghub.com/)
2016-03-13 01:07:51 [scrapy] DEBUG: Crawled (200) <GET https://blog.scrapinghub.com/category/scrapy-tips-from-the-pros/> (referer: https://blog.scrapinghub.com/)
2016-03-13 01:07:51 [scrapy] DEBUG: Crawled (200) <GET https://blog.scrapinghub.com/category/scrapy/> (referer: https://blog.scrapinghub.com/)
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/splash/>
{'title': u'The Road to Loading JavaScript in\xa0Portia on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/splash/>
{'title': u'Google Summer of Code\xa02015 on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/splash/>
{'title': u'Handling JavaScript in Scrapy with\xa0Splash on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Crawled (200) <GET https://blog.scrapinghub.com/category/scrapinghub/> (referer: https://blog.scrapinghub.com/)
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/scrapy-cloud/>
{'title': u'Black Friday, Cyber Monday: Are They Worth\xa0It? on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/scrapy-cloud/>
{'title': u'Introducing Data Reviews on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/scrapy-cloud/>
{'title': u'Introducing Dash on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/scrapy-cloud/>
{'title': u'Why MongoDB Is a Bad Choice for Storing Our Scraped\xa0Data on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/scrapy-cloud/>
{'title': u'Git Workflow for Scrapy\xa0Projects on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/scrapy-cloud/>
{'title': u'Spiders activity graphs on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/scrapy-tips-from-the-pros/>
{'title': u'Scrapy Tips from the Pros: February 2016\xa0Edition on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/scrapy-tips-from-the-pros/>
{'title': u'Scrapy Tips from the Pros: Part\xa01 on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/scrapy/>
{'title': u'Scrapy Tips from the Pros: February 2016\xa0Edition on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/scrapy/>
{'title': u'Python 3 is Coming to\xa0Scrapy on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/scrapy/>
{'title': u'Scrapy Tips from the Pros: Part\xa01 on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/scrapy/>
{'title': u'Chats With RINAR\xa0Solutions on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/scrapy/>
{'title': u'Black Friday, Cyber Monday: Are They Worth\xa0It? on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/scrapy/>
{'title': u'Scrapy on the Road to Python 3\xa0Support on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/scrapy/>
{'title': u'Google Summer of Code\xa02015 on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Crawled (200) <GET https://blog.scrapinghub.com/category/professional-services/> (referer: https://blog.scrapinghub.com/)
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/scrapinghub/>
{'title': u'How Web Scraping is Revealing Lobbying and Corruption in\xa0Peru on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/scrapinghub/>
{'title': u'Splash 2.0 Is Here with Qt 5 and Python\xa03 on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/scrapinghub/>
{'title': u'Migrate your Kimono Projects to\xa0Portia on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/scrapinghub/>
{'title': u'Portia: The Open Source Alternative to Kimono\xa0Labs on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/scrapinghub/>
{'title': u'Web Scraping Finds Stores Guilty of Price\xa0Inflation on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/scrapinghub/>
{'title': u'Python 3 is Coming to\xa0Scrapy on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/scrapinghub/>
{'title': u'Happy Anniversary: Scrapinghub Turns\xa05 on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Crawled (200) <GET https://blog.scrapinghub.com/category/products/> (referer: https://blog.scrapinghub.com/)
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/professional-services/>
{'title': u'Scrapinghub Crawls the Deep\xa0Web on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/professional-services/>
{'title': u'Finding Similar Items on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/products/>
{'title': u'Migrate your Kimono Projects to\xa0Portia on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/products/>
{'title': u'Portia: The Open Source Alternative to Kimono\xa0Labs on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/products/>
{'title': u'Announcing Portia, the Open Source Visual Web\xa0Scraper! on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/products/>
{'title': u'Introducing Dash on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/products/>
{'title': u'Introducing Crawlera, a Smart Page\xa0Downloader on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Crawled (200) <GET https://blog.scrapinghub.com/category/open-source-2/> (referer: https://blog.scrapinghub.com/)
2016-03-13 01:07:51 [scrapy] DEBUG: Crawled (200) <GET https://blog.scrapinghub.com/category/new-hires/> (referer: https://blog.scrapinghub.com/)
2016-03-13 01:07:51 [scrapy] DEBUG: Crawled (200) <GET https://blog.scrapinghub.com/category/portia-2/> (referer: https://blog.scrapinghub.com/)
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/open-source-2/>
{'title': u'Portia: The Open Source Alternative to Kimono\xa0Labs on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/open-source-2/>
{'title': u'Scrapy on the Road to Python 3\xa0Support on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/open-source-2/>
{'title': u'The Road to Loading JavaScript in\xa0Portia on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/open-source-2/>
{'title': u'Google Summer of Code\xa02015 on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/open-source-2/>
{'title': u'Frontera: The Brain Behind the\xa0Crawls on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/open-source-2/>
{'title': u'The History of\xa0Scrapinghub on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/open-source-2/>
{'title': u'Skinfer: A Tool for Inferring JSON\xa0Schemas on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Crawled (200) <GET https://blog.scrapinghub.com/category/euro-python/> (referer: https://blog.scrapinghub.com/)
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/new-hires/>
{'title': u'Marcos Campal Is a\xa0ScrapingHubber! on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/portia-2/>
{'title': u'Migrate your Kimono Projects to\xa0Portia on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/portia-2/>
{'title': u'Portia: The Open Source Alternative to Kimono\xa0Labs on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/portia-2/>
{'title': u'The Road to Loading JavaScript in\xa0Portia on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/portia-2/>
{'title': u'Scrape Data Visually with Portia and Scrapy\xa0Cloud on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Crawled (200) <GET https://blog.scrapinghub.com/category/remote-working-2/> (referer: https://blog.scrapinghub.com/)
2016-03-13 01:07:51 [scrapy] DEBUG: Crawled (200) <GET https://blog.scrapinghub.com/category/human-resources/> (referer: https://blog.scrapinghub.com/)
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/euro-python/>
{'title': u'Scrapy on the Road to Python 3\xa0Support on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/euro-python/>
{'title': u'EuroPython, here we\xa0go! on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/remote-working-2/>
{'title': u'Looking Back at\xa02015 on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/remote-working-2/>
{'title': u'Using git to manage vacations in a large distributed\xa0team on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/remote-working-2/>
{'title': u'Traveling Tips for Remote\xa0Workers on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/remote-working-2/>
{'title': u'A Career in Remote\xa0Working on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/remote-working-2/>
{'title': u'Scrapinghub: A Remote Working Success\xa0Story on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/human-resources/>
{'title': u'Using git to manage vacations in a large distributed\xa0team on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Crawled (200) <GET https://blog.scrapinghub.com/category/crawlera/> (referer: https://blog.scrapinghub.com/)
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/crawlera/>
{'title': u'Black Friday, Cyber Monday: Are They Worth\xa0It? on'}
2016-03-13 01:07:51 [scrapy] DEBUG: Scraped from <200 https://blog.scrapinghub.com/category/crawlera/>
{'title': u'Introducing Crawlera, a Smart Page\xa0Downloader on'}
2016-03-13 01:07:51 [scrapy] INFO: Closing spider (finished)
2016-03-13 01:07:51 [scrapy] INFO: Dumping Scrapy stats:
{'downloader/request_bytes': 4692,
 'downloader/request_count': 17,
 'downloader/request_method_count/GET': 17,
 'downloader/response_bytes': 193649,
 'downloader/response_count': 17,
 'downloader/response_status_count/200': 17,
 'finish_reason': 'finished',
 'finish_time': datetime.datetime(2016, 3, 12, 17, 7, 51, 941390),
 'item_scraped_count': 60,
 'log_count/DEBUG': 78,
 'log_count/INFO': 7,
 'request_depth_max': 1,
 'response_received_count': 17,
 'scheduler/dequeued': 17,
 'scheduler/dequeued/memory': 17,
 'scheduler/enqueued': 17,
 'scheduler/enqueued/memory': 17,
 'start_time': datetime.datetime(2016, 3, 12, 17, 7, 49, 371136)}
2016-03-13 01:07:51 [scrapy] INFO: Spider closed (finished)
[root@iZ28zmja50jZ py]# 
```

