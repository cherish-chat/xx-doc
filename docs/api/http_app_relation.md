# http api 关系

## 1. RequestAddFriend: 请求添加好友

- 请求地址：`/v1/relation/requestAddFriend`
- 请求体：

```protobuf
message RequestAddFriendReq {
  Requester requester = 1;
  string to = 2;
  // 附加消息
  string message = 3;
}
```

- 响应体：

```protobuf
message RequestAddFriendResp {
  CommonResp commonResp = 1;
}
```

## 2. AcceptAddFriend: 接受添加好友请求

- 请求地址：`/v1/relation/acceptAddFriend`
- 请求体：

```protobuf
message AcceptAddFriendReq {
  Requester requester = 1;
  string applyUserId = 2; // 申请人id
  optional string requestId = 3; // 申请id
}
```

- 响应体：

```protobuf
message AcceptAddFriendResp {
  CommonResp commonResp = 1;
}
```

## 3. RejectAddFriend: 拒绝添加好友请求

- 请求地址：`/v1/relation/rejectAddFriend`
- 请求体：

```protobuf
message RejectAddFriendReq {
  Requester requester = 1;
  string applyUserId = 2; // 申请人id
  string requestId = 3; // 申请id
  bool block = 4; // 是否拉黑
}
```

- 响应体：

```protobuf
message RejectAddFriendResp {
  CommonResp commonResp = 1;
}
```

## 4. BlockUser: 拉黑用户

- 请求地址：`/v1/relation/blockUser`
- 请求体：

```protobuf
message BlockUserReq {
  Requester requester = 1;
  string userId = 2;
}
```

- 响应体：

```protobuf
message BlockUserResp {
  CommonResp commonResp = 1;
}
```

## 5. DeleteBlockUser: 取消拉黑用户

- 请求地址：`/v1/relation/deleteBlockUser`
- 请求体：

```protobuf
message DeleteBlockUserReq {
  Requester requester = 1;
  string userId = 2;
}
```

- 响应体：

```protobuf
message DeleteBlockUserResp {
  CommonResp commonResp = 1;
}
```

## 6. DeleteFriend: 删除好友

- 请求地址：`/v1/relation/deleteFriend`
- 请求体：

```protobuf
message DeleteFriendReq {
  Requester requester = 1;
  string userId = 2;
  bool block = 3; // 是否拉黑
}
```

- 响应体：

```protobuf
message DeleteFriendResp {
  CommonResp commonResp = 1;
}
```

## 7. GetFriendList: 获取好友列表

- 请求地址：`/v1/relation/getFriendList`
- 请求体：

```protobuf
//GetFriendListReq 获取好友列表
message GetFriendListReq {
  CommonReq commonReq = 1;
  // 分页
  Page page = 2;
  enum Opt {
    WithBaseInfo = 0; // 带用户的基本信息
    OnlyId = 1; // 只有用户id
    WithBaseInfoAndRemark = 2; // 带用户的基本信息和备注
  }
  Opt opt = 10;
}
```

- 响应体：

```protobuf
message GetFriendListResp {
  CommonResp commonResp = 1;
  repeated string ids = 2;
  map<string, UserBaseInfo> userMap = 3;
  map<string, string> remarkMap = 4; // key: targetId(userId) value: remark
}
```

## 8. GetMyFriendEventList: 获取我的好友事件列表

- 请求地址：`/v1/relation/getMyFriendEventList`
- 请求体：

```protobuf
message GetMyFriendEventListReq {
  CommonReq commonReq = 1;
  // 分页
  string pageIndex = 2; // 上次请求的pageIndex 第一次请求传空
}
```

- 响应体：

```protobuf
enum RequestAddFriendStatus {
  // 未处理
  Unhandled = 0;
  // 已同意
  Agreed = 1;
  // 已拒绝
  Refused = 2;
}
message RequestAddFriendExtra {
  string userId = 1;
  string content = 2;
}
message FriendEvent {
  // 发起人
  string fromUserId = 1;
  // 接收人
  string toUserId = 2;
  // 另一个人的用户信息
  UserBaseInfo otherUserInfo = 3;
  // 申请状态
  RequestAddFriendStatus status = 4;
  // 申请时间
  string createTime = 5;
  // 更新时间
  string updateTime = 6;
  // 附加信息
  RequestAddFriendExtra extra = 7;
}
message GetMyFriendEventListResp {
  CommonResp commonResp = 1;
  repeated FriendEvent friendNotifyList = 2;
  string pageIndex = 3; // 下次请求的pageIndex
}
```

## 9. UpdateUserRemark: 更新用户备注

- 请求地址：`/v1/relation/updateUserRemark`
- 请求体：

```protobuf
//UpdateUserRemarkReq 更新好友备注
message UpdateUserRemarkReq {
  CommonReq commonReq = 1;
  string targetId = 2;
  string remark = 3;
}
```

- 响应体：
```protobuf
message UpdateUserRemarkResp {
  CommonResp commonResp = 1;
}
```
