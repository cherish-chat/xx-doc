# http api 接口-通知

## 1. AckNoticeData: 确认消费通知

- 请求地址：`/v1/notice/ackNoticeData`
- 请求体：

```protobuf
//AckNoticeDataReq 确认通知数据
message AckNoticeDataReq {
  string convId = 1;
  string noticeId = 2;
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
    bool updateConvMsg = 2;
  }
  // 会话信息
  string convId = 1; // 会话id (notice:$noticeId)

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

### convId

- command: 表示是命令消息
- friendMember: 表示是好友成员通知
- groupMember: 表示是群成员通知
- `friend@`为前缀: 表示是好友通知
- `group@`为前缀: 表示是群通知

### contentType

```dart
class NoticeContentType {
  // command
  static const int syncFriendList = 101; // 同步好友列表

  // friend@
  static const int updateUserInfo = 201; // 好友更新了用户信息

  // group@
  static const int groupMemberLeave = 301; // 群成员离开
  static const int createGroup = 302; // 创建群
  static const int newGroupMember = 303; // 新成员加入
  static const int dismissGroup = 304; // 解散群
  static const int setGroupMemberInfo = 305; // 设置群成员信息

  // groupMember
  static const int applyToBeGroupMember = 401; // 申请加入群 
}
```

### 示例

#### 1. dart

```dart
class NoticeConvId {
  static const String command = 'command';
  static const String friendMember = 'friendMember';
  static const String groupMember = 'groupMember';
  static const String friendPrefix = "friend@";
  static const String groupPrefix = "group@";
}

class NoticeContentType {
  // command
  static const int syncFriendList = 101;

  // friend@
  static const int updateUserInfo = 201;

  // group@
  static const int groupMemberLeave = 301;
  static const int createGroup = 302;
  static const int newGroupMember = 303;
  static const int dismissGroup = 304;
  static const int setGroupMemberInfo = 305;

  // groupMember
  static const int applyToBeGroupMember = 401;
}

class NoticeLogic implements ImToolWorker {
  Future<bool> onReceive(NoticeModel notice) async {
    final convId = receiverNid(notice.convId);
    switch (convId) {
      case NoticeConvId.command:
        return await _onCommand(convId, notice);
      case NoticeConvId.friendMember:
        return await _onFriendMember(convId, notice);
      case NoticeConvId.groupMember:
        return await _onGroupMember(convId, notice);
      default:
        if (convId.startsWith(NoticeConvId.friendPrefix)) {
          // 好友
          String friendId = convId.substring(NoticeConvId.friendPrefix.length);
          return await _onFriendNotice(friendId, notice);
        } else if (convId.startsWith(NoticeConvId.groupPrefix)) {
          // 群组
          String groupId = convId.substring(NoticeConvId.groupPrefix.length);
          return await _onGroupNotice(groupId, notice);
        } else {
          // 其他
          // 打印notice内容
          printInfo(
              info:
                  'onReceive notice.content: ${notice.content}, notice.contentType: ${notice.contentType}, notice.convId: ${notice.convId}');
        }
    }
    return true;
  }
}
```