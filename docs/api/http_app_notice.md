# http api 接口-通知

## 1. AckNoticeData: 确认消费通知

- 请求地址：`/v1/notice/ackNoticeData`
- 请求体：

```protobuf
//AckNoticeDataReq 确认通知数据
message AckNoticeDataReq {
  string convId = 1;
  string noticeId = 2;
  CommonReq commonReq = 11;
}
```

- 响应体：

```protobuf
message AckNoticeDataResp {}
```

## 2. 服务端推送通知都有什么

```protobuf
// 通知数据结构
message NoticeData {
  message Options {
    // 客户端是否需要保存消息
    bool storageForClient = 1;
    // 是否需要重新渲染会话
    bool updateConvNotice = 2;
  }
  // 会话信息
  string convId = 1; // 会话id

  // 消息信息
  string noticeId = 11;
  string createTime = 12;
  string title = 13; // 消息标题(显示在会话列表)
  int32 contentType = 14; // 通知数据类型
  bytes content = 15; // 消息数据

  // 附加信息
  Options options = 21; // 通知选项

  // 扩展信息
  bytes ext = 101; // 扩展字段
}
```

### contentType

| 值     | 描述      | content                                                                                                                                   |
|-------|---------|-------------------------------------------------------------------------------------------------------------------------------------------|
| 1     | 已读      | 客户端定义                                                                                                                                     |
| 2     | 已编辑     | 客户端定义                                                                                                                                     |
| ----- | -----   | ----命令-----                                                                                                                               |
| 101   | 同步好友列表  | 无                                                                                                                                         |
| 102   | 同步会话设置  | {convIds: ["xxx", "xxx"], userId: 更新了谁的会话设置}                                                                                              |
| ----- | -----   | ----好友-----                                                                                                                               |
| 201   | 好友信息更新了 | {userId: "", updateMap: {}}                                                                                                               |
| ----- | -----   | ----群-----                                                                                                                                |
| 301   | 群成员离开   | {groupId: "", tip: ""}                                                                                                                    |
| 302   | 群创建     | {groupId: ""}                                                                                                                             |
| 303   | 群成员加入   | {groupId: "", memberId: ""}                                                                                                               |
| 304   | 群解散     | {groupId: ""}                                                                                                                             |
| 305   | 群成员信息更新 | {groupId: "", memberId: "", updateMap: {}}                                                                                                |
| 306   | 设置群信息   | {groupId: ""}                                                                                                                             |
| 307   | 恢复群     | {groupId: ""}                                                                                                                             |
| 308   | 更新群信息     | {groupId: "", updateMap: {}}                                                                                                                             |
| ----- | -----   | ----新群通知-----                                                                                                                             |
| 401   | 申请加群    | {applyId: "", groupId: "", userId: "", result: 0/1/2, reason: "", applyTime: 1234567890123, handleTime: 1234567890123, handleUserId: "" } |
| ----- | -----   | ----新好友通知-----                                                                                                                            |
| 501   | 申请加好友   | -                                                                                                                                         |