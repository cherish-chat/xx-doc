# http api 群组

## 1. CreateGroup: 创建群组

- 请求地址：`/v1/group/createGroup`
- 请求体：

```protobuf
message CreateGroupReq {
  Requester requester = 1;
  // 拉人进群
  repeated string members = 2;
  // 群名称(可选参数)
  optional string name = 3;
  // 群头像(可选参数)
  optional string avatar = 4;
}
```

- 响应体：

```protobuf
message CreateGroupResp {
  CommonResp commonResp = 1;
  // 群ID
  optional string groupId = 2;
}
```

## 2. GetMyGroupList: 获取我的群组列表

- 请求地址：`/v1/group/getMyGroupList`
- 请求体：

```protobuf
//GetMyGroupListReq 获取我的群聊列表
message GetMyGroupListReq {
  CommonReq commonReq = 1;
  // 分页
  Page page = 2;
  // 过滤
  message Filter {
    // 消息接收类型
    // 是否过滤掉群助手
    bool filterFold = 1;
    // 是否过滤已屏蔽的群
    bool filterShield = 2;
  }
  Filter filter = 3;
  enum Opt {
    // 默认
    DEFAULT = 0;
    // 只获取id
    ONLY_ID = 1;
  }
  // 获取选项
  Opt opt = 10;
}
```

- 响应体：

```protobuf
message GroupBaseInfo {
  string id = 1;
  string name = 2;
  string avatar = 3;
}

//GetMyGroupListResp 获取我的群聊列表
message GetMyGroupListResp {
  CommonResp commonResp = 1;
  // 群聊列表
  map<string, GroupBaseInfo> groupMap = 2;
  // ids
  repeated string ids = 3;
}
```

## 3. SetGroupMemberInfo: 设置群成员信息

- 请求地址：`/v1/group/setGroupMemberInfo`

- 请求体：

```protobuf
//SetGroupMemberInfoReq 设置群成员信息
message SetGroupMemberInfoReq {
  CommonReq commonReq = 1;
  // 群ID
  string groupId = 2;
  // 群成员ID
  string memberId = 3;
  // notice content 群里所有人都能收到的notice中的content
  string notice = 4;
  // 群备注
  optional string remark = 11;
  // 群角色
  optional GroupRole role = 12;
  // 解除禁言时间
  optional int64 unbanTime = 13;
  // 群备注
  optional string groupRemark = 14;
}
```

- 响应体：

```protobuf
//SetGroupMemberInfoResp 设置群成员信息
message SetGroupMemberInfoResp {
  CommonResp commonResp = 1;
}
```

## 4. GetGroupMemberInfo: 获取群成员信息

- 请求地址：`/v1/group/getGroupMemberInfo`

- 请求体：

```protobuf
//GetGroupMemberInfoReq 获取群成员信息
message GetGroupMemberInfoReq {
  CommonReq commonReq = 1;
  // 群ID
  string groupId = 2;
  // 群成员ID
  string memberId = 3;
}
```

- 响应体：

```protobuf
message GroupMemberInfo {
  // 群id
  string groupId = 1;
  // 群成员id
  string memberId = 2;
  // 群内显示的昵称
  string remark = 3;
  // 群聊的备注
  string groupRemark = 4;
  // 置顶选项
  bool top = 5;
  // 免打扰选项
  bool disturb = 6;
  // 免打扰选项更多设置
  GroupDisturbOpt disturbMore = 7;
  // 聊天背景图
  string chatBg = 8;
  // 群角色
  GroupRole role = 9;
}

//GetGroupMemberInfoResp 获取群成员信息
message GetGroupMemberInfoResp {
  CommonResp commonResp = 1;
  // 群成员信息
  GroupMemberInfo groupMemberInfo = 2;
}
```

## 5. SearchGroupsByKeyword: 搜索群组

- 请求地址：`/v1/group/searchGroupsByKeyword`
- 请求体：

```protobuf
message SearchGroupsByKeywordReq {
  CommonReq commonReq = 1;
  string keyword = 2;
}
```

- 响应体：

```protobuf
message SearchGroupsByKeywordResp {
  CommonResp commonResp = 1;
  repeated GroupBaseInfo groups = 2;
}
```
