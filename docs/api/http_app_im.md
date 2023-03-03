# http api 接口-IM

## 1. UpdateConvSetting: 更新会话设置

- 请求地址：`/v1/im/updateConvSetting`
- 请求体：

```protobuf
message ConvSetting {
  string userId = 1;
  string convId = 2;
  // 置顶
  optional bool isTop = 3;
  // 设为免打扰
  optional bool isDisturb = 4;
  // 消息通知设置 （当免打扰时，此设置无效）
  // 通知显示消息预览
  optional bool notifyPreview = 5;
  // 通知声音
  optional bool notifySound = 6;
  // 通知自定义声音
  optional string notifyCustomSound = 7;
  // 通知震动
  optional bool notifyVibrate = 8;
  // 屏蔽消息
  optional bool isShield = 9;
  // 聊天背景
  optional string chatBg = 10;
}

//UpdateConvSettingReq 更新会话设置
message UpdateConvSettingReq {
  CommonReq commonReq = 1;
  ConvSetting convSetting = 2;
}
```

- 响应体：

```protobuf
//UpdateConvSettingResp 更新会话设置
message UpdateConvSettingResp {
  CommonResp commonResp = 1;
}
```

## 2. GetConvSetting: 获取会话设置

- 请求地址：`/v1/im/getConvSetting`
- 请求体：

```protobuf
//GetConvSettingReq 获取会话设置
message GetConvSettingReq {
  CommonReq commonReq = 1;
  repeated string convIds = 2;
}
```

- 响应体：

```protobuf
//GetConvSettingResp 获取会话设置
message GetConvSettingResp {
  CommonResp commonResp = 1;
  repeated ConvSetting convSettings = 2;
}
```