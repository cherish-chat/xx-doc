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

## 7. SetSingleConvSetting: 设置单聊设置

- 请求地址：`/v1/relation/setSingleConvSetting`
- 请求体：

```protobuf
message SingleConvSetting {
  string convId = 1;
  string userId = 2;
  // 设为置顶
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
  // 屏蔽此人消息
  optional bool isShield = 9;
  // 聊天背景
  optional string chatBg = 10;
}

//设置单聊设置
message SetSingleConvSettingReq {
  CommonReq commonReq = 1;
  SingleConvSetting setting = 2;
}
```

- 响应体：

```protobuf
message SetSingleConvSettingResp {
  CommonResp commonResp = 1;
}
```

## 8. GetSingleConvSetting: 获取单聊设置

- 请求地址：`/v1/relation/getSingleConvSetting`
- 请求体：

```protobuf
//获取单聊设置
message GetSingleConvSettingReq {
  CommonReq commonReq = 1;
  string convId = 2;
  string userId = 3;
}
```

- 响应体：

```protobuf
message GetSingleConvSettingResp {
  CommonResp commonResp = 1;
  SingleConvSetting setting = 2;
}
```

## 9. GetFriendList: 获取好友列表

- 请求地址：`/v1/relation/getFriendList`
- 请求体：

```protobuf
message GetFriendListReq {
  Requester requester = 1;
  // 分页
  Page page = 2;
  enum Opt {
    WithBaseInfo = 0; // 带用户的基本信息
    OnlyId = 1; // 只有用户id
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
}
```

## 10. GetMyFriendEventList: 获取我的好友事件列表

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