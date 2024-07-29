---
title: gRPC的学习
date: 2021-10-08 22:47:34
tags: 
  - 软件工程
---


## gRPC概述

- gRPC是google公司开源的高性能RPC框架
- 支持多语言：
  - gRPC原生使用C/Java/Go进行了实现，而C语言实现的版本进行封装后又支持C++/C#/Node/objC/Python/Ruby/PHP等开发也语言
- 支持多平台：
  - 支持Linux/Android/IOS/MacOS/Windows
- gRPC的消息协议使用Protocol Buffers协议机制（proto3）序列化
- 传输使用http/2标准，支持双向流和连接多路复用



## gRPC架构图

![image-20211013230217469](https://gitee.com/roninsswang/upload-picture/raw/master/blogimg/image-20211013230217469.png)



## gRPC的使用流程

1. 使用Protocol Buffers( proto3)的IDL接口定义语言定义接口服务，编写在文本文件（以 .proto为后缀名）中
2. 使用protobuf编译器生成服务器和客户端使用的stub代码
3. 编写补充服务器和客户端逻辑代码



---

# Protobuf的使用总结

## 概述

>  protobuf全称：Protocol Buffers

- protobuf是google于2008年开源的可扩展序列化结构数据格式
- 是一种与语言无关，平台无关的可扩展机制，用于序列化结构化数据
- 使用protobuf可以自定义数据结构，对数据结构进行一次描述，即可利用各种不同语言或从各种不同数据流中对结构化数据进行轻松读写
- 目前protobuf有两个版本，在grpc中推荐使用proto3版本

### 优点：

- 1:序列化后体积相比Json和XML很小，适合网络传输
- 2:支持跨平台多语言
- 3:消息格式升级和兼容性还不错
- 4:序列化反序列化速度很快，快于Json的处理速速

### 缺点：

- 功能简单，无法用来表示复杂的概念。
- protobuf是二进制数据格式，需要编码和解码，数据本身不具有可读性，因此只能反序列化之后得到真正可读的数据。而XML已经是行业的标准工具，且具备自我解释性，可以被人们直接读取编辑。

---

## protobuf的安装

### linux安装

下面列出centOS的安装方式：

```shell
# 安装依赖
yum install autoconf automake libtool curl make g++ unzip libffi-dev  glibc-headers gcc-c++ -y

# 下载
git clone https://github.com/protocolbuffers/protobuf.git

# 安装
unzip protobuf.zip
cd protobuf
./autogen.sh
./configure
make && make install
ldconfig        # 刷新共享库

# 测试
protoc --version
```

### mac安装

mac推荐使用homebrew安装：

```shell
# 安装
brew install protobuf
brew install automake
brew install libtool

# 测试
protoc --version
```

### win安装

下载地址: https://github.com/google/protobuf/releases

下载win版本后，配置文件中的bin目录到Path环境变量下即可。

### python语言下的安装和使用

```shell
# 安装protobuf编译器和grpc库
pip install grpcio-tools

# 编译生成代码
python -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. reco.proto

-I                 表示搜索proto文件中被导入文件的目录
--python_out       表示保存生成Python文件的目录，生成的文件中包含接口定义中的数据类型
--grpc_python_out  表示保存生成Python文件的目录，生成的文件中包含接口定义中的服务类型

# 执行上述命令，会自动生成如下两个rpc调用辅助代码模块
reco_pb2.py        保存根据接口定义文件中的数据类型生成的python类
reco_pb2_grpc.py   保存根据接口定义文件中的服务方法类型生成的python调用RPC方法
```

> 

---



## 相关语法

### 版本

```protobuf
syntax = "proto3"; //在grpc中官方推荐使用此版本
//或
syntax = "proto2";
```

### Package包

- 可以使用package来防止命名冲突。
- 可选
- 对于Python而言，`package`会被忽略处理，因为Python中的包是以文件目录来定义的。

```protobuf
package foo.bar;
message Open{....}

message Foo {
  ...
  foo.bar.Open open = 1;
  ...
}
```

### 导入

- 可以导入其它文件消息等（类似于python的import）

```protobuf
import “demoproject/demo_protos.proto”;
```

### 定义消息和服务

- message:定义数据
- service: 定义grpc的方法

### 注释

```protobuf
// 单行注释
//
//
//

/* 
多行注释 
*/
```

---



### 数据类型

#### 基本数据类型

| .proto   | 说明                                                         | Python   |
| :------- | :----------------------------------------------------------- | :------- |
| double   |                                                              | float    |
| float    |                                                              | float    |
| int32    | 使用变长编码，对负数编码效率低， 如果你的变量可能是负数，可以使用sint32 | int      |
| int64    | 使用变长编码，对负数编码效率低，如果你的变量可能是负数，可以使用sint64 | int/long |
| uint32   | 使用变长编码                                                 | int/long |
| uint64   | 使用变长编码                                                 | int/long |
| sint32   | 使用变长编码，带符号的int类型，对负数编码比int32高效         | int      |
| sint64   | 使用变长编码，带符号的int类型，对负数编码比int64高效         | int/long |
| fixed32  | 4字节编码， 如果变量经常大于2^{28} 的话，会比uint32高效      | int      |
| fixed64  | 8字节编码， 如果变量经常大于2^{56} 的话，会比uint64高效      | int/long |
| sfixed32 | 4字节编码                                                    | int      |
| sfixed64 | 8字节编码                                                    | int/long |
| bool     |                                                              | bool     |
| string   | 必须包含utf-8编码或者7-bit ASCII text                        | str      |
| bytes    | 任意的字节序列                                               | str      |

#### 枚举类型

- 枚举定义在一个消息内部或外部都是可以的，如果定义在message内部，而其他message又想使用，可以通过MessageType.EnumType的方式引用
- 定义枚举时，要保证第一个枚举值必须是0，枚举值不能重复
- 可以使用 option allow_alias = true 选项来开启别名。
- 枚举值的范围是32-bit integer，但因为枚举值使用变长编码，所以不推荐使用负数作为枚举值

```protobuf
enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
}
Corpus corpus = 4;


enum EnumAllowingAlias {
    option allow_alias = true;
    UNKNOWN = 0;
    STARTED = 1;
    RUNNING = 1;
}
```

#### 消息类型

> 消息定义中的每个字段都有唯一的编号。**这些字段编号用于以消息二进制格式标识字段，并且在使用消息类型后不应更改。** 请注意，**1到15范围内的字段编号需要一个字节进行编码，包括字段编号和字段类型**。**16到2047范围内的字段编号占用两个字节**。因此，您应该为非常频繁出现的消息元素保留数字1到15。请记住为将来可能添加的常用元素留出一些空间。
>
> 最小的标识号可以从1开始，最大到2^29 - 1,或 536,870,911。不可以使用其中的[19000－19999]的标识号， Protobuf协议实现中对这些进行了预留。如果非要在.proto文件中使用这些预留标识号，编译时就会报警。同样你也不能使用早期保留的标识号。

- 指定字段规则
  - singular：格式良好的消息可以包含该字段中的零个或一个（但不超过一个）
  - repeated：此字段可以在格式良好的消息中重复任意次数（包括零）。将保留重复值的顺序。对应Python的列表。

```protobuf
message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
}
```

- 添加更多消息类型
  - 可以在单个.proto文件中定义多个消息类型。

```protobuf
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}

message SearchResponse {
 ...
}
```

- 保留字段

  > 如果通过完全删除字段或将其注释来更新消息类型，则未来用户可以在对类型进行自己的更新时重用字段编号。如果以后加载相同的旧版本，这可能会导致严重问题，包括数据损坏，隐私错误等。确保不会发生这种情况的一种方法是**指定已删除字段的字段编号（或名称）reserved**。如果将来的任何用户尝试使用这些字段标识符，protobuf编译器将会报错。

  - 保留变量不被使用

    ```protobuf
    message Foo {
      reserved 2, 15, 9 to 11;
      reserved "foo", "bar";
    }
    ```

- 默认值

  > 解析消息时，如果编码消息不包含特定的单数元素，则可以将该字段置为默认值

  ```
  字符串，默认值为空字符串。
  字节，默认值为空字节。
  bools，默认值为false。
  数字类型，默认值为零。
  枚举，默认值是第一个定义的枚举值，该值必须为0。
  消息字段，未设置该字段。它的确切值取决于语言。
  重复字段的默认值为空（通常是相应语言的空列表）。
  ```

- 嵌套类型

  - 可以在其他消息类型中定义、使用消息类型

  ```protobuf
  message SearchResponse {
    message Result {
      string url = 1;
      string title = 2;
      repeated string snippets = 3;
    }
    repeated Result results = 1;
  }
  
  //如果要在其父消息类型之外重用此消息类型，使用
  SearchResponse.Result
  ```

- map映射

  ```protobuf
  //在数据定义中创建关联映射
  map< key_type, value_type> map_field = N ;
  
  /*
  key_type   可以是任何整数或字符串类型,但不能用枚举类型
  value_type 可以是除map映射类型外的任何类型。
  */
  
  //创建项目映射，其中每条Project消息都与字符串键相关联
  map<string, Project> projects = 3 ;
  /*
  map 的字段可以是repeated
  
  序列化后的顺序和map迭代器的顺序是不确定的
  
  当为.proto文件产生生成文本格式的时候，map会按照key 的顺序排序，数值化的key会按照数值排序。
  
  从序列化中解析或者融合时，如果有重复的key则后一个key不会被使用，当从文本格式中解析map时，如果存在重复的key，则解析可能会失败。
  
  如果为映射字段提供键但没有值，则字段序列化时的行为取决于语言。在Python中，使用类型的默认值。
  */
  ```

- oneof

  - 如果你的消息中有很多可选字段， 并且同时至多一个字段会被设置， 你可以加强这个行为，使用oneof特性节省内存。

    ```protobuf
    message SampleMessage {
      oneof test_oneof {
        string name = 4;
        SubMessage sub_message = 9;
      }
    }
    //注意：不能使用repeated 关键字。
    ```

    

### 定义服务

- 使用service定义RPC服务

- 一个service可以定义多个方法

  ```protobuf
  message HelloRequest {
    string greeting = 1;
  }
  
  message HelloResponse {
    string reply = 1;
  }
  
  service HelloService {
    rpc SayHello (HelloRequest) returns (HelloResponse) {}
  }
  //
  ```

  

