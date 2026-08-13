# 数字证书柜 JS 交互接口文档

## 目录

- [一、接口通用说明](#一接口通用说明)
- [二、接口详情](#二接口详情)
  - [1. 批量下发用户](#1批量下发用户)
  - [2. 查询用户](#2查询用户)
  - [3. 绑定/解绑用户](#3绑定解绑用户)
  - [4. 初始化证书柜](#4初始化证书柜)
  - [5. 查询证书柜信息](#5查询证书柜信息)
  - [6. 查询证书柜状态](#6查询证书柜状态)
  - [7. 初始化系统配置](#7初始化系统配置)
  - [8. 显示系统配置](#8显示系统配置)
  - [9. 修改证书柜管理员密码](#9修改证书柜管理员密码)
  - [10. 管理员登录](#10管理员登录)
  - [11. 上报标签状态](#11上报标签状态)
  - [12. 上报门锁状态](#12上报门锁状态)
  - [13. 根据用户身份证号码更新用户信息](#13根据用户身份证号码更新用户信息)
  - [14. 人脸采集功能](#14人脸采集功能)
  - [15. 人脸识别功能](#15人脸识别功能)
  - [16. 人脸预览控制功能](#16人脸预览控制功能)
  - [17. 停止预览请求示例](#17停止预览请求示例)
  - [18. 批量插入操作日志](#18批量插入操作日志)
  - [19. 查询操作日志](#19查询操作日志)

---

## 一、接口通用说明

本文档定义数字证书柜 Web 页面与底层能力的交互接口，统一遵循「构建请求 JSON 对象 + 调用 `invokeAndroid` 方法发起请求 + 异步回调函数接收返回结果」的调用规范，所有接口的动作标识通过 `action` 字段指定。

---

## 二、接口详情

### 1. 批量下发用户

#### 接口说明
批量将人员信息同步至证书柜系统，完成人员基础信息的批量新增操作。

#### 请求示例
```javascript
var doInsertUserPo = function () {
    var action = "doInsertUserPo";
    var userJsonArrayStr = [
        {
            userName: "zhangsan",
            idNoUnique: "123",
            plsNoUnique: "A123",
            deptid: "9001",
            deptName: "行政"
        },
        {
            userName: "lisi",
            idNoUnique: "456",
            plsNoUnique: "A456",
            deptid: "9001",
            deptName: "行政"
        },
        {
            userName: "wangwu",
            idNoUnique: "789",
            plsNoUnique: "A789",
            deptid: "9001",
            deptName: "行政"
        }
    ];
    var json = {
        "action": action,
        "userJsonArrayStr": userJsonArrayStr
    };
    invokeAndroid(json);
};
```

#### 回调方法 `doInsertUserPoCallback` 返回数据示例
```json
{
  "code": 0,
  "msg": "新增用户成功",
  "failCnt": 0,
  "successCnt": 3,
  "userPoListJsonStr": "[{"cabinetCode":"cabinetCode","cabinetName":"cabinetName","deptId":"9001","deptName":"行政","faceBase64_image":"","faceFeature":"","firstFingerFeature":"","idNoUnique":"123","plsNoUnique":"A123","remark":"","secondeFingerFeature":"","slotPositions":"","thirdFingerFeature":"","userName":"zhangsan"},{"cabinetCode":"cabinetCode","cabinetName":"cabinetName","deptId":"9001","deptName":"行政","faceBase64_image":"","faceFeature":"","firstFingerFeature":"","idNoUnique":"456","plsNoUnique":"A456","remark":"","secondeFingerFeature":"","slotPositions":"","thirdFingerFeature":"","userName":"lisi"},{"cabinetCode":"cabinetCode","cabinetName":"cabinetName","deptId":"9001","deptName":"行政","faceBase64_image":"","faceFeature":"","firstFingerFeature":"","idNoUnique":"789","plsNoUnique":"A789","remark":"","secondeFingerFeature":"","slotPositions":"","thirdFingerFeature":"","userName":"wangwu"}]"
}
```

---

### 2. 查询用户

#### 接口说明
获取系统内所有已录入的人员全量信息。

#### 请求示例
```javascript
var doQueryUserPoList = function () {
    var action = "doQueryUserPoList";
    var json = {
        "action": action
    };
    invokeAndroid(json);
};
```

#### 回调方法 `doQueryUserPoListCallback` 返回数据示例
```json
{
  "code": 0,
  "msg": "人员信息查询成功:idNoUnique;//身份证号码; plsNoUnique;//警号",
  "userPoListJsonStr": "[{"cabinetCode":"cabinetCode","cabinetName":"cabinetName","deptId":"9001","deptName":"行政","faceBase64_image":"","faceFeature":"","firstFingerFeature":"","idNoUnique":"123","plsNoUnique":"A123","remark":"","secondeFingerFeature":"","slotPositions":"1_1,2_1","thirdFingerFeature":"","userName":"zhangsan"},{"cabinetCode":"cabinetCode","cabinetName":"cabinetName","deptId":"9001","deptName":"行政","faceBase64_image":"","faceFeature":"","firstFingerFeature":"","idNoUnique":"456","plsNoUnique":"A456","remark":"","secondeFingerFeature":"","slotPositions":"1_2","thirdFingerFeature":"","userName":"lisi"},{"cabinetCode":"cabinetCode","cabinetName":"cabinetName","deptId":"9001","deptName":"行政","faceBase64_image":"","faceFeature":"","firstFingerFeature":"","idNoUnique":"789","plsNoUnique":"A789","remark":"","secondeFingerFeature":"","slotPositions":"","thirdFingerFeature":"","userName":"wangwu"}]",
  "total": 3,
  "bindCnt": 2,
  "unBindCnt": 1
}
```

---

### 3. 绑定/解绑用户

#### 接口说明
实现人员与证书柜插槽的绑定关联，传入空 `bindNo` 即可解除对应插槽的绑定关系。

#### 请求参数说明
- `slotNoUnique`: 插槽编号，格式示例 `1_1`、`1_2`
- `bindNo`: 绑定人员的身份证号，传入空字符串则为解绑操作

#### 请求示例
```javascript
var doUpdateDevicePoBindNoBySlotNoUnique = function () {
    var action = "doUpdateDevicePoBindNoBySlotNoUnique";
    var deviceJsonArrayStr = [
        {
            bindNo: "123",
            slotNoUnique: "1_1"
        },
        {
            bindNo: "123",
            slotNoUnique: "2_1"
        },
        {
            bindNo: "456",
            slotNoUnique: "1_2"
        },
        {
            bindNo: "",
            slotNoUnique: "1_3"
        }
    ];
    var json = {
        "action": action,
        "deviceJsonArrayStr": deviceJsonArrayStr
    };
    invokeAndroid(json);
};
```

#### 回调方法 `doUpdateDevicePoBindNoBySlotNoUniqueCallback` 返回字段补充说明
- `useStatus`: 插槽使用状态 `0`-空闲，`1`-使用中，`2`-脱机
- `status`: 插槽设备状态 `0`-正常，`1`-禁用，`2`-异常

#### 返回数据示例
```json
{
  "code": 0,
  "msg": "绑定/解绑用户成功",
  "devicePoListJsonStr": "[{"bindNo":"123","remark":"","slotNoUnique":"1_1","status":"0","uid":"","useStatus":"0"},{"bindNo":"456","remark":"","slotNoUnique":"1_2","status":"0","uid":"","useStatus":"0"},{"bindNo":"","remark":"","slotNoUnique":"1_3","status":"0","uid":"","useStatus":"0"},{"bindNo":"123","remark":"","slotNoUnique":"2_1","status":"0","uid":"","useStatus":"0"}]"
}
```

---

### 4. 初始化证书柜

#### 接口说明
根据预先配置的行、列参数初始化证书柜所有插槽，仅首次部署时执行一次即可，重复执行会重置所有插槽状态。

#### 请求示例
```javascript
var doInitDevicePo = function () {
    var action = "doInitDevicePo";
    var json = {
        "action": action
    };
    invokeAndroid(json);
};
```

#### 回调方法 `doInitDevicePoCallback` 返回数据示例
```json
{
  "code": 0,
  "msg": "初始化证书柜成功",
  "devicePoListJsonStr": "[{"bindNo":"","remark":"","slotNoUnique":"1_1","status":"0","uid":"","useStatus":"0"},{"bindNo":"","remark":"","slotNoUnique":"1_2","status":"0","uid":"","useStatus":"0"},{"bindNo":"","remark":"","slotNoUnique":"1_3","status":"0","uid":"","useStatus":"0"},{"bindNo":"","remark":"","slotNoUnique":"1_4","status":"0","uid":"","useStatus":"0"},{"bindNo":"","remark":"","slotNoUnique":"1_5","status":"0","uid":"","useStatus":"0"},{"bindNo":"","remark":"","slotNoUnique":"2_1","status":"0","uid":"","useStatus":"0"},{"bindNo":"","remark":"","slotNoUnique":"2_2","status":"0","uid":"","useStatus":"0"},{"bindNo":"","remark":"","slotNoUnique":"2_3","status":"0","uid":"","useStatus":"0"},{"bindNo":"","remark":"","slotNoUnique":"2_4","status":"0","uid":"","useStatus":"0"},{"bindNo":"","remark":"","slotNoUnique":"2_5","status":"0","uid":"","useStatus":"0"}]"
}
```

---

### 5. 查询证书柜信息

#### 接口说明
获取所有证书柜插槽的全量状态数据。

#### 请求示例
```javascript
var doQueryDevicePoList = function () {
    var action = "doQueryDevicePoList";
    var json = {
        "action": action
    };
    invokeAndroid(json);
};
```

#### 回调方法 `doQueryDevicePoListCallback` 返回数据示例
```json
{
  "code": 0,
  "msg": "设备信息查询成功:slotNoUnique;//插槽编号1_1、1_2;bindNo;//绑定人员的编号;useStatus;//使用状态 0-空闲 1-使用中 2:脱机;status;//设备状态0-正常,1-禁用,2-异常;uid;//标签编号.",
  "devicePoListJsonStr": "[{"bindNo":"123","remark":"","slotNoUnique":"1_1","status":"0","uid":"","useStatus":"0"},{"bindNo":"456","remark":"","slotNoUnique":"1_2","status":"0","uid":"","useStatus":"0"},{"bindNo":"","remark":"","slotNoUnique":"1_3","status":"0","uid":"","useStatus":"0"},{"bindNo":"","remark":"","slotNoUnique":"1_4","status":"0","uid":"","useStatus":"0"},{"bindNo":"","remark":"","slotNoUnique":"1_5","status":"0","uid":"","useStatus":"0"},{"bindNo":"123","remark":"","slotNoUnique":"2_1","status":"0","uid":"","useStatus":"0"},{"bindNo":"","remark":"","slotNoUnique":"2_2","status":"0","uid":"","useStatus":"0"},{"bindNo":"","remark":"","slotNoUnique":"2_3","status":"0","uid":"","useStatus":"0"},{"bindNo":"","remark":"","slotNoUnique":"2_4","status":"0","useStatus":"0"},{"bindNo":"","remark":"","slotNoUnique":"2_5","status":"0","uid":"","useStatus":"0"}]",
  "total": 10,
  "bindCnt": 3,
  "freeCnt": 10,
  "busyCnt": 0,
  "abnormalityCnt": 0
}
```

---

### 6. 查询证书柜状态

#### 接口说明
获取当前证书柜的柜门开关状态。

#### 请求示例
```javascript
var doShowCabinetDoorStatus = function () {
    let action = "doShowCabinetDoorStatus";
    let json = {
        "action": action
    };
    invokeAndroid(json);
};
```

#### 回调方法 `doConfigCallback` (通过返回的 `action` 字段区分当前接口回调) 返回数据示例
```json
{
  "code": 0,
  "msg": "证书柜状态查询成功 0-关闭;1-开启",
  "jsonStr": "{"cabinetDoorStatus":"0"}",
  "action": "doShowCabinetDoorStatus"
}
```

---

### 7. 初始化系统配置

#### 接口说明
配置证书柜核心系统参数，仅传入的非空字段会执行更新，建议仅在首次部署时修改一次参数。

#### 请求参数说明

| 字段名 | 字段说明 |
| :--- | :--- |
| `cabinetCode` | 证书柜编号 |
| `cabinetName` | 证书柜名称 |
| `mgrLoginName` | 管理员登录名 |
| `mgrPwd` | 管理员密码 |
| `baiduLicenceCode` | 百度离线人脸识别授权码 |
| `frontOrBackCamera` | 摄像头选择，`front` 为启用前置摄像头 |
| `lockCabinetPath` | 锁控设备串口路径，默认 `/dev/ttyS3` |
| `rows` | 证书柜行数 |
| `columns` | 证书柜列数 |

#### 请求示例
```javascript
var doInitConfig = function () {
    var action = "doInitConfig";
    var cabinetCode = "cabinetCode";
    var cabinetName = "cabinetName";
    var mgrLoginName = "admin";
    var mgrPwd = "123456";
    var baiduLicenceCode = $("#baiduLicenceCode").val();
    var frontOrBackCamera = "front";
    var lockCabinetPath = "/dev/ttyS3";
    var rows = "2";
    var columns = "5";

    var configJsonStr = {
        cabinetCode: cabinetCode,
        cabinetName: cabinetName,
        mgrLoginName: mgrLoginName,
        mgrPwd: mgrPwd,
        baiduLicenceCode: baiduLicenceCode,
        frontOrBackCamera: frontOrBackCamera,
        lockCabinetPath: lockCabinetPath,
        rows: rows,
        columns: columns
    };

    var json = {
        "action": action,
        "configJsonStr": configJsonStr
    };
    invokeAndroid(json);
};
```

#### 回调方法 `doConfigCallback` 返回数据示例
```json
{
  "code": 0,
  "msg": "doInitConfig 初始化配置成功",
  "jsonStr": "{"lockCabinetPath":"\/dev\/ttyS3","cabinetName":"cabinetName","mgrPwd":"123456","columns":"5","cabinetCode":"cabinetCode","mgrLoginName":"admin","rows":"2","frontOrBackCamera":"front"}",
  "action": "doInitConfig"
}
```

---

### 8. 显示系统配置

#### 接口说明
查询当前系统所有已保存的全量配置参数。

#### 请求示例
```javascript
var doShowConfig = function () {
    let action = "doShowConfig";
    let json = {
        "action": action
    };
    invokeAndroid(json);
};
```

#### 回调方法 `doConfigCallback` 返回数据示例
```json
{
  "code": 0,
  "msg": "配置数据查询成功",
  "jsonStr": "{"lockCabinetPath":"\/dev\/ttyS3","cabinetName":"cabinetName","cabinetDoorStatus":"0","mgrPwd":"123456","columns":"5","cabinetCode":"cabinetCode","baiduLicenceCode":"EAQE-FAL8-WXJS-T9NF","mgrLoginName":"admin","rows":"2","frontOrBackCamera":"front"}",
  "action": "doShowConfig"
}
```

---

### 9. 修改证书柜管理员密码

#### 接口说明
更新管理员账号的登录密码。

#### 请求示例
```javascript
var doModifyMgrPwdInitConfig = function () {
    var action = "doInitConfig";
    var mgrLoginName = "admin";
    var mgrPwd = "123456";
    var configJsonStr = {
        mgrLoginName: mgrLoginName,
        mgrPwd: mgrPwd
    };
    var json = {
        "action": action,
        "configJsonStr": configJsonStr
    };
    invokeAndroid(json);
};
```

#### 返回数据示例
```json
{
  "code": 0,
  "msg": "doInitConfig 初始化配置成功",
  "jsonStr": "{"mgrPwd":"123456","mgrLoginName":"admin"}",
  "action": "doInitConfig"
}
```

---

### 10. 管理员登录

#### 接口说明
校验管理员账号密码合法性，完成系统管理员身份鉴权。

#### 请求示例
```javascript
var doMgrLogin = function () {
    var action = "doMgrLogin";
    let mgrLoginName = "admin";
    let mgrPwd = "123456";
    var json = {
        "action": action,
        "mgrLoginName": mgrLoginName,
        "mgrPwd": mgrPwd
    };
    invokeAndroid(json);
};
```

#### 回调方法 `doMgrLoginCallback` 返回状态码说明
- `0`: 登录成功
- `-1`: 管理员用户名不正确
- `-2`: 密码不正确

#### 返回数据示例
```json
{
  "code": 0,
  "msg": "管理员 admin 登录成功",
  "action": "doMgrLogin"
}
```

---

### 11. 上报标签状态

#### 接口说明
控制板自动检测到电机锁标签状态变化后主动触发的通知事件，通知前端调用「查询证书柜信息」接口同步更新插槽显示状态。

#### 事件触发参数
- `layer`: 插槽所属层数
- `numb`: 插槽在当前层的序号
- `uid`: 标签 ID

---

### 12. 上报门锁状态

#### 接口说明
控制板自动监测到门锁开关状态变化后主动触发的通知事件，通知前端调用「查询证书柜状态」接口同步更新门禁显示状态。

#### 事件触发参数
- `doorLockState`: `0x01` 表示门开启，`0x00` 表示门关闭

---

### 13. 根据用户身份证号码更新用户信息

#### 接口说明
通过身份证号作为唯一标识，更新指定人员的部门信息、人脸信息、指纹信息等用户扩展属性。

#### 请求示例
```javascript
var doUpdateUserPoByIdNo = function () {
    var action = "doUpdateUserPoByIdNo";
    let idNoUnique = "123";
    let deptid = "90003";
    let deptName = "财务部";
    let faceFeature = "";
    let faceBase64_image = "";
    let firstFingerFeature = "";
    let userName = "wutian";

    let userPoJsonStr = {
        idNoUnique: idNoUnique,
        deptid: deptid,
        deptName: deptName,
        faceFeature: faceFeature,
        faceBase64_image: faceBase64_image,
        firstFingerFeature: firstFingerFeature,
        userName: userName
    };

    var json = {
        "action": action,
        "userPoJsonStr": userPoJsonStr
    };
    invokeAndroid(json);
};
```

#### 回调方法 `doUpdateUserPoByIdNoCallback` 返回数据示例
```json
{
  "code": 0,
  "msg": "{"cabinetCode":"cabinetCode","cabinetName":"cabinetName","deptId":"90003","deptName":"财务部","faceBase64_image":"","faceFeature":"","firstFingerFeature":"","idNoUnique":"123","plsNoUnique":null,"remark":"","secondeFingerFeature":"","slotPositions":"","thirdFingerFeature":"","userName":"wutian"}数据更新成功"
}
```

---

### 14. 人脸采集功能

#### 接口说明
为指定身份证号的人员采集人脸数据，存储人脸特征和人脸图片。

#### 请求示例
```javascript
var doFaceCollect = function () {
    var action = "doFaceCollect";
    let idNoUnique = "456";
    var json = {
        "action": action,
        "idNoUnique": idNoUnique
    };
    invokeAndroid(json);
};
```

#### 回调方法 `termFaceCallback` (通过返回的 `action` 字段区分回调类型) 返回数据示例
```json
{
  "action": "doFaceCollect",
  "code": "0",
  "msg": "456 人脸采集成功",
  "termFaceFeature": "",
  "termFaceImageBase64Str": ""
}
```

---

### 15. 人脸识别功能

#### 接口说明
核验指定身份证号人员的人脸身份，返回识别到的人脸特征和人脸图片。

#### 请求示例
```javascript
var doFaceDetect = function () {
    var action = "doFaceDetect";
    let idNoUnique = "123";
    var json = {
        "action": action,
        "idNoUnique": idNoUnique
    };
    invokeAndroid(json);
};
```

#### 回调方法 `termFaceCallback` 返回数据示例
```json
{
  "action": "doFaceDetect",
  "idNoUnique": "123",
  "code": "0",
  "msg": "123 人脸识别成功",
  "termFaceFeature": "22356391fc2f1b94f8ad3019324d311d8be5e0805ceaf584fa134b09922a150cdd9",
  "termFaceImageBase64Str": "/9j/4AAQSkZJRgABAQAAAQABAAD/4gHYSUNDX1BST0ZJTEUAAQEAAAHIAAAAAAQWAABtbnRyUkdCIFhZWiAH4AABAAEAAAAAAABhY3NWAAA"
}
```

---

### 16. 人脸预览控制功能

#### 接口说明
控制摄像头人脸预览的启动和停止。

#### 启动预览请求示例
```javascript
var startCameraPreview = function () {
    var action = "startCameraPreview";
    var json = {
        "action": action
    };
    invokeAndroid(json);
};
```

---

### 17. 停止预览请求示例

```javascript
var stopCameraPreview = function () {
    var action = "stopCameraPreview";
    var json = {
        "action": action
    };
    invokeAndroid(json);
};
```

> **通用回调方法**：`termFaceCallback`

---

### 18. 批量插入操作日志

#### 接口说明
支持单条或批量新增证书柜存取操作日志，记录人员存取证书的完整操作轨迹。

#### 请求示例
```javascript
var doInsertOperateLogPo = function () {
    var action = "doInsertOperateLogPo";
    var operateLogJsonArrayStr = [
        {
            userName: "zhangsan",
            idNo: "123",
            plsNo: "A123",
            layer: "1",
            numb: "1",
            operateType: "0",
            remark: "备注 01"
        },
        {
            userName: "lisi",
            idNo: "456",
            plsNo: "A456",
            layer: "1",
            numb: "2",
            operateType: "1",
            remark: "备注 02"
        },
        {
            userName: "wangwu",
            idNo: "789",
            plsNo: "A789",
            layer: "2",
            numb: "1",
            operateType: "0",
            remark: "备注 03"
        }
    ];
    var json = {
        "action": action,
        "operateLogJsonArrayStr": operateLogJsonArrayStr
    };
    invokeAndroid(json);
};
```

#### 回调方法 `doInsertOperateLogPoCallback` 返回数据示例
```json
{
  "code": 0,
  "msg": "日志新增成功",
  "failCnt": 0,
  "successCnt": 3
}
```

---

### 19. 查询操作日志

#### 接口说明
获取证书柜所有历史操作日志数据，统计存取操作数量。

#### 请求示例
```javascript
var doQueryOperateLogPoList = function () {
    var action = "doQueryOperateLogPoList";
    var json = {
        "action": action
    };
    invokeAndroid(json);
};
```

#### 回调方法 `doQueryOperateLogPoListCallback` 返回数据示例
```json
{
  "code": 0,
  "msg": "日志查询成功 operateType (0-存 1-取)",
  "userPoListJsonStr": "[{"cabinetCode":"cabinetCode","cabinetName":"cabinetName","idNo":"123","layer":"1","numb":"1","operateTime":"2026-07-19 16:42:49","operateType":"0","plsNo":"A123","remark":"备注 01","slotPosition":"1_1","userName":"zhangsan"},{"cabinetCode":"cabinetCode","cabinetName":"cabinetName","idNo":"456","layer":"1","numb":"2","operateTime":"2026-07-19 16:42:49","operateType":"1","plsNo":"A456","remark":"备注 02","slotPosition":"1_2","userName":"lisi"},{"cabinetCode":"cabinetCode","cabinetName":"cabinetName","idNo":"789","layer":"2","numb":"1","operateTime":"2026-07-19 16:42:49","operateType":"0","plsNo":"A789","remark":"备注 03","slotPosition":"2_1","userName":"wangwu"}]",
  "total": 3,
  "inCnt": 2,
  "outCnt": 1
}
