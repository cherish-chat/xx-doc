# http api 用户

## 1. Login: 登录

- 请求地址：`/v1/user/white/login`
- 请求体：

```protobuf
message LoginReq {
  CommonReq commonReq = 1;
  string id = 2; // 用户id 只能是英文和数字_，长度为6-20
  string password = 3; // 密码 // md5 数据库中会存入该值加盐后的值
  optional string captchaCode = 4; // 图形验证码
  optional string captchaId = 5; // 图形验证码id
}
```

- 响应体：

```protobuf
message LoginResp {
  CommonResp commonResp = 1;
  // 是否是新用户
  bool isNewUser = 2;// 如果是新用户，token为空
  // token
  string token = 3; 
  // 是否已注销
  bool isDestroyed = 5; // 如果已注销，token为空
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
  optional string nickname = 4; // 昵称
  optional XB xb = 5; // 性别
  optional BirthdayInfo birthday = 6; // 生日
  optional string invitationCode = 7; // 邀请码
  optional string mobile = 8; // 手机号
  optional string mobileCountryCode = 9; // 手机号国家码
  optional string smsCode = 10; // 短信验证码
  optional string avatar = 11; // 头像
  optional string captchaCode = 12; // 图形验证码
  optional string captchaId = 13; // 图形验证码id
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

## 4. SearchUsersByKeyword: 使用关键词搜索用户

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

## 5. GetUserHome: 获取用户主页信息

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

## 6. GetUserSettings: 获取用户设置

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

## 7. SetUserSettings: 更新用户设置

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

## 8. UpdateUserPassword: 更新用户密码

- 请求地址：`/v1/user/updateUserPassword`
- 请求体：

```protobuf
message UpdateUserPasswordReq {
  CommonReq commonReq = 1;
  string oldPassword = 2;
  string newPassword = 3;
}
```

- 响应体：

```protobuf
message UpdateUserPasswordResp {
  CommonResp commonResp = 1;
}
```

## 9. SendSms: 发送短信

- 请求地址：`/v1/user/white/sendSms`
- 请求体：

```protobuf
//SendSmsReq 发送短信请求
message SendSmsReq {
  CommonReq commonReq = 1;
  string phone = 2;
  optional string countryCode = 3; // +86
  // 业务场景
  string scene = 4;
  // 失效时间 分钟 默认5
  optional int32 expireMinute = 5;
}
```

- 响应体：

```protobuf
//SendSmsResp 发送短信响应
message SendSmsResp {
  CommonResp commonResp = 1;
}
```

## 10. VerifySms: 验证短信

- 请求地址：`/v1/user/white/verifySms`
- 请求体：

```protobuf
//VerifySmsReq 验证短信请求
message VerifySmsReq {
  CommonReq commonReq = 1;
  string phone = 2;
  optional string countryCode = 3;
  // 业务场景
  string scene = 4;
  // 验证码
  string code = 5;
  // 验证后是否删除
  bool delete = 6;
}

```

- 响应体：

```protobuf
//VerifySmsResp 验证短信响应
message VerifySmsResp {
  CommonResp commonResp = 1;
}
```

## 11. UpdateUserInfo: 更新用户信息

- 请求地址：`/v1/user/updateUserInfo`
- 请求体：

```protobuf
message UpdateUserInfoReq {
  CommonReq commonReq = 1;
  optional string nickname = 2;
  optional string avatar = 3;
  optional string signature = 4;
}
```

- 响应体：

```protobuf
message UpdateUserInfoResp {
  CommonResp commonResp = 1;
}
```

## 12. ResetPassword: 使用手机号重置密码

- 请求地址：`/v1/user/white/resetPassword`
- 请求体：

```protobuf
// 忘记密码重置密码
//ResetPasswordReq 重置密码
message ResetPasswordReq {
  CommonReq commonReq = 1;
  // 手机号
  string mobile = 2;
  optional string mobileCountryCode = 3;
  // 验证码
  string smsCode = 4;
  // 新密码
  string newPassword = 5;
}
```

- 响应体：

```protobuf
message ResetPasswordResp {
  CommonResp commonResp = 1;
}
```

## 13. ReportUser: 举报用户

- 请求地址：`/v1/user/reportUser`
- 请求体：

```protobuf
// 举报用户
// ReportUserReq 举报用户请求
message ReportUserReq {
  CommonReq commonReq = 1;
  string userId = 2;
  string reason = 3;
}
```

- 响应体：

```protobuf
// ReportUserResp 举报用户响应
message ReportUserResp {
  CommonResp commonResp = 1;
}
```

## 14. DestroyAccount: 账号注销

- 请求地址：`/v1/user/destroyAccount`
- 请求体：

```protobuf
//DestroyAccountReq 注销账号请求
message DestroyAccountReq {
  CommonReq commonReq = 1;
}
```

- 响应体：

```protobuf
//DestroyAccountResp 注销账号响应
message DestroyAccountResp {
  CommonResp commonResp = 1;
}
```

## 15. RecoverAccount: 账号恢复

- 请求地址：`/v1/user/white/recoverAccount`
- 请求体：

```protobuf
//RecoverAccountReq 恢复账号请求
message RecoverAccountReq {
  CommonReq commonReq = 1;
  string userId = 2;
}
```

- 响应体：

```protobuf
//RecoverAccountResp 恢复账号响应
message RecoverAccountResp {
  CommonResp commonResp = 1;
}
```

## 16. GetCaptchaCode: 获取图形验证码

- 请求地址：`/v1/user/white/getCaptchaCode`
- 请求体：

```protobuf
//GetCaptchaCodeReq 获取图形验证码请求
message GetCaptchaCodeReq {
  CommonReq commonReq = 1;
  // 业务场景
  string scene = 2;
  // 失效时间 分钟 默认5
  optional int32 expireMinute = 3;
}
```

- 响应体：

```protobuf
//GetCaptchaCodeResp 获取图形验证码响应
message GetCaptchaCodeResp {
  CommonResp commonResp = 1;
  bytes captcha = 2;
  string captchaId = 3;
}
```

## 17. VerifyCaptchaCode: 验证图形验证码

- 请求地址：`/v1/user/white/verifyCaptchaCode`
- 请求体：

```protobuf
//VerifyCaptchaCodeReq 验证图形验证码请求
message VerifyCaptchaCodeReq {
  CommonReq commonReq = 1;
  string captchaId = 2;
  // 业务场景
  string scene = 3;
  // 验证码
  string code = 4;
  // 验证后是否删除 客户端不要删除 因为服务端会再次验证
  bool delete = 5;
}
```

- 响应体：

```protobuf
//VerifyCaptchaCodeResp 验证图形验证码响应
message VerifyCaptchaCodeResp {
  CommonResp commonResp = 1;
}
```
