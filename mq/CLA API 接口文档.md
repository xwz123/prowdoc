## CLA API 接口文档

### 企业管理员授权

**接口地址：** https://clasign.osinfra.cn/api/v1/corporation-manager/auth

**请求方式：** POST

**请求参数：**

| 参数名   | 描述                                       | 必选 | 类型 |
| -------- | ------------------------------------------ | ---- | ---- |
| link_id  | 欧拉传 gitee_openeuler-1611298811283968340 | 是   | body |
| user     | 管理员用户名或者邮箱                       | 是   | body |
| password | 管理员用户密码                             | 是   | body |

**请求示例：**

```jso
{
  "link_id": "gitee_cla-test_1-1630929347138208500",
  "password": "123123",
  "user": "888@qq.com"
}
```

**返回示例**：

成功：200

```jso
{
    "data": [
        {
            "platform": "gitee",
            "org_id": "cla-test",
            "repo_id": "1",
            "role": "manager",
            "token": "jdosdoskdpkfspfkpskdpf",
            "initial_pw_changed": true
        }
    ]
}

```

失败：

```json
{
  "data": {
    "error_code": "cla.wrong_id_or_pw",
    "error_message": "wrong id or pw"
  }
}
```

### 企业员工管理员列表

**接口地址：** https://clasign.osinfra.cn/api/v1/employee-manager/

**请求方式：** GET

**请求参数：**

| 参数名 | 描述            | 必选 | 类型   |
| ------ | --------------- | ---- | ------ |
| Token  | 授权返回的token | 是   | header |

**返回示例：**

成功：200

```jso
{
    "data": [
        {
            "id": "888",
            "name": "888",
            "email": "888@qq.com",
            "role": "manager"
        },
        {
            "id": "111",
            "name": "111",
            "email": "111@qq.com",
            "role": "manager"
        }
    ]
}
```

失败：401

```json
{
  "data": {
    "error_code": "cla.unknown_token",
    "error_message": "encoding/hex: odd length hex string"
  }
}
```

### 企业员工签署列表

**接口地址**：https://clasign.osinfra.cn/api/v1/employee-signing/

**请求方式**：GET

**请求参数：**

| 参数名 | 描述            | 必选 | 类型   |
| ------ | --------------- | ---- | ------ |
| Token  | 授权返回的token | 是   | header |

**返回示例**：

成功：200

```json
{
    "data": [
        {
            "id": "",
            "email": "tt@qq.com",
            "name": "ttt",
            "date": "2021-09-26",
            "enabled": true
        }
    ]
}
```

失败：401

```json
{
  "data": {
    "error_code": "cla.unknown_token",
    "error_message": "encoding/hex: odd length hex string"
  }
}
```



### 获取所有已签署组织的企业

**请求地址**：https://clasign.osinfra.cn/api/v1/corporation-signing/{link_id}

**请求方式**：GET

**请求参数：**

| 参数名  | 描述                                       | 必选 | 类型   |
| ------- | ------------------------------------------ | ---- | ------ |
| Token   | 授权返回的token                            | 是   | header |
| link_id | 欧拉传 gitee_openeuler-1611298811283968340 | 是   | path   |

**返回示例**：

成功：200

```json
{
    "data": [
        {
            "cla_language": "chinese",
            "admin_email": "969707751@qq.com",
            "admin_name": "ooo",
            "corporation_name": "ooo",
            "date": "2021-09-07",
            "admin_added": true,
            "pdf_uploaded": true
        },
        {
            "cla_language": "chinese",
            "admin_email": "ooo@ee.com",
            "admin_name": "ooo",
            "corporation_name": "ooo",
            "date": "2021-09-09",
            "admin_added": true,
            "pdf_uploaded": true
        }
    ]
}
```

失败：401

```json
{
  "data": {
    "error_code": "cla.unknown_token",
    "error_message": "encoding/hex: odd length hex string"
  }
}
```

