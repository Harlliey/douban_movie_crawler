









# douban_movie_crawer


## 运行方法

* 本项目分为爬虫部分和前端信息展示部分，可分开运行:
   1. 爬虫运行方法:
        * 爬虫部分要求装有python3运行环境和部分依赖库,依赖库安装方法:
        ```pip3 install -r requirements.txt```
        * 可以在config.py文件修改默认的线程配置和数据库配置，老师可以直接使用默认配置
        * 运行: ```python3 main.py```
    2. 前端运行方法:
        * 前端及相应的后端在web/目录下, ```cd web```
        * 前端也有相应的config.py文件配置数据库端口及密码等
        * 运行: ```python3 run.py```
        * 注意: 前端展示网页默认开在5000端口,使用[http://localhost:5000/index](http://localhost:5000/index)访问
        * 老师可以直接访问 [http://39.108.123.85:5000/index](http://39.108.123.85:5000/index)访问我们
        运行在服务器上的展示页面
        
        
    
    



## 架构设计
* 使用豆瓣电影下所有分类作为入口进行爬取
* 爬取每部电影的详细信息和前200条左右的热门短评和影评，存入Mysql数据库
* 使用python多线程模块threading实现一个简易的线程池
* 爬虫架构:
![](https://ws3.sinaimg.cn/large/006tNbRwgy1fyr3mj5d8fj31740tsq6x.jpg)
* 共有五种类型的线程:
    * 爬取某一分类下所有电影的id，并将id加入*电影id*队列
    * 获取某一部电影详细信息，并将获取的详细信息加入到数据库插入队列,同时将该电影加入到获取热门短评和热门评论的队列
    * 获取热门短评线程从短评队列抽取一个电影来抓取它的短评，并将获取的短评加入到数据库插入队列
    * 获取热门评论线程同上
    * 数据库插入线程专门执行数据库的插入操作
    
    
    
## 爬取速率
* 仅爬取电影详细信息，开两个爬取线程和一个数据库插入线程，速度可以达到60部/分钟左右。但是1个数据库插入线程
不足以支持两个爬取线程的插入，数据库队列会逐渐变长，推荐1个爬取线程匹配一个数据库插入线程
![](https://ws3.sinaimg.cn/large/006tNbRwgy1fyr3guoedaj30uk0u0gq3.jpg)
![](https://ws2.sinaimg.cn/large/006tNbRwgy1fyr3h5v4skj30u00vwte0.jpg)
![](https://ws2.sinaimg.cn/large/006tNbRwgy1fyr3ho30jkj31p40iudqh.jpg)

* 等完成课程答辩，测试下100+线程的爬取速率(虽然python并没有实质的多线程，但是这种IO密集型的多线程还是有用的

## 开发日记
### 爬取流程:
1. 首先获取tag
2. 然后遍历tag 获取所有电影， 获取到的电影以Movie对象的形式加入到detail_movie队列
3. 由于以上两部不耗时间，所以可以直接顺序执行
4. 获取电影详细信息: 多线程从detaile_movoie队列中取出来，分别抓取详细信息后插入数据库，同时将每一步电影即movie_id或其他唯一标志符加入到
    短评队列和评论队列，因为只有数据库中存在的电影，才能插入相关短评和评论这是外键约束
5. 此时启动短评队列线程(>5) 以及评论队列( > 5)获取数据库中存在的电影的短评和评论
6. 注意 多线程访问同一队列时锁

### 反 豆瓣发爬虫
* 随机化UA, 项目中大概使用了十几个常见浏览器UA
* 新请求豆瓣会返回一个cookie，使用requests.session可以方便的管理cookie，然后基于
requests.session封装一个网络请求类，该类内部维持当前cookie已经发送请求的个数，目前是一旦
超过20次就清除cookie，重新获取一个cookie, 这样可以有效防止被封ip，每一个线程拥有一个独立的
网络请求类对象。所以可以开多线程爬取，但是也不要开太多线程，同一个ip请求过多也可能被封，亲测5个左右
可以很长时间不被封ip, 找个时候开100个线程测试下最高爬取速率
* 可以在项目中的MyOpener类内部实现代理，但是我测试过，免费代理可用率很低，所以就没有采用，
但是如果有付费代理，可以在里面扩展，每当返回403时就自动更换代理，这样其它爬取代码可以无修改

### 关于去重
* 直接在获取电影那一步实现去重

* 每一部电影的短评和评论爬取完毕后都必须将已经爬取的电影id插入到各自的两张表中

* 每次启动爬虫时都先从数据库读入当前已经读入已经爬取了详细信息的电影id
* 布隆过滤器
  
   
 
  
   