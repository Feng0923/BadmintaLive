### 	前后台交互方式

ajax，以 Json格式进行交互。

### 登录注册

#### 登录

后台接口：../user/login

请求方式：POST

返回类型：JSON

| 参数     | 是否必须 | 描述                      |
| -------- | -------- | ------------------------- |
| username | 必须     | 用户名                    |
| password | 必须     | 密码（已用 base 64 加密） |

正常返回：

```json
{
    "status":200，
    "message":"success"，
    "level":"1"，
}
```

level 来判断是普通会员 1，管理员 2.

账号不存在或者密码不正确：

```json
{
    "status":403，
    "message":"error"，
}
```

账号密码，判空，在后台验证。

账号空则，status 为 0，密码为空则，status 为 1.

#### 注册

后台接口：../user/register

请求方式：POST

返回类型：JSON

| 参数     | 是否必须 | 描述   |
| -------- | -------- | ------ |
| username | 是       | 用户名 |
| password | 是       | 密码   |
| email    | 是       | 邮箱   |

正常返回：

```json
{
    "status":200，
    "message":"success"，
}
```

用户注册后，设为普通会员。

该用户已注册：

```json
{
    "status":10，
    "message":"username exist"，
}
```

同样，判空，账号为空则返回 status = 0，密码为空则返回，status = 1，邮箱为空，返回 status = 2，如果邮箱不正确，则返回 status = 3.

#### 密码找回

后台接口：../user/findPassword

请求方式：POST

| 参数      | 是否必须 | 描述     |
| --------- | -------- | -------- |
| userEmail | 是       | 账号昵称 |

根据账号邮箱，去找用户，然后将用户名跟密码发送给该邮箱。

同样的，对邮箱需要判定，空，或者不存在。空返回status=0，不存在返回 status =1，符合则返回 status =200，成功。



### 管理员

功能：

- 查看比赛
- 发起比赛
  - 设置比赛项目
- 设置裁判组
- 比赛报名
- 生成对局
- 裁判计分
- 直播
- 发起活动

#### 比赛的逻辑

首先，输入比赛的信息，如下表所示，其中比赛场地，我们默认全部选上所有的场地，因为没有场地数据，只有场地的数目一个字段，然后，在设置比赛的项目，男单，女单等等，然后是以队伍来报名比赛项目。然后将所有的队，分成几个小组，每个小组的队伍分到3-4个队，小组数也因此而确定，最好（固定）是偶数个组进行小组赛，最后每个小组选出前两个队伍，然后进行淘汰赛，规则如下，第一组第一跟第二组的第二比赛，第一组的第二跟第二组的第一比赛，这样交叉比赛，赢者继续下面的比赛。后面每组就剩一个队伍，然后跟之前一样比赛，1-2，3-4,..。最后决出冠军。

#### 发起比赛

后台接口：../competition/add

请求方式:POST

| 参数            | 是否必须 | 描述                       |
| --------------- | -------- | -------------------------- |
| competitionName | 是       | 比赛名称                   |
| hostUnit        | 是       | 承办单位                   |
| type_id         | 是       | 比赛类型（0 平台，1 委托） |
| unitId          | 是       | 承办地点(后台获取)         |
| startTime       | 是       | 开始时间(yyyy-mm-dd hh:MM) |
| endTime         | 是       | 结束时间(yyyy-mm-dd hh:MM) |
| brief           | 否       | 比赛简介                   |

插入成功，返回：

```json
{
    "status":200,
    "message":"success"
}
```

插入失败：

```json
{
    "status":10,
    "message":"failure"
}
```



##### 数据库

###### 比赛表

如果想建立新表可以参考，不想得话，则按照原表参考，自行考虑，推荐后者。

比赛表 competition(原表为  badminton_competition_info)，表中括号为原表字段，后面也如此。

| 字段                        | 类型            | 描述                                          |
| --------------------------- | --------------- | --------------------------------------------- |
| cid（competitionId）自增    | int(11)         | 比赛 id                                       |
| cname（competitionName）    | varchar(100)    | 比赛名称                                      |
| unit（hostUnit）            | varchar(11)     | 承办单位                                      |
| type_id（外键 competition） | int(2)          | 比赛类型                                      |
| place_id（外键 unitId）     | int（4）        | 承办地点id                                    |
| start_time(startTime)       | datetime        | 开始时间                                      |
| end_time(endTime)           | datatime        | 结束时间                                      |
| brief                       | varchar(255)    | 比赛简介                                      |
| status(state)               | int(1)[int(11)] | 比赛状态(0已创建1可报名2进行中3已结束4已取消) |

###### 比赛类型表

 competition_type(该表没有，只是在原表中设置0 1 来表示,可能会有增加比赛类型的需求，根据数据库的设计，应该增加该表，自行考虑)

| 字段    | 类型        | 描述                           |
| ------- | ----------- | ------------------------------ |
| type_id | int（1）    | 比赛类型id                     |
| type    | varchar(10) | 比赛类型（目前只有平台和委托） |

###### 承办地点表

 (原表badminton_venue_unit)

| 字段             | 类型         | 描述       |
| ---------------- | ------------ | ---------- |
| venue_id(unitId) | int(11)      | 承办地点id |
| name(name)       | varchar(100) | 场馆名称   |
| province         | varchar(60)  | 省份       |
| city             | varchar(100) | 城市       |
| town             | varchar(100) | 镇         |
| address          | varchar(100) | 地址描述   |
| phone            | varchar(40)  | 电话       |

#### 获取场馆

后台接口：../venue/all

请求方式：GET

参数无

返回：

```json
{
    "status":200，
    "message":"success"，
    "data":[
        {
            "id":"场馆ID"，
            "name":"场馆名称"，
    		"province":"省份"，
    		"city":"城市"，
    		"town":"镇"，
    		"address":"地址描述"，
    		"phone":"电话"，
        }，
        ...
    ]
}
```



#### 设置比赛项目

后台接口：../project/add

请求方式:POST

| 参数            | 是否必须 | 描述              |
| --------------- | -------- | ----------------- |
| competitionId   | 是       | 比赛ID（所属比赛) |
| name            | 是       | 项目名称          |
| team_num        | 是       | 队数上限          |
| team_member_num | 是       | 队员上限          |
| type            | 是       | 项目类型          |

补充：

项目类型：0 男单 1 女单 2 男双 3 女双 4混双 5团体

插入正确返回：

```json
{
    "status":200,
    "message":"success",
}
```

插入错误返回：

```json
{
    "status":10,
    "message":"failure",
}
```



##### 数据库

###### 比赛项目表

competition_project (badminton_competition_project)

| 字段                | 类型        | 描述                                              |
| ------------------- | ----------- | ------------------------------------------------- |
| projectId 主键 自增 | int(11)     | 项目ID                                            |
| competitionId 外键  | int(11)     | 比赛ID                                            |
| projectName         | varchar(50) | 比赛名称                                          |
| maxTeamNum          | int(11)     | 队数上限                                          |
| maxTeamPersonNum    | int(11)     | 队员上限                                          |
| projectType         | int(1)      | 项目类型(0 男单 1 女单 2 男双 3 女双 4混双 5团体) |
| nowTeamNum          | int(11)     | 已报名队伍数量（初始化为0）                       |

#### 查看比赛

后台接口：../project/all

请求方式：GET

无参数。

返回比赛得详细列表。

正确的返回：

```json
{
    "status":200,
    "mesaage":"success",
    "data":[
        {
        "competitionId":"比赛ID",
        "competitionType":"比赛类型",
        "competitionName":"比赛名称",
        "breif":"比赛简介",
        "hostUnit":"承办单位",
        "state":"比赛的状态",
        "startTime":"比赛开始时间",
    	"endTime":"比赛结束时间",
    	"venue_name":"比赛场馆（地点）",
        }，
		...
    ]
}
```

#### 查看比赛项目

后台接口：..competition/allProject

请求方式：GET

参数：

| 参数          | 是否必须 | 描述                         |
| ------------- | -------- | ---------------------------- |
| competitionId | 是       | 根据比赛ID，查看该比赛的项目 |

正确返回：

```json
{
    "status":200,
    "message":"success",
    "data":[
        {
            "competitionId":"比赛ID",
            "projectId":"项目ID",
            "projectName":"比赛名称",
            "maxTeamNum":"队数上限",
            "maxTeamPersonNum":"队员上限",
            "projectType":"项目类型",
            "nowTeamNum":"已报名队伍数量",
        },
        ...
    ]
}
```

如果competitionId ，发现该比赛不存在，则返回 status = 10，message = "not exist".



#### 比赛报名

##### 报名逻辑

以队伍为单位，报名，队员的信息，可以不填，以项目规定的人数为准（默认参赛人数），将队伍个人信息，插入 info_person_info 表里。报名的用户，有name，但是其他的参赛队员是没有名字的，只有personId 表示。比赛是以队伍来进行比赛的。队伍表，为badminton_competition_team表，该表存储单打，双打，团体项目的队伍信息，personId1 和personId2 表示双打的人的personId，来自info_person_info,如果是单打，那么personId2则默认设置为0，只设置personId1，如果是团体的话，那么personId1 personId2均为0 团体信息在badminton_competition_team_person，同一个队的teamId相同，然后队员信息也在info_person_info里，teamId关联着badminton_competition_team表。



后台接口：../competition/register

请求方式：POST

参数：

| 参数                                  | 是否必须 | 描述                                     |
| ------------------------------------- | -------- | ---------------------------------------- |
| projectId                             | 是       | 项目ID                                   |
| teamName                              | 是       | 队伍名称                                 |
| createTime                            | 是       | 创建时间                                 |
| checkState                            | 否       | 队伍审核状态（0 审核中 1 通过 2 未通过） |
| personId1 （外键,现已改为 loginName） | 是       | 用户ID（用户名称）                       |
| personId2（外键）                     | 否       | 队员1 ID                                 |
| remark                                | 是       | (备注)联系方式                           |

插入后，要将队员信息包括队长信息，添加到info_person_info 表。personId自增，而perName 则是，该用户昵称，其他成员则不填名称。

插入成功：

```json
{
    "status":200,
    "message":"success",
    
}
```

插入失败：

```json
{
    "status":10,
    "message":"failure",
}
```

报名成功后，等待管理员审核。

##### 数据库

###### 队伍表

badminton_competition_team表

| 字段          | 类型         | 描述                                     |
| ------------- | ------------ | ---------------------------------------- |
| <u>teamId</u> | INT(11)      | 队伍ID                                   |
| projectId     | INT(11)      | 项目ID                                   |
| teamName      | VARCHAR(50)  | 队伍名称                                 |
| personId1     | INT(11)      | 队长ID（即该用户）                       |
| personId2     | INT(11)      | 队员ID 详情看逻辑说明                    |
| maxMemberNum  | INT(11)      | 最大成员数量（根据项目设定）             |
| nowMemberNum  | INT(11)      | 参赛队伍队员人数                         |
| creatTime     | DATETIME     | 队伍创建时间                             |
| checkState    | VARCHAR(11)  | 队伍审核状态（0 审核中 1 通过 2 未通过） |
| remark        | VARCHAR(100) | 备注（联系方式）                         |

###### 队员信息表

info_person_info

| 字段            | 类型        | 描述     |
| --------------- | ----------- | -------- |
| <u>personId</u> | INT(11)     | 队员ID   |
| perName         | VARCHAR(50) | 队员昵称 |

#### 查看未审核队伍

后台接口：../team/unreview/

请求参数：GET

返回格式：

```json
{
    "status":200,
    "message":"success",
    "data":[
            {
                "teamId":286,
                "projectId":1,
                "competitionName":"xxx羽毛球比赛",
                "projectName":"双打",
                "teamName":"lala",
                "nowMemberNum":2,
                "remark":1834567894,
                "checkState":0,
            }
    ]
}
```

#### 审核队伍

后台接口：../team/check

请求类型：POST

参数：

| 参数   | 是否必须 | 描述   |
| ------ | -------- | ------ |
| teamId | 是       | 队伍ID |

返回格式：

```json
{
    "status":200,
    "message":"审核成功",
}
```



#### 生成对局

先从每个项目，找到审核通过的队伍，然后对其分组，每组3-4个队伍，然后得到队伍数量，对4取模，余0，每个组都 4个队，没问题；余1，则，就有三个组是三个队伍，可以取前九个队，也可以取后九个队，自行选择，本人认为前者为佳，仅供参考；余2，则就有两个组是三个队，那么，与余1同理；余3，则只有一个组为三个队，同理。然后就依据比赛逻辑选出前两名。

生成的小组赛，表在 badminton_competition_team_group。groudId 相同的为同一个小组。

淘汰赛：badminton_competition_game

#### 获取对局

后台接口：../teamgroup/

请求方式：GET

无请求参数.

返回json:

```json
{
    "status":200,
    "message":"success",
    "data":[
        {
            
        }
    ]
}
```

#### 查看活动

后台接口：../activity/all

请求方式：GET

无请求参数

返回格式：

```json
{
    "status":200,
    "message":"success",
    "data":[
        {
            activityId:"活动id",
            activityName:"活动名称",
            activityBrief:"活动简介",
            originator:"发起人",
            placeName:"活动地点",
            startTime:"开始时间",
            endTime:"结束时间",
        }
    ]
}
```

