```python
urllib.request.urlopen() 模拟浏览器向服务器发送请求 
response 服务器返回的数据 
        response的数据类型是HttpResponse # <class 'http.client.HttpResponse'>
        字节‐‐>字符串 解码decode 
        字符串‐‐>字节编码 encode 
        read() 字节形式读取二进制 扩展：rede(5)返回前几个字节 
        readline() 读取一行 
        readlines() 一行一行读取 直至结束 
        getcode() 获取状态码 
        geturl() 获取url 
        getheaders() 获取headers 
urllib.request.urlretrieve() # urlretrieve(url,filename)
        请求网页
        请求图片
        请求视频
```



## 请求对象的定制

```python
语法：request = urllib.request.Request()
```

```python
# 编解码
get请求方式：
	urllib.parse.quote(str) # 将字符转换成Unicode编码
	urllib.parse.urlencode(dict) # 传入字典

post请求方式: urllib.request.Request(url=url,headers=headers,data=data)
```

- get请求方式的参数必须编码，参数是拼接到url后面，(urlencode)编码之后**不需要调用encode方法**
- post请求方式的参数必须编码，参数是放在请求对象定制的方法中，(urlencode)编码之后**需要调用encode方法**



## URLError\HTTPError

1. HTTPError类是URLError类的子类 
2. 导入的包urllib.error.HTTPError urllib.error.URLError
3. .http错误：http错误是针对浏览器无法连接到服务器而增加出来的错误提示。引导并告诉浏览者该页是哪里出了问题
4. 通过urllib发送请求的时候，有可能会发送失败，这个时候如果想让你的代码更加的健壮，可以通过try‐except进行捕获异常，异常有两类，URLError\HTTPError



## Handler处理器

```python
urllib.request.urlopen(url) # 不能定制请求头 
urllib.request.Request(url,headers,data) # 可以定制请求头 
Handler # 定制更高级的请求头 随着业务逻辑的复杂 请求对象的定制已经满足不了我们的需求(动态cookie和代理 不能使用请求对象的定制)
```

```python
import urllib.request
url = 'http://www.baidu.com'
headers = { 'User-Agent': '...' } 
request = urllib.request.Request(url=url,headers=headers)

# 获取handler对象
handler = urllib.request.HTTPHandler()
# 获取opener对象
opener = urllib.request.build_opener(handler)
# 调用open方法
response = opener.open(request)
print(response.read().decode('utf‐8'))
```



## 代理服务器

1. 代理的常用功能

   1. 突破自身IP访问限制，访问国外站点

   2. 访问一些单位或团体内部资源

      扩展：某大学FTP(前提是该代理地址在该资源的允许访问范围之内)，使用教育网内地址段免费代理服务器，就可以用于对教育网开放的各类FTP下载上传，以及各类资料查询共享等服务

   3. 提高访问速度

      扩展：通常代理服务器都设置一个较大的硬盘缓冲区，当有外界的信息通过时，同时也将其保存到缓冲区中，当其他用户再访问相同的信息时， 则直接由缓冲区中取出信息，传给用户，以提高访问速度

   4. 隐藏真实IP

      扩展：上网者也可以通过这种方法隐藏自己的IP，免受攻击

2. 代码配置代理

   - 创建Reuqest对象
   - 创建**ProxyHandler对象**  `proxies = {'http':'ip:port'} `   `urllib.request.ProxyHandler(proxies=proxies) ` 
   - 用handler对象创建opener对象
   - 使用opener.open函数发送请求

> 快代理 https://www.kuaidaili.com/free/

