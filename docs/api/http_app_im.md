# http api IM

## 1. GetAppSystemConfig: 获取应用系统配置

- 请求地址：`/v1/im/white/getAppSystemConfig`
- 请求体：

```protobuf
//GetAppSystemConfigReq 获取系统配置
message GetAppSystemConfigReq {
  CommonReq commonReq = 1;
}
```

- 响应体：

```protobuf
//GetAppSystemConfigResp 获取系统配置
message GetAppSystemConfigResp {
  CommonResp commonResp = 1;
  map<string, string> configs = 2;
}
```
