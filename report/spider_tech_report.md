# [Python]网络爬虫总结

> 颜泽鑫 15331348
> 本文将对Python网络爬虫进行简要的总结，涵盖了我目前所使用的所有方法。

常见的网页形式主要有两大类：
* 静态网页
* 动态网页

所谓的静态的网页，就是网页编写者会将网页数据都直接写入到html中，对于这样的网页，一般而言是无法进行数据更新的，也就是说你今天打开这个网页获得的信息和你一个月后在这个网页获得信息是一样的，不会有任何的改变。

所谓的动态的网页，就是网页编写者只是将网页写成一个框架，具体的数据会放在服务器的数据库了。就比如说，网页是一个书架，你希望获得金融类的书籍，那你就可以向服务器发出这么一个请求——“我希望获得金融类的书籍”，那么服务器就会返回相应的书籍，书架上就会呈现相应的金融类的书籍。这里的请求实际上就是http请求，也就是网页作为前端与服务器作为后端之间的信息通信。动态网页是目前比较常见的网页形式，因为大数据的存在，网页逐渐成为一种呈现的方式，具体的数据会保存在服务器的数据库中，并且不断地改变着。

对于具体的爬虫来说，对于这两种方式，会采用不同的爬虫策略。

## 静态网页
对于静态网页，就不多说了，太简单了。只要用requests库直接把html爬下来，然后用正则表达式匹配即可。但是到了目前互联网发展阶段，已经很少有静态网页了。如果你遇到要爬虫静态网页，那你一定是非常幸福了。
例如这样的网页：[你的名字](http://www.dy2018.com/i/97747.html) 就可以认为是一个静态网页。

```
# -*- coding: utf-8 -*-
import requests
session = requests.session()
# 设置信息头，用于模拟浏览器的行为
session.headers = {'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_5)'
                                 'AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36'}

r = session.get('http://www.dy2018.com/i/97747.html')
html = r.content
# 解码中文
print(html.decode('gb2312'))

```

## 动态网页
动态网页是比较常见的爬虫目标，这里我给出一些比较常见的爬虫方法，仅供参考。

### 爬取数据包
一般来说，要爬虫的内容都是在格式上具有一定的重复性，但同时数据量又非常大。如果你曾经做过网页开发，你就会明白网页开发者对于这样的数据，一般都会采取从服务器发数据包到前端，在前端解析数据的方式来实现，于是这就给了爬虫者巨大的便利。因为一旦我找到了数据包的请求方式，我就可以仿照前端发送相同的请求，来获得相应的JSON数据。

这样请求一般可以认为是http请求，http请求主要分为两种形式：
* Get方法：比如说我们在浏览器上输入一个网络地址，就是发起一个Get方法的请求。这种网络地址就是URL。
* Post方法：在爬虫中不常见，故不详细介绍

对于爬虫者来说，只需要知道Get方法是如何传递参数的即可。在前文，我提到网页就是一个书架，如果我希望书架上的书都是金融类的书，那么我就需要向服务器发送一个需要书的请求，并且这个请求中的一个参数就是“金融类”，于是服务器就能明白我想要的书是金融类的书。

具体来看，这么一个例子：

```
http://www.shgtj.gov.cn/i/tdsc/dklb/?pn=1&ps=15&gh=&fs=&mc=&qx=&bh=&zt=1
```
这是一个Get方法的请求，`?`后面是所有的参数，参数之间用`&`来连接。对于这样的URL，我们一般很难把每个参数的意思都把握好，因为对于不同的coder来说，如何命名参数的方法是不同的，所以我们只需要把一些重要参数的意思把握好就行了。就比如说这个请求，pn很明显是页号（page number），ps很明显是每一页的数量。其他的就不得而知了，也不需要知道了。

剩下的一个问题就是如何找到这个请求了。我个人认为，如果这个网站确实是用这种方式来传递的数据的话，这个请求一般是非常好找的，在chrome的inspect里面，查看network。如果network里面什么都没有话，就刷新。

具体案例如下：

```
import requests
import json
import pandas as pd

session = requests.session()
# 设置请求头信息
session.headers = {'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_5)'
                                 'AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36'}


# 设置获得列表的URL
def get_list_url(page):
    url = 'http://www.shgtj.gov.cn/i/tdsc/dklb/?pn=' + str(page) + '&ps=15&gh=&fs=&mc=&qx=&bh=&zt=1'
    return url


# 设置获得详细信息的URL
def get_detail_url(id):
    url = 'http://www.shgtj.gov.cn/i/tdsc/crdk/?id=' + str(id)
    return url


# 发送请求，获得相应的结果
def get_info(url):
    html = session.get(url)
    # 获得的文本一般都是JSON，即字典
    info = json.loads(html.text)
    return info

# 需要保存的数据
data = pd.DataFrame(columns=['id', '出让面积', '竞得价(万元)', '竞得日期', '工业用地', '容积率'])

for i in range(9, 23, 1):
    list_url = get_list_url(page=i)
    list_info = get_info(list_url)
    id_list = []
    for each in list_info['data']['list']:
        # 获得公告号
        id_list.append(each['bianh'])
    for each in id_list:
        detail_url = get_detail_url(each)
        detail_info = get_info(detail_url)['data']
        #zongbj 竞得价(万元)
        #guihyt 工业用地
        #mianj 出让面积
        #chengjrq 竞得日期
        #rongjl 容积率
        data.loc[data.shape[0] + 1] = {'id': each, '出让面积': detail_info['mianj'],
                                       '竞得价(万元)': detail_info['zongbj'], '竞得日期': detail_info['chengjrq'],
                                       '工业用地': detail_info['guihyt'], '容积率': detail_info['rongjl']}
    # 将文件写为csv。
    data.to_csv('data.csv', index=False, encoding='utf-8')

```


### 非发送数据的动态网页
当然，并不是所有的网页都是用发送请求的方式来获得数据的。例如，[雪球网](https://xueqiu.com/9919963656/91414697) 这样的网页，我个人是花了很长时间也没有找到相应的请求。其实也是很容易理解的，像发送请求的话，一般都是结构化的数据，像这种文章就不是结构化的数据，所以不是发送请求的方式，也是可以理解的。但具体这个网页是怎么写，我个人也无法找到合适的解释。

那么，在这种情况下，就需要采取别的方法来进行网络爬虫了。这里我认为最简单的方法就是使用Selenium。具体的安装不在此赘述。

像[雪球网](https://xueqiu.com/9919963656/91414697) 这个网站的文章，就可以这么写了。代码就非常简单了。Selenium其实本来是用来做网站的自动化测试的，他会模拟浏览器的行为，所以可以直接获得浏览器渲染后的最终结果，也就是说，不管这个网站是什么方式写成的，采用Selenium获得的信息，都是我们正常在浏览器中能够看到的最终结果。换句话说，只要学会Selenium，你就爬虫任何的网站！但是，Selenium的速度非常地慢，所以我个人的爬虫策略是，如果能找到http请求，优先使用http请求，如果不能，就使用Selenium。

具体案例如下：

```
import json
import time
from selenium import webdriver

# 读取需要访问的列表
directoryFile = open('./TopLine_directory/directory.json', 'r', encoding='utf-8')
directoryData = json.load(directoryFile)
driver = webdriver.Chrome()

baseUrl = "https://xueqiu.com"

for eachItem in directoryData:
    driver.get(baseUrl + eachItem['target']) # 打开相应的网站
    title = driver.find_element_by_class_name("status-title").text # 获得文章标题
    info = driver.find_element_by_class_name("status-content").text # 获得文章内容
    f = open('./TopLine_directory/paper/' + title + '.txt', 'w', encoding='utf-8') # 写入本地文件
    print(title + "finished!")
    f.write(info)
    f.close()
    time.sleep(2)
```


## 优化及一些爬虫细节
如果找到了http请求，那么采用发送http请求来获得数据的方法总是优先选择的，但这需要说明的是，这种方法虽然很快，但是对服务器压力非常地大（你甚至可以做到在1分钟内发送500个请求）。服务器的编写者，一般都会写反爬虫程序。我个人觉得，不管编写者是否采取了反爬虫程序，爬虫者都应该遵循君子协议，尽量不要对服务器造成太大的伤害，同时又获得我们需要的数据。

对此，其中一个方法，就是控制速度，也就说每访问一次后就等待几秒。但是这个方法只能解决一点点问题，因为服务器会很容易地认为这是一个robot，于是对其进行封锁。但是必要的控制速度是对网页开发者的尊重。

另一个方式就是更改IP。但我在实践过程中发现，就算更改IP，服务器还是可以对爬虫者进行封锁。这里有可能是因为时钟采用同样的cookie，导致服务器能够发现robot。但是伪造合适的cookie，似乎不是这么简单的事情，以后我解决了再说吧。

以下是可以自动爬取一些免费的IP地址，用于进行网络爬虫。

这里使用了两个网站。
[国内高匿IP](http://www.xicidaili.com/nn)
[检测IP地址](http://ip.chinaz.com/getip.aspx)

```
import requests
import random
import re


session = requests.session()
session.headers = {'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_5)'
                                 'AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36'}


def random_IP(IP_array):
    proxies = {}
    index = random.randint(0, len(IP_array) - 1)
    proxies['http'] = 'http://' + IP_array[index][0] + ':' + IP_array[index][1]
    print(IP_array[index][0])
    return proxies


def get_new_IP():
    r = session.get('http://www.xicidaili.com/nn')
    IP = re.findall(r'<td>(([0-9]|\.)*)</td>', r.text)
    IP_array = []
    for i in range(len(IP)):
        if i % 2 == 0:
            IP_array.append([IP[i][0], IP[i + 1][0]])
    return IP_array

IP_array = get_new_IP()

while True:
    try:
        r = session.get('http://ip.chinaz.com/getip.aspx', proxies=random_IP(IP_array))
        print(r.text)
    except Exception as e:
        print(e)
```











