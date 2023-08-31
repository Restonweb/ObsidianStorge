
```python
from urllib.request import urlopen

import urllib.request

  

Myurl = urlopen("https://www.runoob.com/")

  

print("------------------------------")

print(Myurl.read(300))#read()可以读取整个网页的内容，可以指定字节数

print("------------------------------")

print(Myurl.readline().decode)#读取一行

print("------------------------------")

lines = Myurl.readlines()

for line in lines:#读取全部内容 存于列表变量中

    print(line)

print("------------------------------")

print(Myurl.getcode())#正常输出200

try:

    Myurls = urllib.request.urlopen("https://www.runoob.com/no.html")#获取网页状态码判断是否正常

except urllib.error.HTTPError as e:

    if e.code == 404:

        print('404')#不存在 404

print("------------------------------")

  

f = open('saved.html',"wb")

content = Myurl.read()

f.write(content)#用read()将网页文件写入文件

f.close

print("------------------------------")
```
