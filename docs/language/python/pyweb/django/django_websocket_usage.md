---
title: django中使用websocket
date: 2021-02-27 13:57:01
tags: 
  - Python
  - Django
---

# django中使用websocket


#### 安装

```shell
pip install -U channels
```
#### 把channels加入到INSTALLED_APPS

```python
INSTALLED_APPS = (
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.sites',
    ...
    'channels',
)
```
#### 在settings.py文件的同级目录下创建routing.py文件

```python
from channels.auth import AuthMiddlewareStack
from channels.routing import ProtocolTypeRouter, URLRouter, ChannelNameRouter
import public.routing
from public import consumers

application = ProtocolTypeRouter({
    # (http->django views is added by default)
    'websocket': AuthMiddlewareStack(
        URLRouter(
            public.routing.websocket_urlpatterns
        )
    ),
    "channel": ChannelNameRouter({
        "service-detection": consumers.ServiceConsumer,
    }),
})
```
#### 在settings.py文件中添加：

```python
ASGI_APPLICATION = "total_system.routing.application"
```
#### 在setting.py文件中添加使用redis相关配置
安装channels_redis

```shell
pip install channels_redis
```

```python
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',  # 使用redis缓存
        'CONFIG': {"hosts": ["redis://10.102.100.28:6379/0"], },
    },
}
```
#### app里添加consumers.py文件

```python
from channels.generic.websocket import AsyncWebsocketConsumer
import json
import logging

logger = logging.getLogger("django")


class ServiceConsumer(AsyncWebsocketConsumer):

    async def connect(self):
        self.project_pk = self.scope['url_route']['kwargs']['project_pk']
        self.room_group_name = '%s' % self.project_pk
        print('***********open************')
        print("channel_name", self.channel_name)

        await self.channel_layer.group_add(
            self.room_group_name,
            self.channel_name
        )

        await self.accept()
        # await self.send(text_data=json.dumps({
        #     'text': "success"
        # }))

    async def disconnect(self, close_code):
        print("room_group_name", self.room_group_name)
        print('websocket断开')

        await self.channel_layer.group_discard(
            self.room_group_name,
            self.channel_name
        )

    # Receive message from WebSocket
    async def receive(self, text_data):
        print(text_data)
        text_data_json = json.loads(text_data)

        message = text_data_json['text']
        print(message)
        # Send message to room group
        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'user_message',
                'text': message
            }
        )

    # Receive message from room group
    async def user_message(self, event):
        # todo: get real user from service detection api

        message = event['text']
        # Send message to WebSocket
        # await self.send(text_data=json.dumps(message))
        await self.send(text_data=json.dumps({
            'text': message
        }))
```
#### app里添加routing.py文件

```python
from django.conf.urls import url
from public import consumers

websocket_urlpatterns = [
    url(r'^project/(?P<project_pk>[^/]+)/$', consumers.ServiceConsumer),
    # url(r'^pro/(?P<project_pk>[^/]+)/$', consumers.ProjectConsumer),
]
```
#### 在utils里创建channels_msg.py

```python
import time
import os

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "total_system.settings")

from channels.layers import get_channel_layer
from asgiref.sync import async_to_sync

channel_layer = get_channel_layer()


# channel_name 是保存时 self.room_group_name 的名字


def send_channel_msg(channel_name, msg):
    async_to_sync(channel_layer.group_send)(channel_name, {'type': 'user_message', 'text': msg})


if __name__ == '__main__':
    while True:
        send_channel_msg("ct_01", {"sum": 9.9, "name": "姓名", "num": 4, "all_money": 4412})
        print(time.time())
        time.sleep(5)
```
#### 使用websocket向前端推送消息

```python
from utils.channels_msg import send_channel_msg

send_channel_msg("ct_01", {"is_stop": 1})
```


#### 在settings.py文件同级目录新建asgi.py

```python
import os
import django
from channels.routing import get_default_application

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "total_system.settings")
django.setup()
application = get_default_application()
```

#### 前端代码参考

```python
<!-- chat/templates/chat/room.html -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8"/>
    <title>Chat Room</title>
</head>
<body>
    <textarea id="chat-log" cols="100" rows="20"></textarea><br/>
    <input id="chat-message-input" type="text" size="100"/><br/>
    <input id="chat-message-submit" type="button" value="Send"/>
</body>
<script>
    var roomName = {{ room_name_json }};

    var chatSocket = new WebSocket(
        'ws://' + window.location.host +
        '/ws/chat/' + roomName + '/');

    chatSocket.onmessage = function(e) {
        var data = JSON.parse(e.data);
        var message = data['message'];
        document.querySelector('#chat-log').value += (message + '\n');
    };

    chatSocket.onclose = function(e) {
        console.error('Chat socket closed unexpectedly');
    };

    document.querySelector('#chat-message-input').focus();
    document.querySelector('#chat-message-input').onkeyup = function(e) {
        if (e.keyCode === 13) {  // enter, return
            document.querySelector('#chat-message-submit').click();
        }
    };

    document.querySelector('#chat-message-submit').onclick = function(e) {
        var messageInputDom = document.querySelector('#chat-message-input');
        var message = messageInputDom.value;
        chatSocket.send(JSON.stringify({
            'message': message
        }));

        messageInputDom.value = '';
    };
</script>
</html>
```

#### 后端websocket测试客户端

```python
import asyncio
import websockets
import json
import time


async def test_ws_quote():
    async with websockets.connect('ws://10.10.10.10:8005/project/plc_01/') as websocket:
        req = {'text': {"protocol": "history_req", 'code': 'XAGODS', 'type': 'MINUTE', 'start_pos': '0', 'pos_num': '10'}}
        await websocket.send(json.dumps(req))

        while True:
            try:
                quote = await websocket.recv()
            except:
                print("服务器已关闭···")
                break
            else:
                # time.sleep(5)
                quote = json.loads(quote)
                print(quote)


asyncio.get_event_loop().run_until_complete(test_ws_quote())
```


部署时启动websocket

```shell
daphne -b 10.10.100.10 -p 8005 total_system.asgi:application -v2
```



