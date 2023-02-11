# http api 用户

## 1. Login: 登录

- 请求地址：`/v1/user/white/login`
- 请求体：

```protobuf
message LoginReq {
  Requester requester = 1;
  string id = 2; // 用户id 只能是英文和数字_，长度为6-20
  string password = 3; // 密码 // md5 数据库中会存入该值加盐后的值
}
```

- 响应体：

```protobuf
message LoginResp {
  CommonResp commonResp = 1;
  // 是否是新用户
  bool isNewUser = 2;
  // token
  string token = 3; // 如果是新用户，token为空
}
```

## 2. ConfirmRegister: 确认注册

- 请求地址：`/v1/user/white/confirmRegister`
- 请求体：

```protobuf
message ConfirmRegisterReq {
  Requester requester = 1;
  string id = 2; // 用户id 只能是英文和数字_，长度为6-20
  string password = 3; // 密码 // md5 数据库中会存入该值加盐后的值
}
```

- 响应体：

```protobuf
message ConfirmRegisterResp {
  CommonResp commonResp = 1;
  string token = 2;
}
```

## 3. Register: 注册

- 请求地址：`/v1/user/white/register`
- 请求体：

```protobuf
message RegisterReq {
  CommonReq commonReq = 1;
  string id = 2; // 用户id 只能是英文和数字_，长度为6-20
  string password = 3; // 密码 // md5 数据库中会存入该值加盐后的值
  optional string nickname = 4;
  optional XB xb = 5;
  optional BirthdayInfo birthday = 6;
}
```

- 响应体：

```protobuf
message RegisterResp {
  CommonResp commonResp = 1;
  string token = 2;
  string userId = 3;
}
```

## 3. SearchUsersByKeyword: 使用关键词搜索用户

- 请求地址：`/v1/user/searchUsersByKeyword`
- 请求体：

```protobuf
message SearchUsersByKeywordReq {
  Requester requester = 1;
  string keyword = 2;
}
```

- 响应体：

```protobuf
message SearchUsersByKeywordResp {
  CommonResp commonResp = 1;
  repeated UserBaseInfo users = 2;
}
```

## 4. GetUserHome: 获取用户主页信息

- 请求地址：`/v1/user/getUserHome`
- 请求体：

```protobuf
message GetUserHomeReq {
  Requester requester = 1;
  string id = 2;
}
```

- 响应体：

```protobuf
message GetUserHomeResp {
  CommonResp commonResp = 1;
  string id = 2;
  string nickname = 3;
  string avatar = 4;
  XB xb = 5;
  BirthdayInfo birthday = 6;
  IpRegion ipRegion = 7;
  // 个性签名
  string signature = 8;
  // 等级信息
  LevelInfo levelInfo = 9;
}
```

## 5. GetUserSettings: 获取用户设置

- 请求地址：`/v1/user/getUserSettings`
- 请求体：

```protobuf
enum UserSettingKey {
  HowToAddFriend = 0; // 如何添加好友
  HowToAddFriend_NeedAnswerQuestionCorrectly_Question = 1; // 如何添加好友 需要回答的问题
  HowToAddFriend_NeedAnswerQuestionCorrectly_Answer = 2; // 如何添加好友 需要回答的问题的答案
}
message GetUserSettingsReq {
  Requester requester = 1;
  repeated UserSettingKey keys = 2;
}
```

- 响应体：

```protobuf
message UserSetting {
  string userId = 1;
  UserSettingKey key = 2;
  string value = 3;
}

message GetUserSettingsResp {
  CommonResp commonResp = 1;
  map<int32, UserSetting> settings = 2;
}
```

## 6. SetUserSettings: 更新用户设置

- 请求地址：`/v1/user/setUserSettings`
- 请求体：

```protobuf
message SetUserSettingsReq {
  Requester requester = 1;
  repeated UserSetting settings = 2;
}
```

- 响应体：

```protobuf
message SetUserSettingsResp {
  CommonResp commonResp = 1;
}
```
