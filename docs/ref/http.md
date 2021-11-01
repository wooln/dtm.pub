# HTTP

## 概述
DTM 可以通过HTTP协议交互。因为事务本身具有特殊性，它和普通的api调用只区分成功和失败不同，DTM的结果状态详情，请参考[协议](../summary/protocol)

接口主要分为三类： AP调用TM的接口、AP调用RM的接口、TM调用RM的接口

## AP调用TM的接口

这部分接口的路径前缀为/api/dtmsvr。例如后面介绍的prepare，实际路径为/api/dtmsvr/prepare

这部分接口如果对事务状态进行变更，则dtm会对事务状态做相关的校验，有可能校验失败，而返回错误。以一个实际的例子说明：

如果调用prepare时，这个事务状态已经是failed，那么dtm将返回错误：
``` JSON
{
    "dtm_result":"FAILURE",
    "message":"current status 'failed', cannot prepare"
}
```

每个接口的详细说明如下：

### newGid 获取Gid

此接口用于获取gid。dtm的gid生成采用ip+snowflake。如果发生短时间内ip重用的情况，有极低概率发生gid重复情况。

因为几乎每个公司都会有自己的唯一id生成基础设置，建议在dtm中使用的gid，采用自己公司内部的唯一id生成算法。

| 接口名称 | 请求方法 | 请求格式 | 适用事务 |
|:----:|:-----:|:----:|:-----:|
| newGid | GET | - | - |

请求示例
``` bash
curl 'localhost:8080/api/dtmsvr/newGid'
```

响应示例
``` JSON
{
    "dtm_result":"SUCCESS",
    "gid":"c0a8038b_4nxAcyxSX8N"
}
```

### prepare 准备事务

此接口用于准备事务，正常情况下，后续将提交事务。

| 接口名称 | 请求方法 | 请求格式 | 适用事务 |
|:----:|:-----:|:----:|:-----:|
| prepare | POST | JSON | [MSG][], [TCC][], [XA][] |

请求示例
``` bash
curl --location --request POST 'localhost:8080/api/dtmsvr/prepare' \
--header 'Content-Type: application/json' \
--data-raw '{
    "gid": "xxx",
    "trans_type": "tcc"
}'
```

响应示例
``` JSON
{
    "dtm_result":"SUCCESS"
}
```

失败响应示例
``` JSON
{
    "dtm_result":"FAILURE",
    "message":"current status 'failed', cannot prepare"
}
```

### submit 提交事务

此接口用于提交全局事务

| 接口名称 | 请求方法 | 请求格式 | 适用事务 |
|:----:|:-----:|:----:|:-----:|
| submit | POST | JSON | [MSG][], [SAGA][], [TCC][], [XA][] |

请求示例
``` bash
curl --location --request POST 'localhost:8080/api/dtmsvr/submit' \
--header 'Content-Type: application/json' \
--data-raw '{
    "gid": "xxx",
    "trans_type": "tcc"
}'
```

响应示例
``` JSON
{
    "dtm_result":"SUCCESS"
}
```

### abort 回滚事务

此接口用于回滚全局事务

| 接口名称 | 请求方法 | 请求格式 | 适用事务 |
|:----:|:-----:|:----:|:-----:|
| abort | POST | JSON | [TCC][], [XA][] |

请求示例
``` bash
curl --location --request POST 'localhost:8080/api/dtmsvr/abort' \
--header 'Content-Type: application/json' \
--data-raw '{
    "gid": "xxx",
    "trans_type": "tcc"
}'
```

响应示例
``` JSON
{
    "dtm_result":"SUCCESS"
}
```

### registerXaBranch 注册xa分支

此接口用于xa事务模式中，注册一个xa事务分支

| 接口名称 | 请求方法 | 请求格式 | 适用事务 |
|:----:|:-----:|:----:|:-----:|
| registerXaBranch | POST | JSON | [XA][] |

请求示例
``` bash
curl --location --request POST 'localhost:8080/api/dtmsvr/registerXaBranch' \
--header 'Content-Type: application/json' \
--data-raw '{
    "branch_id":"0101",
    "gid":"c0a8038b_4nxEEB1M7K3",
    "trans_type":"xa",
    "url":"http://localhost:8081/api/busi/xa"
}'
```

响应示例
``` JSON
{
    "dtm_result":"SUCCESS"
}
```

### registerTccBranch 注册tcc分支

此接口用于tcc事务模式中，注册一个tcc事务分支

| 接口名称 | 请求方法 | 请求格式 | 适用事务 |
|:----:|:-----:|:----:|:-----:|
| registerXaBranch | POST | JSON | [TCC][] |

请求示例
``` bash
curl --location --request POST 'localhost:8080/api/dtmsvr/registerTccBranch' \
--header 'Content-Type: application/json' \
--data-raw '{
    "branch_id":"01",
    "cancel":"http://localhost:8081/api/busi/TransOutRevert",
    "confirm":"http://localhost:8081/api/busi/TransOutConfirm",
    "data":"{\"amount\":30,\"transInResult\":\"\",\"transOutResult\":\"\"}",
    "gid":"c0a8038b_4nxEEGy7W5h",
    "trans_type":"tcc",
    "try":"http://localhost:8081/api/busi/TransOut"
}'
```

响应示例
``` JSON
{
    "dtm_result":"SUCCESS"
}
```

### query 查询事务

此接口用于查询指定gid的事务。

| 接口名称 | 请求方法 | 请求格式 | 适用事务 |
|:----:|:-----:|:----:|:-----:|
| query | GET | - | - |

请求示例
``` bash
curl 'localhost:8080/api/dtmsvr/query?gid=xxx'
```

成功响应示例
``` JSON
{
    "branches":[
    ],
    "transaction":{
        "ID":69,
        "CreateTime":"2021-10-25T08:51:27+08:00",
        "UpdateTime":"2021-10-25T08:51:39+08:00",
        "gid":"xxx",
        "trans_type":"tcc",
        "data":"",
        "status":"failed",
        "query_prepared":"",
        "protocol":"http",
        "CommitTime":null,
        "FinishTime":null,
        "RollbackTime":"2021-10-25T08:51:39+08:00",
        "Options":"{}",
        "NextCronInterval":10,
        "NextCronTime":"2021-10-25T08:51:49+08:00"
    }
}
```

未找到gid的响应示例
``` JSON
{
    "branches":[
    ],
    "transaction":null
}
```

### all 批量查询事务

此接口用于批量查询事务。dtm将返回id小于last_id，按照id从大到小排序的100条数据

| 接口名称 | 请求方法 | 请求格式 | 适用事务 |
|:----:|:-----:|:----:|:-----:|
| all | GET | - | - |

请求示例
``` bash
curl 'localhost:8080/api/dtmsvr/all?last_id='
```

参数last_id: 上一次请求返回数据的最小id。

成功响应示例
``` JSON
{
    "transactions":[
        {
            "ID":69,
            "CreateTime":"2021-10-25T08:51:27+08:00",
            "UpdateTime":"2021-10-25T08:51:39+08:00",
            "gid":"xxx",
            "trans_type":"tcc",
            "data":"",
            "status":"failed",
            "query_prepared":"",
            "protocol":"http",
            "CommitTime":null,
            "FinishTime":null,
            "RollbackTime":"2021-10-25T08:51:39+08:00",
            "Options":"{}",
            "NextCronInterval":10,
            "NextCronTime":"2021-10-25T08:51:49+08:00"
        }
    ]
}
```

空响应示例
``` JSON
{
    "transactions":[
    ]
}
```




[MSG]: ../practice/msg "MSG"
[TCC]: ../practice/tcc "TCC"
[XA]: ../practice/xa "XA"
[XA]: ../practice/saga "SAGA"