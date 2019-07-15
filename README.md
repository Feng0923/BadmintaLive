## 环境配置

### IDE

IntelliJ IDEA

![](https://ss1.baidu.com/6ONXsjip0QIZ8tyhnq/it/u=2995446634,2852731158&fm=179&app=42&f=JPG?w=121&h=140)

### 容器

Tomcat 9.0.0

端口：8080

### 框架

spring spring-mvc mybaits

### 数据库

版本：Mysql 8.0

utf-8 编码

数据库名：music

登录用户：root

密码：root

端口：3306

表：

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715140842.png)

user(用户表):

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715141001.png)

level 用户等级（0为超级管理员，1为一般管理员,2为用户），通过登录，账号的level来判定是超级管理员还是一般管理员，只为超级管理员和一般管理设计登录，超级管理员可以编辑，修改，删除。一般管理员只能查看。**所以先要通过登录超级管理员的账号来编辑，修改，删除。**

device（设备表,context为广告）:

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715141829.png)

music（音乐表）:

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715141929.png)

device_status(设备状态改变表):

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715141958.png)

记录，设备改变状态，返回操作记录。

music_status(音乐状态表):

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715142058.png)

记录音乐状态改变表。

recommend 推荐音乐列表：

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715142248.png)

mysql的配置文件，在工程的  resources/properties下配置：

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715141451.png)

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715141517.png)

前端文件位置：

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715172214.png)



### spring spirng-mvc 配置文件

在WEB-INF下：

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190616162614.png)

### 后台

在Controller层，返回的均是json数据。

### 流程

#### 登录

登录地址：

首先登陆超级管理员账号，没有添加超级管理员账号的界面，所以可以通过直接对数据库的更改进行，设置level为0，即可。然后用 session记录。

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715144939.png)

#### 用户管理（只显示账号，用户名，密码）：

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715145004.png)

#### 删除单条用户，

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715145128.png)

添加用户：

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715145221.png)

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715145245.png)

#### 编辑保存用户信息：

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715145320.png)

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715145428.png)

#### 查看设备状况：

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715145504.png)

#### 添加设备：

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715145606.png)

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715145739.png)

不能修改在线情况。

编辑广告，点击发送，即可向该设备发送广告内容。

#### 音乐管理：

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715145956.png)

增删改查，均可，搜索音乐，直接输入搜索词，然后回车即可。

#### 向指定设备推荐音乐

先选择推荐的音乐：

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715150133.png)

点击推荐，然后选择设备，推荐即可：

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715150210.png)

然后通过接口文档的接口，即可获取刚才向设备推荐的音乐。

### 接口说明

#### 1.上下线通知

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715142609.png)

传递接口文档所要求的参数就可以上下线，如果该设备没有在该表中，会自动插入。

#### 2.发送广告内容

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715142806.png)

传递接口文档所要求的参数就可以将该广告，向某设备更新广告，该设备要在原表中存在才行。

#### 3.向android发送广告

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715143347.png)

只返回，最近发送的一条广告。

返回示例，根据接口文档要求返回：

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715143519.png)



#### 4.后台设备状态修改通知：

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715143851.png)

只返回最后一条操作记录。

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715143810.png)

#### 5.向android手机app发送音乐列表

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715144217.png)

返回最近推荐列表。

支持一次推送多首歌给多台设备。

只返回最近一次推荐。

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715144108.png)

#### 6.向android手机发送音乐状态修改通知：

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715144419.png)

只返回最后一次状态修改记录。

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715144449.png)



### 第三方包

所需的第三方包，在pom.xml说明

### 系统日志

log4j。

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715150947.png)



打印在：

```properties
D:\\starMusic\\log\\error.log
```

配置文件，可以修改位置（绝对路径）：

![](https://veng-photo.oss-cn-beijing.aliyuncs.com/picgo/20190715151016.png)

