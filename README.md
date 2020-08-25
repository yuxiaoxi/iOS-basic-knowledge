# iOS-basic-knowledge
iOS 基础知识整理

## 网络相关
### 1、Https 和 Http 区别
1. Https 需要向机构申请 CA 证书，极少免费
2. Https 基于 SSL/TSL 进行加密传输，http 是明文传输
3. Http 的端口号是80，https 的端口号是443
4. Https 是加密传输入，所以更加安全

### 2、Https 建立过程
1. 客户端发出连接请求，并带上支持的加密算法列表、TSL 版本号以及随机串 C
2. 服务端返回约定好的加密算法、服务端证书、公钥以及随机串 S
3. 客户端对证书进行校验，并且根据公钥生成前主密钥
4. 客户端利用前主密钥和随机串 C、S 生成会话密钥
5. 客户端将前主密钥发送至服务端
6. 服务端利用自己的私钥进行解密得到主密钥
7. 服务端利用主密钥和随机串 C、S 生成会话密钥
8. 至此客户端和服务端都已经获取到了数据通信的密钥，可以进行数据传输了

### 3、Http 1.x 和 Http 2.0 区别
1. 新的二进制: http 1.x的解析是基于文本，而 2.0是基于二进制，增强了健壮性
2. 多路复用：2.0 可以支持一个连接多个 request,每个 request 有个 id 进行标识
3. header压缩：2.0 使用 encoder 来减少需要传输入的 header 大小，并且通讯双方各自 cache 一份 header fields 表
4. 2.0 支持服务端推送

## 数据库相关

### 1、数据库引擎
常见的数据库引擎有： ISAM、Memory、Innodb、Archive，而不同引擎所支持的功能也不一样，下面是各引擎的功能支持表。
  引擎 | ISAM | Memory | Innodb | Archive
| ------ | ------ | ------ | ------ | ------ |
| 存储大小 | 256TB | RAM 大小 | 64TB | None |
| 支持事务 | No | No | Yes | No |
| 支持全文索引 | Yes | No | No | No |
| 支持数索引 | Yes | Yes | Yes | No |
| 支持哈希索引 | No | Yes | No | No |
| 支持数据缓存 | No | N/A | Yes | No |
| 支持外键 | No | No | Yes | No |

### 2、SQL 语句查询原理

基本流程如下：
sql语句 -> 解析器 -> 预处理器 -> 查询优化器 -> 执行器 -> 数据库DB


解析器：语法解析，生成语法树
预处理器：判断列名是否存在、权限等
优化器：系统对语法树进行优化，采用最优方式执行
执行器：调用数据库引擎的 API 进行数据操作

### 3、join 相关

1. 左连接：返回左表的所有记录以及右表的连接值相等的记录，右表没有数据的用 null 补全
2. 右连接：返回右表的所有记录以及右表的连接值相等的记录，左表没有数据的用 null 补全
3. 内连接：返回左表和右表接连值相等的记录（取交集）
4. 外连接：返回左表和右表连接值相等的所有记录（取并集）

## 包管理相关

### 1、carthge 和cocoapods

carthage

优势：

1. 去中心化，
2. 非侵入性，不会修改项目文件和生成配置
3. 只需编译一次

劣势：

1. 仅支持 iOS 8+
2. 第三方库对 carthage 的支持不太丰富
3. 无法在 xcode 里定位源码，难于调试
4. 安装包的大小比 cocoapods 体积大

cocoapods

优势：

1. 使用方便，除编写 podfile 文件以外几乎都是自动完成
2. 支持的库多，主流选择
3. 支持 iOS 8

缺点：

1. 每次更新 build 都会把第三方库重新编译一次
2. 有侵入性，会修改项目文件以及生成一些配置文件

### 2、静态库和动态库

#### 区别
1. 动态库以 .dyld 和 .framework 文件形式存在，静态以 .a 和 .framework 文件形式存在
2. 静态库在编译期间直接拷贝一份至目标程序中，编译完成后就可以运行
3. 动态库在编译期间不会进行拷贝，等到运行的时候才会被动态加载进来，同一份为可以被多个程序使用，具有共享性质，但是会带来部分性能损失

iOS 系统提供的库均是动态库，可以共享，但是提供给开发者可制作的动态库是被剦割的，叫 embedded framework。也需要要拷贝至 App 中。


## iOS 基础特性

### KVC 和 KVO

KVC: 键值编码，可以通过 key 或者 keypath 直接设置和获取对象的属性和方法的一种机制
主要的方法有： 
```swift
setValue: forKey
setValue: forKeyPath
getValue: forKey
getValue: forKeyPath

```
当能过 key 或者 keypath 找不到方法时，会报出 crash，unfindedKeyForValue

KVO: 键值监听，属于 OC runtime 特性之一，作用是某个观察者注册该对象的某个属性时监听时，当该对象的该属性修改时，观察者能收到通知和新的内容。

实现原理：某对象 A 被观察者监听时，系统会自动生成一个派生类 NSKVONotifying_A 的实例，该实现实现了 A Class 的方法及属性，并重写了属性的 setter 方法，该派生类的实例指向了 A 对象的 isa 指针，重写的 setter 方法会调用 keyforValueWillChange 以及 keyforValueDidChange 方法，并向观察者发送通知。

#### 使用 KVO 会有哪些问题？

1. 当观察者添加了监听时，未在 dealloc 方法调用的时候 remove 监听会导致 crash
2. KVO 的监听和 remove 数不匹配时会 crash，例如：重复 remove
3. 观察者未实现监听的回调函数 observeValueForKeyPath

### Runtime

Runtime: 运行时机制，为 OC 提供一种可以在运行时动态消息转发的可能
常用场景：Swizzle、KVO、关联属性、KV归档、动态方法解析

#### 消息传递机制

方法调用是通过向目标对象发送消息来完成的，使用的是 OC 的 `objc_msgSend(id receiver, SEL sel)`

#### 对象结构体

对象的结构体：
```
struct objc_object {
	Class isa OBJC_ISA_AVAILABILITY
}
```

类对象的结构体：
```
struct objc_class {
	Class isa // 指向元类
	Class Super_class // 父类
	const char name // 名称
	long version // 版本号
	long info // 基本信息
	long instancesize // 大小
	struct objc_iva_list ivars // 实例变量列表
	struct objc_method_list methodLists // 实例方法列表
	struct objc_protocol_list protocols // 协议列表
	struct objc_cache cache // 方法 cache
}
```

#### 元类对象 metaClass
元类对象中存放类变量、类方法（静态方法）以及类对象的创建方法及信息，元类中也有 isa 指针，指向的是 NSObject 的元类对象。

### Runtime 消息转发

消息发送是通过调用 `objc_msgSend(id receiver, SEL sel)` ，向接收者对象发送消息，首先是从对象的 cache 方法列表中找，然后从对象的方法列表开始找，找不到向父类的方法列表找，依次找到基类 NSObject 的方法列表，如果都找不到就开始消息转发流程。

#### 消息转发流程分为三步

1. 动态方法解析，使用 `resolveInstanceMethod` 来处理，如果返回 YES 则处理该消息
2. 备用接收者，使用 `forwardingTargetForSelector` 来返回备用接收者，如果返回不为 nil 则处理该消息
3. 消息重定向，分为两步，第一步，使用 `methodSignatureForSelector` 进行方法签名，如果返回 nil 则无法处理消息，并报出 unrecognized Selector 异常，反之则走第二步，使用 `forwardingInvocation` 来生成新的 NSInvocation 进行消息处理，如果未处理成功也会报出 unrecognized selector 异常。

#### 如何通过 Selector 寻找 IMP

runtime 提供两种方式：
1. `class_getMethodImplemention(Class cls, SEL name)`
2. `method_getImplemention(Method m)`

#### Swizzle

swizzle 作用是给方法进行替换，原理很简单即将方法的 IMP 进行替换即可。

方法添加方式： `class_addMethod(Class cls, SEL sel, IMP method, String code)`
cls: 被添加的类对象
name: 方法名
IMP: 方法实现指针
类型编码: "V@:"，内存模型相关

方法替换的方式有三种：
1. `method_exchangeImplementions` 来交换两个方法的实现
2. `class_replaceImplemention` 替换方法的实现
3. `method_setImplemention` 直接设置方法的实现





















