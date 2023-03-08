---
tags: 💽/🐍
aliases: 
  - Scrapy
cssclass:
---

# [[Scrapy]]

> [Scrapy 2.8 Docs](https://docs.scrapy.org/en/latest/)

---

> Note:  All instructions are performed in MacOS on Scrapy 2.8 with Python 3.9.13 virtual environment

## Installation

```bash
pip install Scrapy
```

## Create a Project

This command will create a PROJECT_NAME directory with multiple .py files.

```bash
scrapy startproject PROJECT_NAME
```


## Create a Spider

A Spider is a class that you define which Scrapy uses to scrape information from a website (or group of websites).

Example Spider from Scrapy tutorial:

```python
from pathlib import Path

import scrapy

class QuotesSpider(scrapy.Spider):
    name = "quotes"

    def start_requests(self):
        urls = [
            'https://quotes.toscrape.com/page/1/',
            'https://quotes.toscrape.com/page/2/',
        ]
        for url in urls:
            yield scrapy.Request(url=url, callback=self.parse)

    def parse(self, response):
        page = response.url.split("/")[-2]
        filename = f'quotes-{page}.html'
        Path(filename).write_bytes(response.body)
        self.log(f'Saved file {filename}')
```


## Running a Spider

```bash
scrapy crawl quotes
```

### What just happened?

- Scrapy schedules the `scrapy.Request` objects returned by the `start_requests` method of the Spider. Upon receiving a response for each one, it instantiates `Response` objects and calls the callback method associated with the request (in this case, the `parse` method) passing the response as argument.


## Shortcut to `start_requests`method

In this case, the `parse()` method will be called to handle each of the requests for those URLs in `start_urls`, even though we haven't explicitly told Scrapy to do so.

```python
from pathlib import Path

import scrapy

class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'https://quotes.toscrape.com/page/1/',
        'https://quotes.toscrape.com/page/2/',
    ]

    def parse(self, response):
        page = response.url.split("/")[-2]
        filename = f'quotes-{page}.html'
        Path(filename).write_bytes(response.body)
```

## ⭐ Extracting Data

Extracting data using [Scrapy shell](https://docs.scrapy.org/en/latest/topics/shell.html#topics-shell):

> Note: If on Windows, use double quotes (" ") instead of single
```bash
scrapy shell 'https://quotes.toscrape.com/page/1/'
```
After executing the line above, Scrapy will scrape the website pasting the response in the terminal. The following example commands are ran inside this terminal, however Scrapy allows python commands to be submitted.

### Examples of Commands for Data Extraction

```python
response.css('title') # -> [<Selector xpath='descendant-or-self::title' data='<title>Quotes to Scrape</title>'>]

response.css('title::text').getall() # -> ['Quotes to Scrape']
response.css('title').getall() # -> ['<title>Quotes to Scrape</title>']
response.css('title::text').get() # -> 'Quotes to Scrape'
response.css('title::text')[0].get() # -> 'Quotes to Scrape'
response.css('noelement')[0].get() # -> IndexError: list index out of range

# Use `.get()` directly on the `SelectorList` instance to return `None` if there are no results
response.css("noelement").get() # -> None

# Regular Expressions
response.css('title::text').re(r'Quotes.*') # -> ['Quotes to Scrape']
response.css('title::text').re(r'Q\w+') # -> ['Quotes']
response.css('title::text').re(r'(\w+) to (\w+)') # -> ['Quotes', 'Scrape']

# XPath (besides CSS, Scrapy selectors supports XPath expressions)
response.xpath('//title') # -> [<Selector xpath='//title' data='<title>Quotes to Scrape</title>'>]
response.xpath('//title/text()').get() # -> 'Quotes to Scrape'
```


### ⭐ Extracting Quotes and Authors

The HTML `div` element looks like this:

```html
<div class="quote">
    <span class="text">“The world as we have created it is a process of our
    thinking. It cannot be changed without changing our thinking.”</span>
    <span>
        by <small class="author">Albert Einstein</small>
        <a href="/author/Albert-Einstein">(about)</a>
    </span>
    <div class="tags">
        Tags:
        <a class="tag" href="/tag/change/page/1/">change</a>
        <a class="tag" href="/tag/deep-thoughts/page/1/">deep-thoughts</a>
        <a class="tag" href="/tag/thinking/page/1/">thinking</a>
        <a class="tag" href="/tag/world/page/1/">world</a>
    </div>
</div>
```

**This:**
```python
# Get a list of selectors for the quote HMTL elements with:
response.css("div.quote")
```
**Returns:**
```python
[<Selector xpath="descendant-or-self::div[@class and contains(concat(' ', normalize-space(@class), ' '), ' quote ')]" data='<div class="quote" itemscope itemtype...'>,
 <Selector xpath="descendant-or-self::div[@class and contains(concat(' ', normalize-space(@class), ' '), ' quote ')]" data='<div class="quote" itemscope itemtype...'>,
 ...]
```

```python
# Extract the following as such
quote = response.css("div.quote")[0]
text = quote.css("span.text::text").get()
author = quote.css("small.author::text").get()

# text -> '"The world as we have created it is a process...etc"'
# author -> 'Albert Einstein'
```

Given that the tags are a list of strings, we can use the `.getall()` method to get all of them:
```python
tags = quote.css("div.tags a.tag::text").getall()
# tags -> ['change', 'deep-thoughts', 'thinking', 'world']
```

Having figured out how to extract each bit, we can now iterate over all the quotes elements and put them together into a Python dictionary:
**This:**
```python
for quote in response.css("div.quote"):
	text = quote.css("span.text::text").get()
	author = quote.css("small.author::text").get()
	tags = quote.css("div.tags a.tag::text").getall()
	print(dict(text=text, author=author, tags=tags))
```
**Returns:**
```python
{
	'text': '“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”',
	'author': 'Albert Einstein',
	'tags': ['change', 'deep-thoughts', 'thinking', 'world'],
}

{
	'text': '“It is our choices, Harry, that show what we truly are, far more than our abilities.”',
	'author': 'J.K. Rowling',
	'tags': ['abilities', 'choices'],
}
```


## ⭐ Extracting Data in Spider

In this example, the spider generates many dictionaries containing the data extracted from the page. This is why we use `yield` in the callback.
```python
import scrapy

class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'https://quotes.toscrape.com/page/1/',
        'https://quotes.toscrape.com/page/2/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').get(),
                'author': quote.css('small.author::text').get(),
                'tags': quote.css('div.tags a.tag::text').getall(),
            }
```


## Following Links

The ability to scrape stuff from all pages in a website

**HTML Link to the next page**
```html
<ul class="pager">
    <li class="next">
        <a href="/page/2/">Next <span aria-hidden="true">&rarr;</span></a>
    </li>
</ul>
```

**Link extraction code**
```python
response.css('li.next a').get() # -> '<a href="/page/2/">Next <span aria-hidden="true">→</span></a>'
response.css('li.next a::attr(href)').get() # -> '/page/2/'
response.css('li.next a').attrib['href'] # -> '/page/2/'
```

**Final Spider with link following code**
```python
import scrapy

class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'https://quotes.toscrape.com/page/1/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').get(),
                'author': quote.css('small.author::text').get(),
                'tags': quote.css('div.tags a.tag::text').getall(),
            }

        next_page = response.css('li.next a::attr(href)').get()
        if next_page is not None:
            next_page = response.urljoin(next_page)
            yield scrapy.Request(next_page, callback=self.parse)
```

### Shortcut for Creating Requests

Unlike `scrapy.Request` above, `response.follow` supports relative URLs directly–no need to call `response.urljoin`. 
```python
# response.follow uses their href attribute automatically
for a in response.css('ul.pager a'):
	yield response.follow(a, callback=self.parse)

# To create multiple requests from an iterable, you can use `response.follow_all` instead
anchors = response.css('ul.pager a')
yield from response.follow_all(anchors, callback=self.parse)

# Above consolidated further
yield from response.follow_all(css='ul.pager a', callback=self.parse)
```


### More Examples and Patterns

```python
import scrapy

class AuthorSpider(scrapy.Spider):
    name = 'author'

    start_urls = ['https://quotes.toscrape.com/']

    def parse(self, response):
        author_page_links = response.css('.author + a')
        yield from response.follow_all(author_page_links, self.parse_author)

        pagination_links = response.css('li.next a')
        yield from response.follow_all(pagination_links, self.parse)

    def parse_author(self, response):
        def extract_with_css(query):
            return response.css(query).get(default='').strip()

        yield {
            'name': extract_with_css('h3.author-title::text'),
            'birthdate': extract_with_css('.author-born-date::text'),
            'bio': extract_with_css('.author-description::text'),
        }
```

This spider will start from the main page, it will follow all the links to the authors pages calling the `parse_author` callback for each of them, and also the pagination links with the `parse` callback as we saw before.

Here we’re passing callbacks to [`response.follow_all`](https://docs.scrapy.org/en/latest/topics/request-response.html#scrapy.http.TextResponse.follow_all "scrapy.http.TextResponse.follow_all") as positional arguments to make the code shorter; it also works for `Request`.

The `parse_author` callback defines a helper function to extract and cleanup the data from a CSS query and yields the Python dict with the author data.

**Another interesting thing this spider demonstrates is that, even if there are many quotes from the same author, we don’t need to worry about visiting the same author page multiple times. By default, Scrapy filters out duplicated requests to URLs already visited, avoiding the problem of hitting servers too much because of a programming mistake.** This can be configured by the setting [`DUPEFILTER_CLASS`](https://docs.scrapy.org/en/latest/topics/settings.html#std-setting-DUPEFILTER_CLASS).

## Q/A

- **Does Scrapy respect the site's robots.txt file?** [link](https://stackoverflow.com/questions/55297976/scrapy-and-respect-of-robots-txt)
	- According to the docs, it's enabled by default only when you create a project using `scrapy startproject` command, otherwise should be default `False`.
	- Answering your question, yes, `scrapy shell` command does respect `robots.txt` configuration defined in `settings.py`. If `ROBOTSTXT_OBEY = True`, trying to use `scrapy shell` command on a protected URL will generate a response `None`.
	- You can also test it passing robots.txt settings via command line:
		- `scrapy shell https://www.netflix.com --set="ROBOTSTXT_OBEY=True"




🔗 Links to this page:
[[Python]]