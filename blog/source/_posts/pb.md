---
title: pb应知应会   
date: 2022-01-03 15:21:49   
tags: pb   
categories:
  - go
  - grpc
  - pb   
description: pb作为grpc的IDL语言，其自身具有极大的优点，但也存在一些使用上的问题；本文将介绍pb的规范、管理方式、以及优点和缺点。
---
## pb简介
> Protobuf是Protocol Buffers的简称，它是Google公司开发的一种数据描述语言，并于2008年对外开源。Protobuf刚开源时的定位类似于XML、JSON等数据描述语言，通过附带工具生成代码并实现将结构化数据序列化的功能。

说的好抽象啊，我们来具象化一下：
> 写过`thrift`的朋友，可能立马反应过来了，这东西也是用来定义消息以及消息是如何通信的嘛。都是为RPC服务，<u>我们知道RPC的作用，就是让远程过程调用，看起来像是本地调用一样；但实际上，是请求远端的服务，既然是请求远端的服务，我们肯定要知道对方的<span style="color: orange">服务名(service)</span>、<span style="color: orange">方法名(func)</span>、<span style="color: orange">消息结构(message/struct)</span>吖，不然我找谁去请求，我怎么去解析数据</u>。proto文件就是干这么一件事，所以proto也是一种描述性语言嘛。

### pb的好处
> `Protocol Buffers`是一种轻便高效的结构化数据存储格式，可以用于结构化数据的序列化，很适合做数据存储或 RPC 数据交换格式。它可用于通讯协议、数据存储等领域的语言无关、平台无关、可扩展的序列化结构数据格式。

听起来，设计rpc的通信协议，好像很简单，也没什么嘛，不就是简单的协议（语法、语义、时序）定义嘛！！！其实不然，这其中，需要考虑的问题很多，比如：<u style="color: orange">数据发送方如何序列化传输数据</u>、<u style="color: orange">数据接受方需要如何接收并反序列化数据</u>、<u style="color: orange">数据的传输效率如何提高</u>、<u style="color: orange">各种语言如何与pb语义对应上</u>等等。

### pb编码方式
> 在XML或JSON等数据描述语言中，一般通过成员的名字来绑定对应的数据。但是Protobuf编码却是通过成员的唯一编号来绑定对应的数据，因此Protobuf编码后数据的体积会比较小，但是也非常不便于人类查阅。

### pb的使用
- `.proto`文件的书写
- 使用IDL编译器编译成对应语言的代码

## [pb书写规范](https://developers.google.com/protocol-buffers/docs/style)
- `pb`文件名为小写+下划线形式，文件后缀以`.proto`结尾
- 保证每行80字符左右；请使用2个空格缩进
- 除结构定义之外的语句均以分号结尾
- 包名必须小写, 并应与目录层次结构相对应. 例如: test/pb/api.proto 包名应该为test.pb
- `message`结构体命名采用驼峰命名方式，字段命名采用小写字母加下划线分隔方式
- `enums`类型名采用驼峰命名方式，字段命名采用大写字母加下划线分隔方式
- `service`与`rpc`方法名统一采用驼峰式命名

### 案例说明
```protobuf
// 指定proto的版本，默认proto2；proto3对语言进行了提炼简化，所有成员均采用类似Go语言中的零值初始化（不再支持自定义默认值）
// 因此消息成员也不再需要支持required特性。
syntax = "proto3";

// 定义包名(import path)，防止message重名
package main;

// 导入外部pb
import "google/protobuf/timestamp.proto";

// 添加可选项
option go_package = "github.com/protocolbuffers/protobuf/examples/go/tutorialpb";

enum PersonType {
  WHITE = 0;
  BLACK = 1;
  YELLOW = 2;
}

message Person {
  PersonType type = 1;
  string name = 2;
  // Unique ID number for this person.
  int32 id = 3;
  string email = 4;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    string number = 1 [(gogoproto.jsontag) = "number", json_name = "number" ];
    PhoneType type = 2 [(gogoproto.jsontag) = "type", json_name = "type" ];
  }

  repeated PhoneNumber phones = 5;

  google.protobuf.Timestamp last_updated = 6;
}

// 定义message（可定义多个）
message EntranceReq {
  // 定义字段： type fieldName = fieldNumber; 
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}

// EntranceResp 入口返回结果
message EntranceResp {
    // 图标
    string icon = 1;
    // 名称
    string name = 2;
}

// 定义服务和方法
service TestService {
    // 活动入口
    rpc TestEntrance (EntranceReq) returns (EntranceResp);
}
```

### 划重点
#### 字段的`fieldNumber`
> 这个并不是`fieldName`的值，只是一个标号（`tag`），意味着：往后见到`fieldNumber`就代表是`fieldName`；换句话说，字段叫啥名在protobuf中并不重要, 因为在传输的时候，二进制数据流里用的是`fieldNumber`而不是`fieldName`；所以`fieldNumber`一旦被使用, 终生这个编号都不要改变，否则很可能引发线上故障，这也是为什么我们说pb字段的`fieldNumber`只能追加，不能修改，或者插入的原因。
> `fieldNumber`的取值范围是1~2^29-1. 而常用的`fieldNumber`范围是: 1-15(只用1个byte编码),  16-2047(采用2个byte编码). 所以为了节省编码后的长度, 经常使用的一些字段名(如:name, id等), 分配1-15的`fieldNumber`.

#### 字段的定义
  * singular单数字段: protobuf的默认字段规则, 就是说这个字段只能出现0或者1次.
  * repeated重复字段: 代表该字段是一个数组或者list. 数组里面可以有任意数量的元素. 如果有多个元素, 元素的顺序会被保留.

```protobuf
message Shop {
  int32 id = 1;
  string name = 2;
  // 商店会有多个服务员
  repeated string staff = 3;
}
```

#### 保留字段
> 保留字段的意思就是, 这些字段保留下来, 后续在protobuf中,不能再次使用了.(即: 防止字段名一样, 但是字段含义不同)

案例说明
```protobuf
syntax = "proto3";

message Person {
	reserved 2, 3 to 7; 	// 保留这几个fieldNumber
    reserved "foo", "bar"; 	// 保留这几个字段名
}
```

举个例子解释下为啥要保留字段
```protobuf
syntax = "proto3";

// 一开始的需求, UserInfo绑定的是微信的账号和密码
message UserInfo {
	int32 Id = 1;
    stirng name = 2;
    
    string wechat_account = 3;
    string wechat_pwd = 4;
}

// 现在需求变了,要求用户信息绑定QQ账号密码
// 此时我删除了 wechat_account wechat_pwd两个字段, 并添加QQ_account, QQ_pwd
// 同时, 之前分配给wechat_account和wechat_pwd的fieldNumber 3 4, 又再一次分配给了 QQ_account和QQ_pwd.
message UserInfo {
	int32 Id = 1;
    stirng name = 2;
    
    string QQ_account = 3;
    string QQ_pwd = 4;
}

// 想想会有什么问题?
// 别想了,我直接说了, 假如server端修改了protobuf的定义,但是client端还没有更新.
// 此时, 客户端传给server微信的账号/密码, 服务端作为QQ的账号密码去验证,肯定是错的.

// 所以呢? 所以修改(如删掉)的字段和对应的fieldNumber都应该保留, 后续都不能在使用了.
```

#### `enum` 枚举类型
> 枚举可以定义在message里面,也可以定义在外面(便于复用)；在另一个message类型中,可以通过UserInfo.Gender, 使用枚举类型. reserved同样也可以适用于枚举类型.

案例说明
```protobuf
message UserInfo {

	Gender gender = 1; // 使用Gender枚举类型
    
    // 定义枚举类型
	enum Gender {
    	FEMAIL = 0; // 必须从0开始
        MAIL = 1;
    }
}
```

#### `message`类型
```protobuf
syntax = "proto3"; 

message Person {
    int32 id = 1; 
    string name = 2;

    Date birthday = 3; // 使用message类型作为字段的type
}

// 定义消息类型Date:生日
message Date {
    int32 year = 1;
    int32 mounth = 2;
    int32 day = 3;
}
```

Tips：一般来说,不相关的消息, 每个message,创建一个proto文件. 如果需要用到其他.proto文件中定义的message, 要通过import进行引入. 编译器会在--proto_path参数指定的路径下寻找相应的需要导入的proto文件. 不写默认在当前目录寻找.

#### `package`包名
> 给一个.proto文件指定package, 是为了避免和其他的.proto文件的message名称冲突.

案例说明
```protobuf
// bar.proto
package foo.bar;
message Open { ... }
```
后面可以使用该.proto文件的包名去使用message Open
```protobuf
// foo.proto

import "bar.proto"
message Foo {
  ...
  foo.bar.Open open = 1;
  ...
}
```
当被IDL编译器翻译成GO语言后, Go代码的包名, 默认就是.proto文件的pacakge名称, 除非在.proto文件中显示的用go_pacakge指定IDL编译后的Go文件的import path.

#### import的搜寻路径是？
> 搜寻路径由protoc -I或者protoc --proto_path指定. 所以, import 要和 protoc -I/--proto_path 命令配合好.

### 踩坑笔记
- 任何地方的命名，都不要使用关键字，会出问题
- java_package中，包含关键字（如：public / interface）
- pb文件中，混入奇怪的不可打印字符，或者是混入无用的、语法不对的字符
- enum和message名字不一样就好
- message的名字不要和文件名一样，小心踩坑

## pb管理办法
> pb是好用，可是如何优雅的管理起来，是个头疼的问题；微服务化后，多人协作开发上，就很容易出问题。不禁引人发问：proto这个IDL的代码到底应该放在哪里，该怎么管理？这里简单讨论下

能够想到的几种方式如下：

- 代码仓库
- 独立仓库
- 集中仓库
- 镜像仓库
- 组合方式

### 代码仓库
直接将项目所依赖的所有proto文件都存放在项目的`proto/`目录下，不经过开发工具的自动拉取和发布：
<div>
  <img src="manage1.png" width="300px" alt="代码仓库">
</div>

- 优点
  * 项目所有依赖的 Proto 都存储在代码仓库下，因此不涉及个人开仓库权限的问题。
  * 多 Proto 的切换开销减少，因为都在代码仓库下，不需要看这看那。
- 缺点
  * 项目所有依赖的 Proto 都存储在代码仓库下，因此所有依赖 Proto 都需要人工的向其它业务组 “要” 来，再放到 proto/ 目录下，人工介入极度麻烦。
  * Proto 升级和变更，经常要重复第一步，沟通成本高。

### 独立仓库
> 独立仓库存储是我们最早采取的方式，也就是每个服务对应配套一个 Proto 仓库

<div>
  <img src="manage2.png" width="400px" alt="独立仓库">
</div>

这个方案的好处就是可以独立管理所有 Proto 仓库，并且权限划分清晰。但最大的优点也是最大的缺点。因为一个服务会依赖多个 Proto 仓库，并且存在跨业务组调用的情况

<div>
  <img src="manage3.png" width="400px" alt="独立仓库">
</div>

如上图所示，svc-user 服务分别依赖了三块 Proto 仓库，分别是自己组的、业务组 A、业务组 B 总共的 6 个 Proto 仓库。

- 优点
    * 使得安全性较高（但 IDL 本身没有太多的秘密）。
    * 按需拉取，不需要关注其余的服务 Proto。
- 缺点
    * 假设你是一个新入职的开发人员，那么你就需要找不同的业务组申请不同的仓库权限，非常麻烦。如果没有批量赋权工具，也没有管理者权限，那么就需要一个个赋权，非常麻烦。
    * 在运行服务的时候，你需要将所有相关联的 Proto 仓库拉取下来，如果没有工具做半自动化的支持，麻烦程度无法忍受。

### 集中仓库
> 集中仓库也是一些公司考虑的方式之一，是按公司或大事业部的维度进行 Proto 代码的存储。这样子只需要拉取一个仓库，就可以获取到所有所需的IDL.

<div>
  <img src="manage4.png" width="400px" alt="集中仓库">
</div>

- 优点
    * 只需要拉取一次Proto仓库就可以轻松把一个服务所需的 IDL 集齐。
    * 仓库权限管理的复杂度下降。
- 缺点
    * 安全性下降，因为其它业务组的IDL也全都 “泄露” 了。
    * 非按需拉取，在查看原始文件时，需要关注一些多余的。

### 镜像仓库
> 自己服务的 Proto 文件存放在代码仓库的 proto 文件中，在本次 feature 提交或发布后，自动同步到镜像仓库去。你所依赖的其他服务 Proto 则直接通过读取集中的镜像仓库的方式获取.

<div>
  <img src="manage5.png" width="500px" alt="镜像仓库">
</div>

这样子的话，通过开发工具的配合，开发人员在开发时就只需要关注自己项目的 Proto 就好了。集中的镜像仓库主要用于构建和部署，大幅度降低了多Proto的关注和切换开销。

### 组合方式
> 单一的方式，或多或少的都存在一些问题，如果采用组合的方式，可以最大程度地发挥作用。例如：独立仓库+集中仓库，对于公共的、需要暴露出去的部分放到集中仓库，不需要暴露出去的就放到独立仓库（例如：部门内部的，可以放到独立仓库，需要跨部门的，可以放到集中仓库），可以一定程度上降低安全性问题。

实际工作中，我们不仅需要考虑proto文件的管理，还需要管理proto编译产物的管理，而这个过程，需要考虑到区分版本的问题，因为测试和上线是两个不同的阶段，不能让测试的版本，被用到了线上。

## 参考文章
- [Protobuf入门（大白话版）](https://juejin.cn/post/6865126893063471112)
- [真是头疼，Proto 代码到底放哪里？](https://mp.weixin.qq.com/s/cBXZjg_R8MLFDJyFtpjVVQ)
