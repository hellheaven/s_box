# 警务数字证书管理系统 — 后端 API 接口文档

## 概述

Base URL: `https://<server>/api`

所有接口返回统一格式：

```json
{
  "code": 0,
  "msg": "成功",
  "data": {}
}
```

| code | 说明 |
|------|------|
| 0 | 成功 |
| 1001 | 参数错误 |
| 1002 | 未授权 / token 无效 |
| 2001 | 人脸识别失败 |
| 2002 | 用户不存在 |
| 2003 | 锁孔已被占用 |
| 2004 | 锁孔未绑定用户 |
| 2005 | 柜门未关闭 |
| 5000 | 服务器内部错误 |

认证方式：除登录接口外，均需在 Header 携带 `Authorization: Bearer <token>`。

---

## 1. 管理员登录

### `POST /auth/login`

**Request Body：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| username | string | 是 | 管理员用户名 |
| password | string | 是 | 管理员密码 |

**Response：**

```json
{
  "code": 0,
  "msg": "登录成功",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIs..."
  }
}
```

---

## 2. 人脸识别认证

### `POST /auth/face`

页面点击"领取"或"存放"后，调用本接口进行人脸识别，获取警员身份及分配的锁孔信息。

**Request Body：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| faceImage | string | 是 | 人脸图片 Base64 |
| mode | string | 是 | `"retrieve"` 领取 / `"store"` 存放 |

**Response：**

```json
{
  "code": 0,
  "msg": "识别成功",
  "data": {
    "policeNo": "666661",
    "name": "张三",
    "certNo": "CERT-2026-066661",
    "assignedSlots": [
      { "layer": 1, "numb": 5, "status": "free" },
      { "layer": 2, "numb": 2, "status": "used" }
    ]
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| data.policeNo | string | 警号 |
| data.name | string | 姓名 |
| data.certNo | string | 当前证书编号 |
| data.assignedSlots | array | 该警员分配的锁孔列表 |
| data.assignedSlots[].layer | int | 层号 |
| data.assignedSlots[].numb | int | 孔号 |
| data.assignedSlots[].status | string | `"free"` 空闲（可存放） / `"used"` 占用（可领取） |

---

## 3. 人员管理

### 3.1 获取人员列表

### `GET /personnel`

**Query Parameters：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| keyword | string | 否 | 按警号或姓名模糊搜索 |

**Response：**

```json
{
  "code": 0,
  "data": {
    "records": [
      {
        "policeNo": "666661",
        "name": "张三",
        "post": "民警",
        "dept": "刑侦支队",
        "phone": "13800138001",
        "faceRegistered": true,
        "fingerRegistered": false,
        "boundSlots": [
          { "layer": 1, "numb": 5, "status": "free" },
          { "layer": 2, "numb": 2, "status": "used" }
        ]
      }
    ]
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| faceRegistered | boolean | 是否已录入人脸 |
| fingerRegistered | boolean | 是否已录入指纹 |
| boundSlots | array | 绑定锁孔列表 |

### 3.2 上传人脸特征

### `POST /personnel/{policeNo}/face`

**Request Body：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| faceImage | string | 是 | 人脸图片 Base64 |

**Response：**

```json
{
  "code": 0,
  "msg": "人脸录入成功"
}
```

### 3.3 上传指纹特征

### `POST /personnel/{policeNo}/finger`

**Request Body：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| fingerImage | string | 是 | 指纹图片 Base64 |

**Response：**

```json
{
  "code": 0,
  "msg": "指纹录入成功"
}
```

---

## 4. 锁孔管理

### 4.1 获取所有锁孔状态

### `GET /slots`

**Response：**

```json
{
  "code": 0,
  "data": {
    "slots": [
      {
        "layer": 1,
        "numb": 1,
        "status": "used",
        "policeNo": "666661",
        "certNo": "CERT-2026-066661",
        "userName": "张三"
      },
      {
        "layer": 1,
        "numb": 5,
        "status": "free",
        "policeNo": "",
        "certNo": "",
        "userName": ""
      }
    ]
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| data.slots[].status | string | `"free"` 空闲 / `"used"` 占用 / `"disconnected"` 断开 |

### 4.2 绑定锁孔

### `POST /slots/bind`

管理员在管理页面将人员绑定到指定锁孔。

**Request Body：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| policeNo | string | 是 | 警号 |
| layer | int | 是 | 层号 |
| numb | int | 是 | 孔号 |

**Response：**

```json
{
  "code": 0,
  "msg": "绑定成功"
}
```

### 4.3 解绑锁孔

### `POST /slots/unbind`

**Request Body：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| policeNo | string | 是 | 警号 |
| layer | int | 是 | 层号 |
| numb | int | 是 | 孔号 |

**Response：**

```json
{
  "code": 0,
  "msg": "解绑成功"
}
```

---

## 5. 证书操作

### 5.1 领取证书

### `POST /certificates/retrieve`

确认领取证书，后端记录领取日志。

**Request Body：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| policeNo | string | 是 | 警号 |
| certNo | string | 是 | 证书编号 |
| layer | int | 是 | 锁孔层号 |
| numb | int | 是 | 锁孔号 |

**Response：**

```json
{
  "code": 0,
  "msg": "领取成功"
}
```

### 5.2 存放证书

### `POST /certificates/store`

确认存放证书，后端验证柜门已关闭后完成存放，记录存放日志。

**Request Body：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| policeNo | string | 是 | 警号 |
| certNo | string | 是 | 证书编号 |
| layer | int | 是 | 锁孔层号 |
| numb | int | 是 | 锁孔号 |

**Response：**

```json
{
  "code": 0,
  "msg": "存放成功"
}
```

---

## 6. 数据同步

### `POST /sync`

管理员点击管理页面"同步"按钮，从外部系统（如警员 HR 系统）同步最新人员信息及锁孔绑定关系。

**Request Body：**

无

**Response：**

```json
{
  "code": 0,
  "msg": "数据同步成功",
  "data": {
    "personnelSync": {
      "total": 150,
      "added": 2,
      "updated": 5,
      "removed": 1
    },
    "slotSync": {
      "total": 60,
      "bound": 45,
      "unbound": 15
    }
  }
}
```


