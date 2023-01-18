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
