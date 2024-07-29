---
title: 服务端认证实现-JWT
date: 2023-03-31 17:47:40
tags: 
  - 软件工程
---
## JWT概述：
- 全称：Json Web Token
- 是一种规范，这个规范允许我们使用JWT在两个组织之间传递安全可靠的信息；
## JWT实现方式：
### 实现方式一：JWS

- Json Web Signature
#### 结构组成：
```json
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

- Header(头部)
   - 头部用于保存JWT的基本信息，如其类型和签名所用的算法，JSON内容要经过Base64编码生成字符串成为Header.
```
jwt的头部承载两部分信息：
声明类型，这里是jwt
声明加密的算法 通常直接使用 HMAC SHA256
完整的头部就像下面这样的JSON：

{
  'typ': 'JWT',
  'alg': 'HS256'
}
然后将头部进行base64加密（该加密是可以对称解密的),构成了第一部分.

eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
```

- Payload（载荷）
   - iss: 该JWT的签发者
   - sub: 该JWT所面向的用户
   - aud: 接收该JWT的一方
   - exp(expires): 什么时候过期，这里是一个Unix时间戳
   - iat(issued at): 在什么时候签发的
   - jti: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击。
   - 其他信息可以按需补充。 JSON内容要经Base64 编码生成字符串成为PayLoad。
```
载荷就是存放有效信息的地方。这个名字像是特指飞机上承载的货品，这些有效信息包含三个部分

标准中注册的声明
公共的声明
私有的声明
标准中注册的声明 (建议但不强制使用) ：

公共的声明 ： 公共的声明可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息.但不建议添加敏感信息，因为该部分在客户端可解密.

私有的声明 ： 私有声明是提供者和消费者所共同定义的声明，一般不建议存放敏感信息，因为base64是对称解密的，意味着该部分信息可以归类为明文信息。

定义一个payload:

{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
然后将其进行base64加密，得到Jwt的第二部分。

eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9
```

- signature(签名)
   - header和payload按照header中申明的加密方式，使用secret进行加密，生成签名，
   - 主要的目的是保证数据在传输过程中不被修改，验证数据的完整性，
   - 由于仅采用Base64对消息内容进行编码，因此不保证数据的不可泄露性，所以不适合适合传递敏感数据。
```
jwt的第三部分是一个签证信息，这个签证信息由三部分组成：

header (base64后的)
payload (base64后的)
secret
这个部分需要base64加密后的header和base64加密后的payload使用.连接组成的字符串，然后通过header中声明的加密方式进行加盐secret组合加密，然后就构成了jwt的第三部分。

// javascript
var encodedString = base64UrlEncode(header) + '.' + base64UrlEncode(payload);

var signature = HMACSHA256(encodedString, 'secret'); // TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
将这三部分用.连接成一个完整的字符串,构成了最终的jwt:

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
注意：secret是保存在服务器端的，jwt的签发生成也是在服务器端的，secret就是用来进行jwt的签发和jwt的验证，所以，它就是你服务端的私钥，在任何场景都不应该流露出去。一旦客户端得知这个secret, 那就意味着客户端是可以自我签发jwt了。
```
### 实现方式二：JWE
相对于JWS，JWE则同时保证了安全性与数据完整性。 

- JWE由五部分组成：
   - JOSE含义与JWS头部相同。
   - 生成一个随机的Content Encryption Key （CEK）。
   - 使用RSAES-OAEP 加密算法，用公钥加密CEK，生成JWE Encrypted Key。
   - 生成JWE初始化向量。
   - 使用AES GCM加密算法对明文部分进行加密生成密文Ciphertext,算法会随之生成一个128位的认证标记Authentication Tag。 对五个部分分别进行base64编码。
- JWE的计算过程相对繁琐，不够轻量级，因此适合与数据传输而非token认证，但该协议也足够安全可靠，用简短字符串描述了传输内容，兼顾数据的安全性与完整性。
## Python中使用JWT
### itsdangerous库

- JSONWebSignatureSerializer
- TimedJSONWebSignatureSerializer （可设置有效期）
### pyjwt库
[官方文档链接](https://pyjwt.readthedocs.io/en/latest/)
#### 使用方式
```
# 安装
pip install pyjwt

# demo
import pyjwt

encoded_jwt = jwt.encode({'some': 'payload'}, 'secret', algorithm='HS256')
print(encoded_jwt)  # 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzb21lIjoicGF5bG9hZCJ9.4twFt5NiznN84AWoo1d7KO1T_yoc0Z6XOpOVswacPZg'
jwt.decode(encoded_jwt, 'secret', algorithms=['HS256'])  # {'some': 'payload'}
```

---

## Django（DRF）中使用JWT

- 安装
```bash
pip install djangorestframework-jwt
```

- 配置
```
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_jwt.authentication.JSONWebTokenAuthentication',
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.BasicAuthentication',
    ),
}

JWT_AUTH = {
    'JWT_EXPIRATION_DELTA': datetime.timedelta(days=1), # WT_EXPIRATION_DELTA 指明token的有效期
}
```

---

- 手动创建token的方法
```
from rest_framework_jwt.settings import api_settings


jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER

payload = jwt_payload_handler(user)
token = jwt_encode_handler(payload)

user.token = token
```

---

- DRF中的具体实现
> 修改CreateUserSerializer序列化器，在create方法中增加手动创建token的方法

```
from rest_framework_jwt.settings import api_settings

class CreateUserSerializer(serializers.ModelSerializer):
    """
    创建用户序列化器
    """
    ...
    token = serializers.CharField(label='登录状态token', read_only=True)  # 增加token字段

    class Meta：
        ...
        fields = ('id', 'username', 'password', 'password2', 'sms_code', 'mobile', 'allow', 'token')  # 增加token
        ...

    def create(self, validated_data):
        """
        创建用户
        """
        # 移除数据库模型类中不存在的属性
        del validated_data['password2']
        del validated_data['sms_code']
        del validated_data['allow']
        user = super().create(validated_data)

        # 调用django的认证系统加密密码
        user.set_password(validated_data['password'])
        user.save()

        # 补充生成记录登录状态的token
        jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
        jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER
        payload = jwt_payload_handler(user)
        token = jwt_encode_handler(payload)
        user.token = token

        return user
```
## 常见问题的解决思路
### 如何解决刷新问题？
设置有效期，但有效期不宜过长，需要刷新

- 思路：
   - 手机号+验证码（或帐号+密码）验证后颁发接口调用token与refresh_token（刷新token）
   - Token 有效期为2小时，在调用接口时携带，每2小时刷新一次
   - 提供refresh_token，refresh_token 有效期14天
   - 在接口调用token过期后凭借refresh_token 获取新token
   - 未携带token 、错误的token或接口调用token过期，返回401状态码
   - refresh_token 过期返回403状态码，前端在使用refresh_token请求新token时遇到403状态码则进入用户登录界面从新认证。
   - token的携带方式是在请求头中使用如下格式：
- 实现：
   - 注册或登录获取token
   - jwt验证用户身份
   - 用户必须登录装饰器
   - 更新token接口
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzb21lIjoicGF5bG9hZCJ9.4twFt5NiznN84AWoo1d7KO1T_yoc0Z6XOpOVswacPZg
  
# Bearer前缀与token中间有一个空格
```

### JWT禁用问题
token颁发给用户后，在有效期内服务端都会认可，但是如果在token的有效期内需要让token失效，该怎么办？

- 此问题的应用场景：
   - 用户修改密码，需要颁发新的token，禁用还在有效期内的老token
   - 后台封禁用户
- 解决办法：
   - 在redis中使用set类型保存新生成的token
- redis记录设置有效期的时长是一个token的有效期，保证旧token过期后，redis的记录也能自动清除，不占用空间。
- 使用set保存新token的原因是，考虑到用户可能在旧token的有效期内，在其他多个设备进行了登录，需要生成多个新token，这些新token都要保存下来，既保证新token都能正常登录，又能保证旧token被禁用
```
key = 'user:{}:token'.format(user_id)
pl = redis_client.pipeline()
pl.sadd(key, new_token)
pl.expire(key, token有效期)
pl.execute()
```

   - 客户端使用token进行请求时，如果验证token通过，则从redis中判断是否存在该用户的user:{}:token记录：
      - 若不存在记录，放行，进入视图进行业务处理
      - 若存在，则对比本次请求的token是否在redis保存的set中：
         - 若存在，则放行
         - 若不在set的数值中，则返回403状态码，不再处理业务逻辑
```
key = 'user:{}:token'.format(user_id)
valid_tokens = redis_client.smembers(key, token)
if valid_tokens and token not in valid_tokens:
    return {'message': 'Invalid token'.}, 403
```

