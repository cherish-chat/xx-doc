# http api 接口-通知

## 1. AckNoticeData: 确认消费通知

- 请求地址：`/v1/notice/ackNoticeData`
- 请求体：

```protobuf
//AckNoticeDataReq 确认通知数据
message AckNoticeDataReq {
  CommonReq commonReq = 1;
  string noticeId = 2;
  bool success = 3; // 是否消费成功
}
```

- 响应体：

```protobuf
message AckNoticeDataResp {
  CommonResp commonResp = 1;
}
```
