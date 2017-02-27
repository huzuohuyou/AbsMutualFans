# AbsMutualFans
- 这里是列表文本启动fiddler，打开网易晕音乐官网，进入个人中心粉丝模块
![输入图片说明](https://static.oschina.net/uploads/img/201702/26213217_PGlH.png "在这里输入图片标题")
- 这里是列表文本查看在fiddler中抓到的包，进行分析
惊奇地发现有个方法叫做 getfollows 好巧哦！
![输入图片说明](https://static.oschina.net/uploads/img/201702/26213501_5J9Z.png "在这里输入图片标题")

- 这里是列表文本对比分析对你已经偷偷取消关注的人与没有对你取消关注的人的数据区别
![输入图片说明](https://static.oschina.net/uploads/img/201702/26214029_CwvB.png "在这里输入图片标题")

- 这里是列表文本找一个已经对你取消关注的人进行“取消关注操作”
![输入图片说明](https://static.oschina.net/uploads/img/201702/26222514_3Gw0.png "在这里输入图片标题")
- 这里是列表文本编码分两步 
    第一步：获取你关注的但没有关注你的人
    第二部：将这些人进行取关操作
在这个过程中遇到了一些麻烦主要是，网易云音乐对页码进行了加密，没法轻松地获取所有的对你取关的人，我这里采用的是比较笨的办法就是把三十多个跳也页的参数都手动复制下来放到文本文件中。
![输入图片说明](https://static.oschina.net/uploads/img/201702/26215612_3vC7.png "在这里输入图片标题")
- 三十条页码参数记录-
![输入图片说明](https://static.oschina.net/uploads/img/201702/26215813_J1UP.png "在这里输入图片标题")

```
import urllib
import http.cookiejar
import ssl
import requests
```
抓取那些对你取关的人
```
def getNoMutal(params,enSecKey):
    print(params)
    print(enSecKey)
    data = {'params': str(params.strip()),
            'encSecKey':str(enSecKey.strip())}
    headers = {
    'POST http':'//music.163.com/weapi/user/getfollows/112272936?csrf_token=8c133a69f9cbf30a37e04ef55af6444f HTTP/1.1',
    'Host':'music.163.com',
    'Connection':' keep-alive',
    'Content-Length':' 512',
    'Originv http':'//music.163.com',
    'User-Agent':' Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/49.0.2623.110 Safari/537.36',
    'Content-Type':' application/x-www-form-urlencoded',
    'Accept':' */*',
    'Referer':' http://music.163.com/user/follows?id=112272936',
    'Accept-Encoding':' gzip, deflate',
    'Accept-Language':' zh-CN,zh;q=0.8',
    'Cookie':'你的cookie在fiddler中粘过来'}
    r = requests.post('http://music.163.com/weapi/user/getfollows/112272936?csrf_token=8c133a69f9cbf30a37e04ef55af6444f', data=data, headers=headers)
    print(r.text)
    import json
    data = json.loads(r.text)
    defollows=[]
    for follower in data['follow']:
        if follower['mutual']==False:
            defollows.append(follower['userId'])
    print('defollows:'+str(defollows))
    return defollows

```

对这些用户进行取关
```
def defollowed(defollows):
    data = {'params': 'c3S6p3BC4SU7idoGy4GpyJ/Az7LlSC0KmIcCfi9435TbeYMrSBtOqgFAqdIwoLecalp1RBLHOvfpuZ/RT3OfwSnJ/zs5TEW'
                       '+PqRWMrCHVcoqm4qFYK72Tm7aLSTCbATvINJ1JPBmDRFntv4TLRNaSA==',
            'encSecKey':'d577dd3f46d316483db1ea103e3e55d4d07cefb055135c4ae462c703c1d24061c6b0b67aa94c3eb10a52da65cc9321f6'
                        'd92b8e2b8635909ae4e0679e01f56a42dd4ff6f34375d0cfbd11cce355bdbfab385b3ea4834e777d17da4631a959b7e9f2'
                        'be23da3e908925bdb2500b5187a57dda16704b1fa9069cae85bdb02344d732'}
    header={
    #'POST http://music.163.com/weapi/user/delfollow/127028008?csrf_token=8c133a69f9cbf30a37e04ef55af6444f HTTP/1.1'
    'Host':' music.163.com',
    'Connection':' keep-alive',
    'Content-Length':' 438',
    'Origin':' http://music.163.com',
    'User-Agent':' Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/49.0.2623.110 Safari/537.36',
    'Content-Type':' application/x-www-form-urlencoded',
    'Accept':' */*',
    'Referer':' http://music.163.com/user/home?id=127028008',
    'Accept-Encoding':' gzip, deflate',
    'Accept-Language':' zh-CN,zh;q=0.8',
    'Cookie':'你的cookie在fiddler中粘过来''            
    }
    for id in defollows:
        print('id:'+str(id))
        url='http://music.163.com/weapi/user/delfollow/{userid}?csrf_token=8c133a69f9cbf30a37e04ef55af6444f'.format(userid=str(id))
        print('url:'+url)
        r = requests.post(url,data=data,headers=header)
        print('defollow {userid} {result}'.format(userid=id,result=r.text))
```

最后运行
```
if __name__ == '__main__':
    defollows=[]#getNoMutal()

    file = open("params&key")
    index=0
    while 1:
        index=index+1
        line = file.readline()
        if not line:
            break
        else:
            params=line.split('#')[0]
            enSecKey=line.split('#')[1]
            defollows.extend(getNoMutal(params,enSecKey))
    defollowed(defollows)
```

- 运行脚本前

![输入图片说明](https://static.oschina.net/uploads/img/201702/26220404_2j5Y.png "在这里输入图片标题")
- 运行脚本后

![输入图片说明](https://static.oschina.net/uploads/img/201702/26220421_0hNe.png "在这里输入图片标题")
