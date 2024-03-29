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
    // 群信息+自己成员信息
    WITH_MY_MEMBER_INFO = 2;
  }
  // 获取选项
  Opt opt = 10;
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
  // 群角色
  GroupRole role = 4;
  // 解封时间
  int64 unbanTime = 5;
  // 用户基本信息
  optional UserBaseInfo userBaseInfo = 11;
}

message GroupBaseInfo {
  string id = 1;
  string name = 2;
  string avatar = 3;
  string owner = 4;

  int64 dismissTime = 5; // 解散时间 如果为0表示未解散 否则进入会话详情应该提示群已解散

  bool allMute = 11; // 是否全员禁言
  bool memberCanAddFriend = 12; // 群成员是否可以加好友

  optional GroupMemberInfo myMemberInfo = 31; // 自己在群里的信息
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

## 6. GetGroupHome: 获取群主页

- 请求地址：`/v1/group/getGroupHome`
- 请求体：

```protobuf
message GetGroupHomeReq {
  CommonReq commonReq = 1;
  // 群ID
  string groupId = 2;
}
```

- 响应体：

```protobuf
//GetGroupHomeResp 获取群聊首页
message GetGroupHomeResp {
  CommonResp commonResp = 1;
  // 群ID
  string groupId = 2;
  // 群名称
  string name = 3;
  // 群头像
  string avatar = 4;
  // 创建日期
  string createdAt = 5;
  // 成员人数
  int32 memberCount = 6;
  // 群介绍
  string introduction = 7;
  // 群主id
  string owner = 8;
  int64 dismissTime = 9; // 解散时间 如果为0表示未解散 否则进入会话详情应该提示群已解散

  // 群禁言
  bool allMute = 11;
  // 群禁止加好友
  bool memberCanAddFriend = 12;

  // 成员统计
  message MemberStatistics {
    // 统计标题
    string title = 1;
    // 人数
    int32 count = 2;
    // 占百分比
    int32 percentage = 3;
  }
  repeated MemberStatistics memberStatistics = 21;
}
```

## 7. GetGroupMemberList: 获取群成员列表

- 请求地址：`/v1/group/getGroupMemberList`
- 请求体：

```protobuf
//GetGroupMemberListReq 获取群成员列表
message GetGroupMemberListReq {
  CommonReq commonReq = 1;
  // 群ID
  string groupId = 2;
  // 分页
  Page page = 3;
  // Filter
  message GetGroupMemberListFilter {
    // 是否接受离线推送
    optional bool noDisturb = 1;
    // 只包含群主
    optional bool onlyOwner = 2;
    // 只包含管理员
    optional bool onlyAdmin = 3;
    // 只包含成员
    optional bool onlyMember = 4;
  }
  GetGroupMemberListFilter filter = 4; // 如果只包含群主和管理员 那么设置onlyOwner=true,onlyAdmin=true
  message GetGroupMemberListOpt {
    // 是否只获取id
    optional bool onlyId = 1;
  }
  GetGroupMemberListOpt opt = 5;
}
```

- 响应体：

```protobuf
enum GroupRole {
  // 普通成员
  MEMBER = 0;
  // 管理员
  MANAGER = 1;
  // 群主
  OWNER = 2;
}
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
  bool noDisturb = 6;
  // 聊天背景图
  string chatBg = 8;
  // 群角色
  GroupRole role = 9;
  // 解封时间
  int64 unbanTime = 10;
  // 消息通知是否预览
  bool preview = 11;
}

//GetGroupMemberListResp 获取群成员列表
message GetGroupMemberListResp {
  CommonResp commonResp = 1;
  // 群成员列表
  repeated GroupMemberInfo groupMemberList = 2;
}
```

## 8. ApplyToBeGroupMember: 申请加入群组

- 请求地址：`/v1/group/applyToBeGroupMember`
- 请求体：

```protobuf
//applyToBeGroupMember 申请加入群聊
message ApplyToBeGroupMemberReq {
  CommonReq commonReq = 1;
  // 群ID
  string groupId = 2;
  // 申请理由
  string reason = 3;
}
```

- 响应体：

```protobuf
message ApplyToBeGroupMemberResp {
  CommonResp commonResp = 1;
}
```

## 9. HandleGroupApply: 处理群申请

- 请求地址：`/v1/group/handleGroupApply`
- 请求体：

```protobuf
//handleGroupApply 处理群聊申请
message HandleGroupApplyReq {
  CommonReq commonReq = 1;
  // 申请ID
  string applyId = 2;
  // 处理结果
  GroupApplyHandleResult result = 3;
}
```

- 响应体：

```protobuf
message HandleGroupApplyResp {
  CommonResp commonResp = 1;
}
```

## 10. GetGroupApplyList: 获取群申请列表

- 请求地址：`/v1/group/getGroupApplyList`
- 请求体：

```protobuf
//getGroupApplyList 获取群聊申请列表
message GetGroupApplyListReq {
  CommonReq commonReq = 1;
  // 分页
  Page page = 2;
  // 过滤
  message Filter {
    // 申请状态
    optional GroupApplyHandleResult result = 1;
  }
  Filter filter = 3;
}
```

- 响应体：

```protobuf
message GroupApplyInfo {
  // 申请ID
  string applyId = 1;
  // 群ID
  string groupId = 2;
  // 申请人
  string userId = 3;
  // 申请状态
  GroupApplyHandleResult result = 4;
  // 申请理由
  string reason = 5;
  // 申请时间
  int64 applyTime = 6;
  string applyTimeStr = 7;
  // 处理时间
  int64 handleTime = 8;
  string handleTimeStr = 9;
  // 处理人
  string handleUserId = 10;

  // 申请人的baseInfo
  optional UserBaseInfo userBaseInfo = 11;
  // 处理人的baseInfo
  optional UserBaseInfo handleUserBaseInfo = 12;

  // 群的baseInfo
  optional GroupBaseInfo groupBaseInfo = 13;
}
message GetGroupApplyListResp {
  CommonResp commonResp = 1;
  // 申请列表
  repeated GroupApplyInfo groupApplyList = 2;
  // 总数
  int64 total = 3;
}
```

## 11. KickGroupMember: 踢出群成员(可以踢自己)

- 请求地址：`/v1/group/kickGroupMember`
- 请求体：

```protobuf
//KickGroupMemberReq 踢出群成员
message KickGroupMemberReq {
  CommonReq commonReq = 1;
  // 群ID
  string groupId = 2;
  // 群成员ID
  string memberId = 3;
}
```

- 响应体：

```protobuf
//KickGroupMemberResp 踢出群成员
message KickGroupMemberResp {
  CommonResp commonResp = 1;
}
```

## 12. ReportGroup: 举报群

- 请求地址：`/v1/group/reportGroup`
- 请求体：

```protobuf
message ReportGroupReq {
  CommonReq commonReq = 1;
  string groupId = 2;
  string reason = 3;
}
```

- 响应体：

```protobuf
// ReportGroupResp 举报群
message ReportGroupResp {
  CommonResp commonResp = 1;
}
```

## 13. EditGroupInfo: 编辑群信息

- 请求地址：`/v1/group/editGroupInfo`
- 请求体：

```protobuf
//EditGroupInfoReq 编辑群资料
message EditGroupInfoReq {
  CommonReq commonReq = 1;
  // 群ID
  string groupId = 2;
  // 群名称
  optional string name = 3;
  // 群头像
  optional string avatar = 4;
  // 群介绍
  optional string introduction = 5;
  // 群禁言
  optional bool allMute = 11;
  // 群禁止加好友
  optional bool memberCanAddFriend = 12;
}
```

- 响应体：

```protobuf
//EditGroupInfoResp 编辑群资料
message EditGroupInfoResp {
  CommonResp commonResp = 1;
}
```

## 14. BatchKickGroupMember: 批量踢出群成员

- 请求地址：`/v1/group/batchKickGroupMember`
- 请求体：

```protobuf
//BatchKickGroupMemberReq 批量踢出群成员
message BatchKickGroupMemberReq {
  CommonReq commonReq = 1;
  // 群ID
  string groupId = 2;
  // 群成员ID
  repeated string memberIds = 3;
}
```

- 响应体：

```protobuf
//BatchKickGroupMemberResp 批量踢出群成员
message BatchKickGroupMemberResp {
  CommonResp commonResp = 1;
}
```

## 15. InviteFriendToGroup: 邀请好友加入群

- 请求地址：`/v1/group/inviteFriendToGroup`
- 请求体：

```protobuf
//InviteFriendToGroupReq 邀请好友进群
message InviteFriendToGroupReq {
  CommonReq commonReq = 1;
  // 群ID
  string groupId = 2;
  // 好友ID
  repeated string friendIds = 3;
}
```

- 响应体：

```protobuf
//InviteFriendToGroupResp 邀请好友进群
message InviteFriendToGroupResp {
  CommonResp commonResp = 1;
}
```

## 16. ResetGroupInfo: 重设群信息

- 请求地址：`/v1/group/resetGroupInfo`
- 请求体：

```protobuf
//ResetGroupInfoReq 编辑群资料
message ResetGroupInfoReq {
  CommonReq commonReq = 1;
  // 群ID
  string groupId = 2;
  // 群名称
  string name = 3;
  // 群头像
  string avatar = 4;
  // 群介绍
  string introduction = 5;
  // 群禁言
  bool allMute = 11;
  // 群禁止加好友
  bool memberCanAddFriend = 12;
}
```

- 响应体：

```protobuf
//EditGroupInfoResp 编辑群资料
message EditGroupInfoResp {
  CommonResp commonResp = 1;
}
```

## 17. SearchGroupMember: 搜索群成员

- 请求地址：`/v1/group/searchGroupMember`
- 请求体：

```protobuf
//SearchGroupMemberReq 搜索群成员
message SearchGroupMemberReq {
  CommonReq commonReq = 1;
  // 群ID
  string groupId = 2;
  // 关键词
  string keyword = 3;
  // 或在userIds中搜索
  repeated string orInUserIds = 4;
  // 分页
  Page page = 5;
}
```

- 响应体：

```protobuf
//SearchGroupMemberResp 搜索群成员
message SearchGroupMemberResp {
  CommonResp commonResp = 1;
  // 群成员列表
  repeated GroupMemberInfo groupMemberList = 2;
}
```

## 18. BanGroupMember: 禁言群成员

- 请求地址：`/v1/group/banGroupMember`
- 请求体：

```protobuf
//BanGroupMemberReq 禁言群成员
message BanGroupMemberReq {
  CommonReq commonReq = 1;
  // 群ID
  string groupId = 2;
  // 群成员ID
  string memberId = 3;
  // 解除禁言时间
  int64 unbanTime = 4;
}
```

- 响应体：

```protobuf
//BanGroupMemberResp 禁言群成员
message BanGroupMemberResp {
  CommonResp commonResp = 1;
}
```

## 19. UnbanGroupMember: 解除禁言群成员

- 请求地址：`/v1/group/unbanGroupMember`
- 请求体：

```protobuf
//UnbanGroupMemberReq 解除禁言群成员
message UnbanGroupMemberReq {
  CommonReq commonReq = 1;
  // 群ID
  string groupId = 2;
  // 群成员ID
  string memberId = 3;
}
```

- 响应体：

```protobuf
//UnbanGroupMemberResp 解除禁言群成员
message UnbanGroupMemberResp {
  CommonResp commonResp = 1;
}
```

// 20. SetGroupMemberRole: 设置群成员角色

- 请求地址：`/v1/group/setGroupMemberRole`
- 请求体：

```protobuf
//SetGroupMemberRoleReq 设置群成员角色
message SetGroupMemberRoleReq {
  CommonReq commonReq = 1;
  // 群ID
  string groupId = 2;
  // 群成员ID
  string memberId = 3;
  // 群角色
  GroupRole role = 4;
}
```

- 响应体：

```protobuf
//SetGroupMemberRoleResp 设置群成员角色
message SetGroupMemberRoleResp {
  CommonResp commonResp = 1;
}
```
