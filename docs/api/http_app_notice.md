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

## 2. 服务端推送通知都有什么

```protobuf
// 通知数据结构
message NoticeData {
  message Options {
    // 客户端是否需要保存消息
    bool storageForClient = 1;
    // 是否需要重新渲染会话
    bool updateConvMsg = 2;
    // 只推送在线用户一次
    bool onlinePushOnce = 3;
  }
  // 会话信息
  string convId = 1; // 会话id (notice:$noticeId)
  int32 unreadCount = 2; // 会话未读数
  // 未读数量是绝对值还是增量
  bool unreadAbsolute = 3;

  // 消息信息
  string noticeId = 11;
  string createTime = 12;
  string title = 13; // 消息标题(显示在会话列表)
  int32 contentType = 14; // 通知数据类型
  bytes content = 15; // 消息数据

  // 附加信息
  Options options = 21; // 通知选项

  // 扩展信息
  bytes ext = 31; // 扩展字段
}
```

### convId 解释

- notice:FriendNotice (好友通知, 用户A请求加好友B时, B会收到通知)

- notice:GroupNotice (群组通知, 用户A请求加群B时, B群主/管理员会收到通知)

- notice:SyncFriendList (同步好友列表通知, B同意添加A为好友, A会收到通知)

- notice:group@${groupId} (群组通知, 比如群信息改变、群成员改变、群通知更新等)
  > noticeId: UpdateGroupInfo、UpdateGroupMember、UpdateGroupNotice

- notice:user@${userId} (用户通知, 比如用户信息改变、用户发布朋友圈、用户在线状态变更等)
  > noticeId: UpdateUserInfo、UpdateUserPost、UpdateUserStatus 