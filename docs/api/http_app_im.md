# http api 接口-IM

## 1. UpdateConvSetting: 更新会话设置

- 请求地址：`/v1/im/updateConvSetting`
- 请求体：

```protobuf
//UpdateConvSettingReq 更新会话设置
message UpdateConvSettingReq {
  CommonReq commonReq = 1;
  string convId = 2;
  optional bool isTop = 3;
}
```

- 响应体：

```protobuf
//UpdateConvSettingResp 更新会话设置
message UpdateConvSettingResp {
  CommonResp commonResp = 1;
}
```
