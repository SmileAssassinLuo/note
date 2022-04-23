#### scrapy

应用程序框架

##### 一，安装

cmd 命令行 `pip install scrapy -i https://pypi.douban.com/simple`

##### 二，开发项目

1. 选择目标网站
2. 定义要抓取的数据（通过`Scrapy Items`来完成的）
3. 编写提取数据的`spider`
4. 执行`spider`，获取数据
5. 数据存储

`scrapy startproject mySpider`: 创建项目

`cd mySpider` , `scrapy genspider baidu "https://www.baid.com"` 创建一个爬虫文件

`scrapy  crawl baidu`  执行爬虫

##### 三，目录文件说明

```python
“
scrapy.cfg ：项目的配置文件

mySpider/ ：项目的Python模块，将会从这里引用代码

mySpider/items.py ：项目的目标文件

mySpider/pipelines.py ：项目的管道文件

mySpider/settings.py ：项目的设置文件

mySpider/spiders/ ：存储爬虫代码目录
mySpider/spiders/ init.py 爬虫的核心文件
”
```

``` 
response 属性和方法
response.text  返回字符串
response.body  返回二进制
span_list = response.xpath("//div[@id='filter']/div[@class='tabs']/a/span")  直接使用xpath方法解析response的内容
response.extract()        提取selector对象的属性值
response.extract_first()  提取selector列表的第一个数据
```



##### 四，scrapy架构组成,原理

![](D:\work\gitRespository\note\Python\img\scrapy原理.png)





