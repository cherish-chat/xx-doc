# http api 通用proto消息体

## 1. 通用

### 1.1 CommonResp: 响应消息体

```protobuf
message CommonResp {
  enum Code {
    Success = 0;

    UnknownError = 1;  // 未知 error
    InternalError = 2; // 内部错误
    RequestError = 3;  // 请求错误
    AuthError = 4;     // 鉴权错误 // 应该退出登录
    ToastError = 5;    // toast 错误 只有 message
    AlertError = 7;    // alert 错误 只有一个确定按钮
    RetryError = 8;    // alert 错误 有一个取消按钮 和一个重试按钮
  }
  Code code = 1;
  optional string msg = 2;
  bytes data = 3;
}
```

### ~~1.2 CommonReq: 请求消息体~~

```protobuf
message CommonReq {}
```

### 1.3 IpRegion: ip地区

```protobuf
message IpRegion {
  string country = 1;
  string province = 2;
  string city = 3;
  string isp = 4;
}
```

### 1.4 Page: 分页

```protobuf
message Page {
  int32 page = 1;
  int32 size = 2;
}
``` 

## 2. 用户

### 2.1 Xb: 用户性别

```protobuf
enum XB {
  UnknownXB = 0;
  Male = 1;
  Female = 2;
}
```

### 2.2 Constellation: 星座

```protobuf
enum Constellation {
  UnknownConstellation = 0;
  Aries = 1;
  Taurus = 2;
  Gemini = 3;
  Cancer = 4;
  Leo = 5;
  Virgo = 6;
  Libra = 7;
  Scorpio = 8;
  Sagittarius = 9;
  Capricorn = 10;
  Aquarius = 11;
  Pisces = 12;
}
```

### 2.3 BirthdayInfo: 生日信息

```protobuf
message BirthdayInfo {
  int32 year = 1;
  int32 month = 2;
  int32 day = 3;
  int32 age = 4;
  Constellation constellation = 5;
}
```

### 2.4 LevelInfo: 等级信息

```protobuf
message LevelInfo {
  int32 level = 1;
  int32 exp = 2;
  // 下一级所需经验
  int32 nextLevelExp = 3;
}
```

### 2.5 UserBaseInfo: 用户基本信息

```protobuf
message UserBaseInfo {
  string id = 1;
  string nickname = 2;
  string avatar = 3;
  XB xb = 4;
  // 生日信息
  BirthdayInfo birthday = 5;
  // 最后一次连接 ip所在地
  IpRegion ipRegion = 6;
  int32 role = 7; // 0:普通用户 1:管理员/客服 2:游客
}
```

## 3. im

### 3.1 MsgNotifyOpt: 消息通知选项

```protobuf
//MsgNotifyOpt 消息通知选项
message MsgNotifyOpt {
  bool preview = 1; // 是否预览
  bool sound = 2; // 是否声音
  string soundName = 3; // 声音名称
  bool vibrate = 4; // 是否震动
}
```

## 4. 消息

### 4.1 ConvType: 会话类型

```protobuf
enum ConvType {
  SINGLE = 0; // 单聊
  GROUP = 1; // 群聊
}
```

### 4.2 ContentType: 消息类型

```protobuf
// 这是msgData的ContentType类型
enum ContentType {
  UNKNOWN = 0;
  TYPING = 1; // 正在输入
  TIP = 2; // 提示 灰色小字 比如：xxx加入群聊、xxx撤回了一条消息

  TEXT = 11; // 文本
  IMAGE = 12; // 图片
  AUDIO = 13; // 语音
  VIDEO = 14; // 视频
  FILE = 15; // 文件
  LOCATION = 16; // 位置
  CARD = 17; // 名片
  MERGE = 18; // 合并
  EMOJI = 19; // 表情
  COMMAND = 20; // 命令
  RICH_TEXT = 21; // 富文本
  MARKDOWN = 22; // markdown

  CUSTOM = 100; // 自定义消息
}

// 这是noticeData的ContentType类型
enum NoticeType {
  INVALID = 0;
  // 已读
  READ = 1;
  // 编辑
  EDIT = 2;
}
```

### 4.3 MsgData: 消息体

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
```

### 4.4 MsgDataList: 消息列表

```protobuf
message MsgDataList {
  repeated MsgData msgDataList = 1;
}
```

### 4.5 NoticeData: 通知消息体

```protobuf
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
