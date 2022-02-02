## 基本使用

```shell
# 文档：
    官方文档
    	http://cn.python‐requests.org/zh_CN/latest/ 
    快速上手
    	http://cn.python‐requests.org/zh_CN/latest/user/quickstart.html

# 安装:
	pip install requests
```



**response的属性以及类型:**

| 类型                 | models.Response    |
| -------------------- | ------------------ |
| response.text        | 获取网站源码       |
| response.encoding    | 访问或定制编码方式 |
| response.url         | 获取请求的url      |
| response.content     | 响应的字节类型     |
| response.status_code | 响应的状态码       |
| response.headers     | 响应的头信息       |



## get请求

```python
# requests.get()
import requests

url = 'http://www.baidu.com/s?' 
headers = { 'User‐Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36' }
data = { 'wd':'北京' }
response = requests.get(url,params=data,headers=headers)
```

1. 参数使用params传递 

2. 参数无需urlencode编码 

3. 不需要请求对象的定制 

4. 请求资源路径中？可加可不加



## post请求

```python
# requests.post()
import requests 

post_url = 'http://fanyi.baidu.com/sug' 
headers={ 'User‐Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36' } 
data = { 'kw': 'eye' }
r = requests.post(url = post_url,headers=headers,data=data)
```

**get和post区别:** 

1. get请求的参数名字是params 

   post请求的参数的名字是data 

2. 不需要做请求对象的定制

3. 不需要手动编解码

4. 请求资源路径后面可以不加?



## 代理

```python
# proxy定制
# 在请求中设置proxies参数
# 参数类型是一个字典类型
import requests 

url = 'http://www.baidu.com/s?' 
headers = { 'user‐agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36' } 
data = { 'wd':'ip' }
proxy = { 'http':'219.149.59.250:9797' } 
r = requests.get(url=url,params=data,headers=headers,proxies=proxy)

with open('proxy.html','w',encoding='utf‐8') as fp:
    fp.write(r.text)
```



## cookie定制

云打码平台(验证码识别API)
