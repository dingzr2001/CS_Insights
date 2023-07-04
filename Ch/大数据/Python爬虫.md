# Python爬虫

## 一、Request库

### 1、request库安装

①同时按下win+R，输入cmd，打开命令行。

②输入pip install requests。

③输入python，启动python，再输入import requests。若不报错则安装成功。

### 2、Requests库的7个主要方法

requests.request()：构造一个请求，支撑一下各方法的基础方法。

requests.get()：获取html网页的主要方法，对应于http的get。

requests.head()：获取html网页头信息的方法，对应于http的head。

requests.post()：向html网页提交post请求的方法，对应于http的post。

requests.put()：向html网页提交put请求的方法，对应于http的put。

requests.patch()：向html网页提交patch请求的方法，对应于http的patch。

requests.delete()：向html页面提交删除请求，对应于http的DELETE。

### 3、Response对象的属性

r.status_code：http请求的返回状态，200表示连接成功，404表示失败，。

r.text：http响应内容的字符串形式，即url对应的页面内容。

r.encoding：从http header中猜测的相应内容编码方式

r.apprent_encoding：从内容中分析出的响应内容编码方式（备选编码方式）。

r.content：http响应内容的二进制形式。

### 4、Requests库异常

requests.ConnectionError：网络连接错误异常，如DNS查询失败、拒绝连接等。

requests.HTTPError：HTTP错误异常。

requests.URLRequired：URL缺失异常。

requests.TooManyRedirects：超过最大重定向次数，产生重定向异常。

requests.ConnectTimeout：连接远程服务器超时异常。

requests.Timeout：请求URL超时，产生超时异常。

### 5、理解Requests库的异常

r.raise_for_status()：如果不是200，产生异常requests.HTTPError。

### 6、http协议

#### URL格式：`http://host[:port][path]`

host：合法的Internet主机域名或IP地址。

port：端口号，可不写，默认为80。

path：请求资源的路径。

## 二、Beautiful Soup库

### 1、安装

打开cmd，输入pip install beautifulsoup4。

### 2、使用

`from bs4 import BeautifulSoup`

`soup=BeautifulSoup(data,'html.parser')`

其中data为爬取的网页源代码，html.parser为对data的解释器。

### 3、基本元素

#### bs4库的理解

bs4库是解析、遍历、维护“标签树”的功能书。

`<p class="title">..</p>`

`<p>..</p>`是以p为名称的标签类型。

class=“title”是该标签的属性域，是一个键值对。

#### 引用方式

`from bs4 import BeautifulSoup`

`import bs4`

#### 解析器

bs4的HTML解析器：BeautifulSoup(mk,'html.parser') 需安装bs4库。

lxml的HTML解析器：BeautifulSoup(mk,'lxml')需pip install lxml。

lxml的XML解析器：BeautifulSoup(mk,'xml')需pip install lxml。

html5lib的解析器：BeautifulSoup(mk,'html5lib')需pip install html5lib。

#### BeautifulSoup类的基本元素

Tag：标签，最基本的信息组织单元，分别用<>和</>标明开头或结尾。

Name：标签的名字，<p>...</p>的名字是p，格式：<tag>.name。

Attributes：标签的属性，字典形式组织，格式：<tag>.attrs

NavigableString：标签内非属性字符串，<>...</>中字符串，格式为<tag>.string。

Comment：标签内字符串的注释部分，一种特殊的Comment类型。

### 4、基于bs4库的HTML内容遍历方法

#### 标签树下行遍历

.contents：子节点的列表，将<tag>所有儿子节点存入列表。

.children：子节点的迭代类型，与.content类似，用于循环遍历儿子节点。

.descendants：子孙节点的迭代类型，包含所有子孙节点，用于循环遍历。

#### 标签树的上行遍历

.parent：节点的父亲标签

.parents：节点先辈（不止父亲）标签的迭代类型，用于循环遍历先辈节点。

#### 标签树的平行遍历

.next_sibling：返回按照html文本顺序的下一个平行节点标签。

.previous_sibling：返回按照html文本顺序的上一个平行节点标签。

.next_siblings：迭代类型，返回按照html文本顺序的后续所有平行节点标签。

.previous_siblings：迭代类型，返回按照html文本顺序的前序所有平行节点标签。

遍历后续节点

`for sibling in soup.a.next_siblings:`

​        `print(sibling)`

遍历前序节点

`for sibling in soup.a.previous_siblings:`

​        `    print(sibling)`



#迭代类型只能用在循环中！！！

#### 基于bs4库的HTML格式化和编码

**格式化**

.prettify()：为标签添加换行符。

编码：UTF-8，支持中文。

#### 基于bs4库的HTML内容查找方法

.find_all(name,attrs,recursive,string,**kwargs)

name：对标签名称的检索字符串。

attrs：对标签属性值的检索字符串，可标注属性检索。

recursive：是否对子孙所有节点进行搜索，默认为true。

string：<>...</>中字符串区域的检索字符串。

简写：

<tag>(...)等价于<tag>.find_all(...)

soup(...)等价于soup.find_all(...)

#### 扩展方法

<>.find()：搜索且只返回一个结果，字符串类型，同.find_all()参数。

<>.find_parents()：在先辈节点中搜索，返回列表类型，同.find_all()参数。

<>.find_parent()：在先辈节点中返回一个结果，字符串类型，同.find()参数。

<>.find_next_siblings()：在后续平行节点中搜索，返回列表类型，同.find_all()参数。

<>.find_next_sibling()：在后续平行节点中返回一个结果，字符串类型，同.find()参数。

<>.find_previous_siblings()：在前序平行节点中搜索，返回列表类型，同.find_all()参数。

<>.find_previous_sibling()：在前序平行节点中返回一个结果，字符串类型，同.find()参数。

## 三、正则表达式

### 1、正则表达式的概念

用来表达字符串的简洁方式。

通用的字符串表达框架。

字符串匹配。

### 2、正则表达式的语法

#### 正则表达式长常用操作符

.：表示任何单个字符。

[]：字符集，对单个字符给出取值范围。[abc]表示a,b,c，[a-z]表示a到z的单个字符。

[^ ]：非字符集，对单个字符给出排除范围。`[^abc]`表示非a或非b或非c的单个字符。

*：前一个字符出现0次或无限次扩展。abc`*`表示ab，abc，abcc，abccc等。

+：前一个字符1次或无限次扩展。abc+表示abc，abcc，abcc等

?：前一个字符0次或1次扩展。abc?表示ab、abc。

|：左右表达式任意一个。abc|def表示abc、def。

{m}：表示扩展前一个字符m次。ab{2}c表示abbc。

{m,n}：扩展前一个字符m至n次（含n），ab{1,2}c表示abc、abbc。

^：匹配字符串开头。^abc表示abc且在一个字符串的开头。

$：匹配字符串结尾。abc$表示abc且在一个字符串的结尾。

()：分组标记，内部只能使用|操作符。(abc)表示abc，(abc|def)表示abc、def。

\d：数字，等价于[0-9]。

\w：单词字符，等价于[A-za-z0-9_]。

#匹配中文字符串：[\u4e00-\u9fa5]

#### 匹配IP地址的正则表达式

IP地址分4段，每段0-255。

(([1-9]?\d|1\d{2}|2[0-4]\d|25[0-5]).){3}([1-9]?\d|1\d{2}|2[0-4]\d|25[0-5])

### 3、Re库的基本使用

#### Re库介绍

Re库是python标准库，主要用于字符串的匹配。

调用方式：import re

#### 正则表达式的表示类型

**raw string类型（原生字符串类型）**

用法：r''。即在字符串前面加一个r。

原生字符串：不含转义符\的字符串。

**string类型**

即不加r，是平时用的字符串，更繁琐。

需用`\\`来表示\。

#### Re库主要功能函数

**re.search(pattern,string,flags=0)**：

在一个字符串中搜索匹配正则表达式的第一个位置，返回match对象。

pattern：正则表达式的字符串或原生字符串表示。

string：待匹配字符串。

flags：使用时的控制标记。

re.I | re.IGNORECASE：忽略正则表达式的大小写，[A-Z]能够匹配小写字符。

re.M | re.MULTILINE：正则表达式种的^操作符能够将给定字符串的每行当作匹配开始。

re.S | re.DOTALL：正则表达式种的.操作符能够匹配所有字符，默认匹配除换行外的所有字符。

**re.match(pattern,string,flags=0)**：

从一个字符串的开始位置起匹配正则表达式，返回match对象。

参数含义同re.search。

**re.findall(pattern,string,flags=0)**：

搜索字符串，以列表类型返回全部能匹配的字串。

参数含义同re.search。

**re.splite(pattern,string,maxsplit=0,flags=0)**：

将一个字符串按照正则表达式匹配结果进行分割，返回列表类型。

参数含义同re.search。

maxsplit：最大分割数，剩余部分作为最后一个元素输出。

**re.finditer(pattern,string,flags=0)**：

搜索字符串，返回一个匹配结果的迭代类型，每个迭代元素是match对象。

参数含义同re.search。

**re.sub(pattern,repl,string,count=0,flags=0)**：

在一个字符串中替换所有匹配正则表达式的子串，返回替换后的子串。

参数含义同re.search。

repl：替换匹配字符串的字符串。

count：匹配的最大替换次数。

#re.compile：面向对象。

rst=re.search(r'[1-9]\d{5},'BIT 100081")

等价于

pat=re.compile(r'[1-9]\d{5}')

rst=pat.search('BIT 100081')

#**re.compile(pattern,flags=0)**：

将正则表达式的字符串形式编译成正则表达式对象。

### 4、Re库的match对象

#### Match对象的属性

.string：待匹配的文本。

.re：匹配时使用的pattern对象（正则表达式）。

.pos：正则表达式搜索文本的开始位置。

.endpos：正则表达式搜索文本的结束位置。

#### Match对象的方法

.group(0)：获得匹配后的字符串。

.start()：匹配字符串在原始字符串的开始位置。

.end()：匹配字符串在原始字符串的结束位置。

.span()：返回(.start(),.end())

### 5、Re库的贪婪匹配和最小匹配

#### 贪婪匹配

Re库默认采用贪婪匹配，即输出匹配最长的子串。

#### 最小匹配操作符

*?：前一个字符0次或无限次扩展，最小匹配。

+?：前一个字符1次或无限次扩展，最小匹配。

??：前一个字符0次或1次，最小匹配。

{m,n}?：扩展前一个字符m至n次(含n)，最小匹配。

## 四、Scrapy框架

### 1、Scrapy库安装

打开cmd输入`pip install scrapy`。

### 2、Scrapy爬虫框架结构

#### 5+2结构、数据流

![img.JPG](https://github.com/dingzr2001/CS_Insights/blob/main/Ch/%E5%A4%A7%E6%95%B0%E6%8D%AE/img.JPG?raw=true)

### 3、Scarpy爬虫框架解析

**ENGINE:**

控制所有模块之间的数据流。

根据条件触发事件。

不需要用户修改。

**DOWNLOADER:**

根据请求下载网页。

不需要修改。

**SCHEDULER:**

对所有爬取请求进行调度管理。

不需要用户修改

**DOWNLOADER MIDDLEWARE**

downloader和engine两个模块之间的中间键。

目的：实施Engine、Scheduler和Downloader之间进行用户可配置的控制。

功能：修改、丢弃、新增请求或响应。

用户可以编写代码修改。

**Spider：**

解析Downloader返回的响应（Response）。

产生爬取项（scraped item）。

产生额外的爬取请求（Request）。

需要用户编写配置代码。

**Item Pipelines:**

以流水线方式处理Spider产生的爬取项。

由一组操作顺序组成，类似流水线，每个操作是一个Item Pipeline类型。

可能操作包括：清理、检验和查重爬取项中的HTML数据、将数据存储到数据库。

需要用户编写配置代码。

**Spider Middleware**

Spider和Engine之间的中间键。

目的：对请求和爬取项的再处理。

功能：修改、丢弃、新增请求或爬取项。

用户可以编写配置代码。

### 4、Scrapy爬虫的常用命令

#### Scrapy命令行(cmd)

**startproject**

创建一个新工程。

scrapy startproject<name>[dir]

**genspider**

创建一个爬虫。

scrapy genspider[options]<name><domain>

**settings**

获得爬虫配置信息。

scrapy settings[options]

**crawl**

运行一个爬虫。

scrapy crawl<spider>

**list**

列出工程中所有爬虫。

scrapy list

**shell**

启动url调试命令行。

scrapy shell[url]

#### 操作步骤

**1、建立一个Scrapy爬虫工程**

①、打开命令行，切换到自己希望存储的目录。

②、输入`scrapy startproject 文件名`。

③、生成工程目录

文件夹：外层目录

  scrapy.cfg：部署scrapy爬虫的配置文件。

  文件夹：scrapy框架的用户自定义python代码。

​    `_init_.py`：初始化脚本

​    `item.py`：Items代码模板（继承类）

​    `middlewares.py`：Middlewares代码模板（继承类）

​    `pipelines.py`：pipelines代码模板（继承类）

​    `settings.py`：scrapy爬虫的配置文件

​    `spiders/`：spiders代码模板目录（继承类）

​      `_init_.py`：初始文件，无需修改。

​      `_pycache_/`：缓存目录，无需修改。

**2、在工程中产生一个Scrapy爬虫**

命令行中输入`scrapy genspider 文件名 爬取网站网址`

**3、配置产生的spider爬虫**

```python
#例子
import scrapy


class DemoSpider(scrapy.Spider):
    name = 'demo'
    #allowed_domains = ['python123.io']
    start_urls = ['http://python123.io/ws/demo.html']

    def parse(self, response):
        fname=response.url.split('/')[-1]
        with open(fname,'wb') as f:
            f.write(response.body)
        self.log('Saved file %s.' % name)
        pass
```

**4、运行爬虫，获取网页**

### 5、yield关键词使用

**yeild<-->生成器**

1、生成器是一个不断产生值的函数。

2、包含yield语句的函数是一个生成器。

3、生成器每次产生一个值（yield语句），函数被冻结，被唤醒后再产生一个值。

**实例**

生成器写法

```python
def gen(n):
    for j in range(n):
        yield j**2
#产生所有小于n的数的平方值，运行到yield这一行时，运算i**2的值并返回，然后函数被冻结。
for i in gen(5):
    print(i, " ", end="")
#>>>输出结果：0  1  4  9  16
#这块儿只可意会不可言传，大概解释一下就是，进入主函数的for循环的时候，先运行gen(5)，此时进入gen函数，第一次走gen里的循环，j=0，经过yield，返回0给i，此时i为0。第一次i循环结束。走第二次，再次调用gen，此时gen里的j=1，重复上述。按我的说法就是，yield是一个有记录状态功能的return。
```

### 6、Scrapy爬虫的基本使用

#### Scrapy爬虫的使用步骤：

1、创建一个工程和Spider模板

2、编写Spider

3、编写Item Pipeline

4、优化配置策略

#### Scrapy爬虫的数据类型：

**Request类**

`class scrapy.http.Request()`

Request对象表示一个HTTP请求。

由Spider生成，由Downloader执行。

属性或方法：

.url：Request对应的请求URL地址。

.method：对应的请求方法，'GET''POST'等。

.headers：字典类型风格的请求头。

.body：请求内容主体，字符串类型。

.meta：用户添加的扩展信息，在Scrapy内部模块间传递信息使用。

.copy()：复制该请求。

**Response类**

`class scrapy.http.Response()`

Response对象表示一个HTTP响应。

由Downloader生成，由Spider处理。

属性或方法：

.url：Response对应的url地址。

.status：HTTP状态码，200、404、403……

.headers：Response对应的头部信息。

.body：Response对应的内容信息，字符串类型。

.flags：一组标记。

.request：产生Response类型对应的Request对象。

.copy()：复制该响应。

**Item类**

`class scrapy.item.Item()`

Item对象表示一个从HTML页面中提取的信息内容。

由Spider生成，由Item Pipeline处理。

Item类似字典类型，可以按照字典类型操作。

#### Scrapy爬虫支持的HTML信息提取方法

Beautiful Soup

lxml

re

XPath Selector

**CSS Selector**

使用格式：`<html>.css('a::attr(href)').extract()`

其中，a为标签名称，href是标签属性。

