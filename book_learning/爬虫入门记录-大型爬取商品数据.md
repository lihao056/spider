#### 2.用API爬取天气预报数据

​	某些网站会提供一些API供我们爬取信息，使我们不用通过网页源码获取数据。

##### 2.1 注册免费API

​	点击和风天气官网并创建一个账号，并创建一个应用获取信息。![1556889686661](D:\typora\1556889686661.png)

​	点击新建应用，应用名称可以随意订。

![1556889899882](D:\typora\1556889899882.png)

​	之后选择添加KEY后选择Web API，IP地址可以不用填写![1556890156360](D:\typora\1556890156360.png)

![1556890225087](D:\typora\1556890225087.png)

​	之后可以看到出现一个Key的值，说明可以使用该免费的API获取数据

![1556890276861](D:\typora\1556890276861.png)

​	可以通过网站提供的API文档来使用API获取资料，其中parameters是后面所用到的参数，weather-type是获取天气类型的值。

![1556890918427](D:\typora\1556890918427.png)

​	点击参考资料处可以获得我们想要的城市文件，复制其链接地址供我们使用

​	https://cdn.heweather.com/china-city-list.csv

![1556971606163](D:\typora\1556971606163.png)

##### 2.2 获取API数据

​	由于前一节已经知道城市代码的链接，所以可以通过此链接获取城市数据。

![1556972293813](D:\typora\1556972293813.png)

​	可以看到该文档是用Get方式获取。

```python
import requests
url = 'https://cdn.heweather.com/china-city-list.csv'
strhtml = requests.get(url)
# 将strhtml转换成str格式
data = strhtml.text
print(data)
```

​	下面是城市列表中的前几个，之后就需要对这些做处理

```
HEWeather China City List v2.1,update by 2018-06-20,,,,,,,,,,,,
City_ID,City_EN,City_CN,Country_code,Country_EN,Country_CN,Province_EN,Province_CN,Admin_ district_EN,Admin_ district_CN,Latitude,Longitude,AD_code,
CN101010100,beijing,北京,CN,China,中国,beijing,北京,beijing,北京,39.904987,116.40529,"110100,110000,100000",
CN101010200,haidian,海淀,CN,China,中国,beijing,北京,beijing,北京,39.956074,116.31032,110108,
CN101010300,chaoyang,朝阳,CN,China,中国,beijing,北京,beijing,北京,39.92149,116.48641,110105,
CN101010400,shunyi,顺义,CN,China,中国,beijing,北京,beijing,北京,40.128937,116.65353,110113,

```

```python
# 由上图的数据可以看到，这是一个str数据，每个都以'\n'换行，同时由于前两行是开头之类的，所以移除列表头两个
data1 = data.split('\n')[2]
```

![1556973575119](D:\typora\1556973575119.png)

​	由图片中可以看到，需要获取城市地点的代码或者名称以及刚刚申请的key值调用请求URL

```python
for item in data1:
    # 其中key中对应着你申请的值，item[0:11]表示城市代码
    url = 'https://free-api.heweather.net/s6/weather/now?location=' + item[0:11] + '&key=*******'
    strhtml = requests.get(url)
	# 使用time.sleep延时
    time.sleep(1)
    # 由于数据是以Json格式返回的，所以使用.json()方法转成json格式
    dic = strhtml.json()
```

​	可以使用在线的json工具可以看到相应的格式

![1556974722414](D:\typora\1556974722414.png)





##### 2.3 存储数据到MongoDB

###### 2.3.1 安装MongoDB并配置

​	可以使用MongoDB存储刚刚获取的数据

​	可以查看这个[链接](<https://www.cnblogs.com/zhuziyu/p/9213891.html>)下载并安装

​	每次启动的时候都需要启动MongoDB，需要先在命令行中cd至MongoDB的目录下

![1556975655333](D:\typora\1556975655333.png)

![1556975687217](D:\typora\1556975687217.png)

​	之后输入下面指令即可

```
mongod --dbpath d:\data\db			# 后面的目录可以随你建立
```

![1556975727826](D:\typora\1556975727826.png)

###### 2.3.2 在Pycharm中安装Mongo Plugin

​	这一步让Pycharm可以查看Mongo数据库的内容。

​	依次点击setting -> Plugins -> Browse Repositories 并搜索Mongok可以弹出界面，并安装即可。![1557062170084](D:\typora\1557062170084.png)

​	我这里出现一些错误，无法按照正常的步骤安装

​	网络上查到的[解决方法](<https://blog.csdn.net/biebersen/article/details/83515365>)，按照他的步骤可以解决。

​	安装完成后，点击

![1557062546238](D:\typora\1557062546238.png)

​	出现下图的界面，点击 + 号

![1557062555302](D:\typora\1557062555302.png)

​	Labal随便选择一个即可，之后点击test connection，若出现successful说明成功。

![1557062649223](D:\typora\1557062649223.png)

​	还需要安装一个pymongo库，这个库是用来提供python和MongoDB连接。

###### 2.3.3 将数据存入MongoDB

​	下面是一段使用MongoDB的代码

```python
'''
    使用API爬取数据
    将数据存入MongoDB数据库中
'''
import requests
import time
import pymongo

# 建立连接，其中localhost是主机名 
client = pymongo.MongoClient('localhost', 27017)
# 在MongoDB中新建名为weather的数据库
book_weather = client['weather']
# 在weather数据库中建立名为sheet_weather_3的表
sheet_weather = book_weather['sheet_weather_3']

url = 'https://cdn.heweather.com/china-city-list.csv'
strhtml = requests.get(url)
data = strhtml.text
data1 = data.split('\n')[2:]
for item in data1:
    url = 'https://free-api.heweather.net/s6/weather/now?location=' + item[0:11] + '&key=0d88444981d244a6abd3adc77635ca8d'
    strhtml = requests.get(url)
    time.sleep(1)
    print(strhtml.text)
    dic = strhtml.json()
    # 向表中插入一个数据
    sheet_weather.insert_one(dic)
```

​	最终的效果如下。

![1557226785143](D:\typora\1557226785143.png)

###### 2.3.4 MongoDB数据库查询

​	由于使用上面的代码已经爬取过一些数据了，可以使用代码读取一下刚刚爬取的一些数据。

```python
import pymongo
# 建立连接
client = pymongo.MongoClient('localhost', 27017)
book_weather = client['weather']
sheet_weather = book_weather['sheet_weather_3']
# 使用sheet.find()方法查询，无传入参数说明返回的是全部数据
for item in sheet_weather.find():
    tmp = item['HeWeather6'][0]['now']['tmp']
    print(tmp)
    # update_one用于指定更新一条数据，第一个参数是查询条件，对应_id字段。第二个参数表示更新信息，$set是一个修改器，将键值修改为为int类型
    sheet_weather.update_one({'_id':item['_id']}, {'$set':{'HeWeather6.0.now.tmp':int(tmp)}})
# 有参数说明，返回当前键值大于10的数据
for item in sheet_weather.find({'HeWeather6.0.now.tmp':{'$gt':10}}):
    print(item['HeWeather6'][0]['basic']['location'])
```

​	

| 修改符 | 应用                                     |
| ------ | ---------------------------------------- |
| $inc   | 对值为数字型的键进行增减操作             |
| $unset | 用于删除键                               |
| $push  | 向文档的某个数组类型的键增加一个数组元素 |

| 修改符 | 表示 |
| ------ | ---- |
| $gt    | >    |
| $lt    | <    |
| $lte   | <=   |
| $gte   | >=   |

