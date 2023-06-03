# http api IM

## 1. GetAppSystemConfig: 获取应用系统配置

- 请求地址：`/v1/appmgmt/white/appGetAllConfig`
- 请求体：

```protobuf
//AppGetAllConfigReq 获取所有配置
message AppGetAllConfigReq {
  CommonReq commonReq = 1;
}
```

- 响应体：

```protobuf
//AppGetAllConfigResp 获取所有配置
message AppGetAllConfigResp {
  CommonResp commonResp = 1;
  map<string, string> configMap = 2;
}
```

- configMap kv配置

| key                                      | 描述                   | value  | 描述             |
|------------------------------------------|----------------------|--------|----------------|
| friend_max_count                         | 好友上限                 | number |                |
| avatars_default                          | 默认头像                 | string |                |
| register.allow.$platform                 | 是否允许`$platform`注册    | bool   | 0是false 1是true |
| register.invite_code.show                | 注册时是否显示邀请码           | bool   |                |
| register.invite_code.required            | 注册时邀请码是否必填           | bool   |                |
| register.avatar.show                     | 注册时是否显示头像            | bool   |                |
| register.avatar.required                 | 注册时头像是否必填            | bool   |                |
| register.nickname.show                   | 注册时是否显示昵称            | bool   |                |
| register.nickname.required               | 注册时昵称是否必填            | bool   |                |
| register.mobile.required                 | 注册时手机号是否必填           | bool   |                |
| register.mobile.sms                      | 注册时是否需要短信验证          | bool   |                |
| login.allow_guest.$platform              | 是否允许游客在`$platform`登录 | bool   |                |
| friend.add.user                          | 普通用户能否添加普通用户为好友      | bool   |                |
| friend.add.service                       | 普通用户能否添加客服为好友        | bool   |                |
| friend.add.guest                         | 普通用户能否添加游客为好友        | bool   |                |
| friend.add.robot                         | 普通用户能否添加机器人为好友       | bool   |                |
| group.create.user                        | 普通用户能否创建群聊           | bool   |                |
| group.create.service                     | 普通用户能否创建客服群聊         | bool   |                |
| group.create.guest                       | 普通用户能否创建游客群聊         | bool   |                |
| message.show.online_status               | 消息页面是否展示在线状态         | bool   |                |
| message.show.read_status                 | 消息页面是否展示已读状态         | bool   |                |
| message.show.typing_status               | 消息页面是否展示正在输入状态       | bool   |                |
| message.show.chat_log_button             | 消息页面是否展示聊天记录按钮       | bool   |                |
| message.message_limit.time               | 消息发送频率限制时间           | number | 秒              |
| message.message_time_tip.time            | 消息时间间隔提示时间(居中提示)     | number | 秒              |
| message.same_message_limit.allow         | 是否允许发送相同内容的消息(刷屏)    | bool   |                |
| message.same_message_limit               | 相同消息发送上限             | number | 次              |
| message.same_message_limit.time          | 相同消息发送上限时间           | number | 秒              |
| message.message_length_limit.text        | 文本消息长度限制             | number | 字              |
| group.search.allow                       | 是否允许搜索群聊             | bool   |                |
| group.quit_user.allow                    | 是否允许普通用户退出群聊         | bool   |                |
| group.show_member.allow                  | 是否显示群成员              | bool   |                |
| group.owner_clear_screen.allow           | 是否允许群主清屏             | bool   |                |
| group.show_member_count.allow            | 是否显示群成员数量            | bool   |                |
| group.show_invite_msg.allow              | 是否显示群邀请消息            | bool   |                |
| group.show_fake_member_count.allow       | 是否显示群假成员数量           | bool   |                |
| group.show_group_info.user               | 普通用户是否显示群信息          | bool   |                |
| group.show_group_info.user.link_name     | 发现底导的名称              | string |                |
| group.show_group_info.user.link_show     | 是否展示发现底导             | bool   |                |
| group.show_group_info.user.link_url      | 发现底导的链接              | string |                |
| group.show_group_info.user.discover_name | 发现底导的名称              | string |                |
| group.show_group_info.user.discover_show | 是否展示发现底导             | bool   |                |

## 2. GetLatestVersion: 获取最新版本

- 请求地址：`/v1/appmgmt/white/appGetLatestVersion`
- 请求体：

```protobuf
//AppMgmtVersion app管理系统 版本
message AppMgmtVersion {
  string id = 1;
  string version = 2;
  string platform = 3;
  int32 type = 4; // 0: 不提示 1: 提示 2: 强制
  string content = 5;
  string downloadUrl = 6;
  int64 createdAt = 7;
  string createdAtStr = 8;
}

//App获取最新版本
message GetLatestVersionReq {
  CommonReq commonReq = 1;
}
```

- 响应体：

```protobuf
//App获取最新版本
message GetLatestVersionResp {
  CommonResp commonResp = 1;
  AppMgmtVersion appMgmtVersion = 2;
}
```

## 3. GetUploadInfo: 获取上传信息

- 请求地址：`/v1/appmgmt/getUploadInfo`
- 请求体：

```protobuf
//GetUploadInfoReq 获取上传信息
message GetUploadInfoReq {
  CommonReq commonReq = 1;
  string objectId = 2;
  // 过期秒数
  int32 expireSeconds = 3;
}
```

- 响应体：

```protobuf
//GetUploadInfoResp 获取上传信息
message GetUploadInfoResp {
  CommonResp commonResp = 1;
  string uploadUrl = 2;
  map<string, string> headers = 3;
}
```

### 如何使用响应中的uploadUrl和headers

#### 1. 使用PUT请求上传文件

- 构建http[PUT]请求

```go
req, err := http.NewRequest("PUT", uploadUrl, file) // 其中file为文件流
```

- 设置请求头

```go
for k, v := range headers {
req.Header.Set(k, v)
}
```

- 发送请求

```go
response, err := http.DefaultClient.Do(req)
```

- 响应码处理

```go
if response.StatusCode != http.StatusOK {
if response.StatusCode == http.Unauthorized { // 401
// token过期/无效
}
if response.StatusCode == http.NotFound { // 404
// url不存在 或 文件名不包含文件md5
}
if response.StatusCode == http.ServerError { // 500
// 服务器错误 可能没有设置正确的对象存储
}
} else {
// 上传成功
// 返回 {"url": ""}
}
```

#### 2. 使用POST表单上传文件

- 构建http[POST]请求

```go
req, err := http.NewRequest("POST", uploadUrl, nil)
```

- 设置请求头

```go
for k, v := range headers {
req.Header.Set(k, v)
}
```

- 设置表单

```go
formBuf := &bytes.Buffer{}
writer := multipart.NewWriter(formBuf)
fileWriter, _ := writer.CreateFormFile("file", "test.png")
_, _ = io.Copy(fileWriter, bytes.NewReader(b))
_ = writer.Close()
request.Body = io.NopCloser(formBuf)
request.Header.Set("Content-Type", writer.FormDataContentType())
```

- 发送请求

```go
response, err := http.DefaultClient.Do(req)
```

- 响应码处理

```go
if response.StatusCode != http.StatusOK {
if response.StatusCode == http.Unauthorized { // 401
// token过期/无效
}
if response.StatusCode == http.NotFound { // 404
// url不存在 或 文件名不包含文件md5
}
if response.StatusCode == http.ServerError { // 500
// 服务器错误 可能没有设置正确的对象存储
}
} else {
// 上传成功
// 返回 {"url": ""}
}
```

## 4. GetAllAppMgmtLink: 获取发现页的外链

- 请求地址：`/v1/appmgmt/getAllAppMgmtLink`
- 请求体：

```protobuf
//GetAllAppMgmtLinkReq 获取所有app发现链接
message GetAllAppMgmtLinkReq {
  CommonReq commonReq = 1;
  Page page = 2; // 分页，传page=1&size=9999即可
  map<string, string> filter = 3; // 不传即可
}
```

- 响应体：

```protobuf
//GetAllAppMgmtLinkResp 获取所有app发现链接
message GetAllAppMgmtLinkResp {
  CommonResp commonResp = 1;
  repeated AppMgmtLink appMgmtLinks = 2;
  int64 total = 3;
}
```

## 5. AppGetRichArticleList: 获取富文本文章列表

- 请求地址：`/v1/appmgmt/appGetRichArticleList`
- 请求体：

```protobuf
//AppGetRichArticleListReq app获取富文本文章列表
message AppGetRichArticleListReq {
  CommonReq commonReq = 1;
  Page page = 2;
}
```

- 响应体：

```protobuf
//AppMgmtRichArticle 富文本文章
message AppMgmtRichArticle {
  // 文章id
  string id = 1;
  // 文章标题
  string title = 2;
  // 富文本内容
  string content = 3;
  // 内容类型
  string contentType = 4; // example: text/html; text/markdown; text/plain; application/json
  // url地址
  string url = 5;
  // 是否启用
  bool isEnable = 6;
  // 创建时间
  int64 createdAt = 7;
  // 创建时间字符串
  string createdAtStr = 8;
  // 更新时间
  int64 updatedAt = 9;
  // 更新时间字符串
  string updatedAtStr = 10;
  // 排序
  int32 sort = 11;
}

//AppGetRichArticleListResp app获取富文本文章列表
message AppGetRichArticleListResp {
  CommonResp commonResp = 1;
  repeated AppMgmtRichArticle appMgmtRichArticles = 2;
  int64 total = 3;
}
```
