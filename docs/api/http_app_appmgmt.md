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

| key                                  | 描述                   | value  | 描述             |
|--------------------------------------|----------------------|--------|----------------|
| friend_max_count                     | 好友上限                 | number |                |
| avatars_default                      | 默认头像                 | string |                |
| register.allow.$platform             | 是否允许`$platform`注册    | bool   | 0是false 1是true |
| register.invite_code.show            | 注册时是否显示邀请码           | bool   |                |
| register.invite_code.required        | 注册时邀请码是否必填           | bool   |                |
| register.avatar.show                 | 注册时是否显示头像            | bool   |                |
| register.avatar.required             | 注册时头像是否必填            | bool   |                |
| register.nickname.show               | 注册时是否显示昵称            | bool   |                |
| register.nickname.required           | 注册时昵称是否必填            | bool   |                |
| register.mobile.required             | 注册时手机号是否必填           | bool   |                |
| register.mobile.sms                  | 注册时是否需要短信验证          | bool   |                |
| login.allow_guest.$platform          | 是否允许游客在`$platform`登录 | bool   |                |
| friend.add.user                      | 普通用户能否添加普通用户为好友      | bool   |                |
| friend.add.service                   | 普通用户能否添加客服为好友        | bool   |                |
| friend.add.guest                     | 普通用户能否添加游客为好友        | bool   |                |
| friend.add.robot                     | 普通用户能否添加机器人为好友       | bool   |                |
| group.create.user                    | 普通用户能否创建群聊           | bool   |                |
| group.create.service                 | 普通用户能否创建客服群聊         | bool   |                |
| group.create.guest                   | 普通用户能否创建游客群聊         | bool   |                |
| message.show.online_status           | 消息页面是否展示在线状态         | bool   |                |
| message.show.read_status             | 消息页面是否展示已读状态         | bool   |                |
| message.show.typing_status           | 消息页面是否展示正在输入状态       | bool   |                |
| message.show.chat_log_button         | 消息页面是否展示聊天记录按钮       | bool   |                |
| message.message_limit.time           | 消息发送频率限制时间           | number | 秒              |
| message.message_time_tip.time        | 消息时间间隔提示时间(居中提示)     | number | 秒              |
| message.same_message_limit.allow     | 是否允许发送相同内容的消息(刷屏)    | bool   |                |
| message.same_message_limit           | 相同消息发送上限             | number | 次              |
| message.same_message_limit.time      | 相同消息发送上限时间           | number | 秒              |
| message.message_length_limit.text    | 文本消息长度限制             | number | 字              |
| group.search.allow                   | 是否允许搜索群聊             | bool   |                |
| group.quit_user.allow                | 是否允许普通用户退出群聊         | bool   |                |
| group.show_member.allow              | 是否显示群成员              | bool   |                |
| group.owner_clear_screen.allow       | 是否允许群主清屏             | bool   |                |
| group.show_member_count.allow        | 是否显示群成员数量            | bool   |                |
| group.show_invite_msg.allow          | 是否显示群邀请消息            | bool   |                |
| group.show_fake_member_count.allow   | 是否显示群假成员数量           | bool   |                |
| group.show_group_info.user           | 普通用户是否显示群信息          | bool   |                |
| group.show_group_info.user.link_name | 发现底导的名称              | bool   |                |
| group.show_group_info.user.link_show | 是否展示发现底导             | bool   |                |