---
title: Hexo-gitalk-Pushover 实现消息推送
author: zranbo
abbrlink: 609f08f9
tags:
  - Hexo
categories: []
date: 2019-05-28 23:10:00
---
<b>前言</b>

用Hexo搭建了这个博客，并且找到了一个利用 GitHub Issue 实现评论功能的插件 gitalk

{% githubCard user:gitalk repo:gitalk %}

从而实现了网站的评论功能。但网站的评论不同于微信等即时通讯的消息，别人的评论有可能你没法马上看到。所以有个想法：怎么样能在别人评论完之后我马上就能知道呢？

考虑到消息推送的时效，想到了 iPhone 的消息推送。这样每当有评论留言可以很快地收到一个消息提醒自己。但 iPhone 的消息推送还是很复杂的，首先要有一个上架 AppStore 的应用。但幸好已经有了这样的应用，这里我就用了 [Pushover](https://itunes.apple.com/cn/app/pushover-notifications/id506088175) 替我进行消息的推送。

<b>配置Pushover</b>

AppStore 上的 Pushover 是付费应用，价格 ¥30.00

购买后可以申请一个账户，然后注册该设备，之后你会得到一个`User Key`
![](https://zranbo.oss-cn-beijing.aliyuncs.com/blog/pushover_key_demo.jpg)

然后从浏览器进入 Pushover 界面 <a href="https://pushover.net" target="_blank">https://pushover.net</a> 从右上角登录刚刚注册的账户

如果之前的都已经做了，那么在 Your Devices 部分应该可以看到你刚刚的设备了。

在底下 Your Applications 创建一个应用
![](https://zranbo.oss-cn-beijing.aliyuncs.com/blog/pushover_application.jpg)

填写好这些信息后，你会获得一个 `API Token` 至此有关 Pushover 的配置就完成了，有了上面的 `User Key` 和刚刚拿到的 `API Token` 我们就能自定义的给刚刚注册的手机发送推送通知了。

<b>Push Demo</b>

首先我们先试一下这个消息推送

消息推送采用的是 Restapi 风格的接口。地址是 <a href="https://api.pushover.net/1/messages.json" target="_blank">https://api.pushover.net/1/messages.json</a>

具体的接口文档参阅 [Pushover Message API](https://pushover.net/api)

如何做到评论之后马上就能推送消息呢？这里我们用 GitHub 的 Webhook ，当有新的评论的时候，就触发这个 Webhook 。我们要做的就只需要找一台拥有公网 ip 的服务器主机，搭建一个简单的 Web 服务。

tool.py
```python
import requests

class PushOver(object):
    def __init__(self, token, user):
        self.api = "https://api.pushover.net/1/messages.json"

        self.rdata = {
            "token": token,
            "user": user,
            "message": "",
            "url": ""
        }

    def loads(self, dictdata):
        title = dictdata['issue']['title']
        user = dictdata['comment']['user']['login']
        comment = dictdata['comment']['body']
        url = dictdata['issue']['body'].split(' ')[0]

        msg = "{t}\n{u}:{c}".format(t=title, u=user, c=comment)

        self.rdata['message'] = msg
        self.rdata['url'] = url

    def send(self):
        try:
            # print(self.rdata)
            requests.post(self.api, data=self.rdata)
        except Exception as e:
            raise e

pushover = PushOver([:API Token], [:User Key])
```
pushover.py
```python
import json

from flask import Flask, jsonify, redirect, request
from tool import pushover

app = Flask(__name__)

@app.route('/webhook', methods=['POST'])
def webhook():
    req = json.loads(request.data)
    action = req['action']
    if action == 'created':
        pushover.loads(req)
        try:
            pushover.send()
            return 'OK'
        except Exception:
            return 'ERROR'
    else:
        return 'PASS'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80)
```

将这两个脚本放到服务器上，安装 gunicorn 启动脚本运行 Web 服务

`gunicorn -w 4 -b 0.0.0.0:80 webhook:app`

<b>配置 Webhook</b>

打开 gitalk 配置的评论仓库，在 Webhooks 设置中添加一个 webhook

![](https://zranbo.oss-cn-beijing.aliyuncs.com/blog/webhook_demo.jpg)

Payload URL 中填写 `http://ip:port/webhook` 如果你的服务器提供商允许使用 `80` 端口可以不写 `:80` 
保存之后就可以试一下评论看有没有消息推送。

![](https://zranbo.oss-cn-beijing.aliyuncs.com/blog/pushover_msg_demo.jpg)

如果没有，可以在 Github Webhook 页面看到调用 wenhook 的记录，排查错误。
![](https://zranbo.oss-cn-beijing.aliyuncs.com/blog/webhook_recent.jpg)