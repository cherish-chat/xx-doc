# http api 接口-消息

## 0.MsgContentType 消息类型

```protobuf
enum MsgContentType {
  unknown = 0; // 未知类型
  typing = 1; // 正在输入
  tip = 2; // 提示

  text = 11; // 文本
  image = 12; // 图片
  audio = 13; // 语音
  video = 14; // 视频
  file = 15; // 文件
  location = 16; // 位置
  card = 17; // 名片
  merge = 18; // 合并
  emoji = 19; // 表情
  command = 20; // 命令
  richText = 21; // 富文本
  markdown = 22; // markdown

  custom = 100; // 自定义消息
}
```

## 1. SendMsg: 发送消息

- 请求地址：`/v1/msg/sendMsgList`
- 请求体：

```protobuf

message MsgData {
  message OfflinePush {
    string title = 1;
    string content = 2;
    string payload = 3;
  }
  message Options {
    // 服务端是否需要保存消息
    bool storageForServer = 1;
    // 客户端是否需要保存消息
    bool storageForClient = 2;
    // 是否需要解密 （端对端加密技术，服务端无法解密）
    bool needDecrypt = 3;
    // 是否需要离线推送
    bool offlinePush = 4;
    // 是否需要重新渲染会话
    bool updateConvMsg = 5;
    // 消息是否需要计入未读数
    bool updateUnreadCount = 6;
  }
  string clientMsgId = 1; // 客户端消息id 一般是uuid 客户端聊天记录表的主键
  string serverMsgId = 2; // 服务端消息id
  string clientTime = 3; // 客户端时间戳 13位
  string serverTime = 4; // 服务端时间戳 13位

  string senderId = 11; // 发送者id
  bytes senderInfo = 12; // 发送者信息

  string convId = 21; // 会话id (单聊时 single:user1-user2，群聊时 group:groupId，通知号 notice:noticeId)
  repeated string atUsers = 22;   // 强提醒用户id列表 用户不在线时，会收到离线推送，除非用户屏蔽了该会话 如果需要提醒所有人，可以传入"all"

  int32 contentType = 31; // 消息内容类型
  bytes content = 32; // 消息内容
  string seq = 33; // 消息序号 会话内唯一且递增

  Options options = 41; // 消息选项 
  OfflinePush offlinePush = 42; // 离线推送

  bytes ext = 101;
}

message SendMsgListReq {
  repeated MsgData msgDataList = 1;
  // options
  // 1. 延迟时间（秒） 不得大于 864000秒 也就是10天
  optional int32 deliverAfter = 2;
  CommonReq commonReq = 11;
}
```

- 响应体：

```protobuf
message SendMsgListResp {}
```

## 2. BatchGetMsgListByConvId: 批量获取会话消息列表

- 请求地址：`/v1/msg/batchGetMsgListByConvId`
- 请求体：

```protobuf
message BatchGetMsgListByConvIdReq {
  message Item {
    string convId = 1;
    repeated string seqList = 2;
  }
  repeated Item items = 1;
  bool push = 2;
  CommonReq commonReq = 11;
}
```

- 响应体：

```protobuf
message GetMsgListResp {
  repeated MsgData msgDataList = 1;
  CommonResp commonResp = 11;
}
```

## 3. getMsgById: 通过id获取消息

- 请求地址：`/v1/msg/getMsgById`
- 请求体：

```protobuf
message GetMsgByIdReq {
  optional string serverMsgId = 1;
  optional string clientMsgId = 2;
  bool push = 3;
  CommonReq commonReq = 11;
}
```

- 响应体：

```protobuf
message GetMsgByIdResp {
  MsgData msgData = 1;
  CommonResp commonResp = 11;
}
```

## 4. batchGetConvSeq: 批量获取会话的消息seq

- 请求地址：`/v1/msg/batchGetConvSeq`
- 请求体：

```protobuf
message BatchGetConvSeqReq {
  repeated string convIdList = 1;
  CommonReq commonReq = 11;
}
```

- 响应体：

```protobuf
message BatchGetConvSeqResp {
  message ConvSeq {
    string convId = 1;
    string minSeq = 2;
    string maxSeq = 3;
    string updateTime = 4;
  }
  map<string, ConvSeq> convSeqMap = 1;
  CommonResp commonResp = 11;
}
```

## 5. sendReadMsg: 发送已读消息

- 请求地址：`/v1/msg/sendReadMsg`
- 请求体：

```protobuf
message ReadMsgReq {
  string senderId = 1;
  string convId = 2;
  string seq = 3;
  bytes noticeContent = 4;
  CommonReq commonReq = 11;
}
```

- 响应体：

```protobuf
message ReadMsgResp {
  CommonResp commonResp = 1;
}
```

## 6. sendEditMsg: 编辑已发送的消息

- 请求地址：`/v1/msg/sendEditMsg`
- 请求体：

```protobuf
message EditMsgReq {
  string senderId = 1;
  string serverMsgId = 2;
  int32 contentType = 3;
  bytes content = 4;
  bytes ext = 5;
  bytes noticeContent = 6;
  CommonReq commonReq = 11;
}
```

- 响应体：

```protobuf
message EditMsgResp {
  CommonResp commonResp = 1;
}
```
