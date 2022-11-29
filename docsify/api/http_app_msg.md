# http api 接口-消息

## 1. SendMsg: 发送消息

- 请求地址：`/v1/msg/sendMsg`
- 请求体：

```protobuf
message SendMsgListReq {
  Requester requester = 1;
  repeated MsgData msgDataList = 2;
  // options
  //  延迟时间（秒） 不得大于 864000秒 也就是10天 只有开启了Pulsar的延迟消息功能才有效
  optional int32 deliverAfter = 11;
}
```

- 响应体：

```protobuf
message SendMsgListResp {
  CommonResp commonResp = 1;
}
```

## 2. GetMsgListByConvId: 获取会话消息列表

- 请求地址：`/v1/msg/getMsgListByConvId`
- 请求体：

```protobuf
message GetMsgListByConvIdReq {
  Requester requester = 1;
  string convId = 2;
  repeated string seqList = 3;
  bool push = 4; // 是否使用websocket推送的方式
}
```

- 响应体：

```protobuf
message GetMsgListResp {
  CommonResp commonResp = 1;
  repeated MsgData msgDataList = 2; // 如果是websocket推送的方式，这里是空的
}
```

## 3. getMsgById: 通过id获取消息

- 请求地址：`/v1/msg/getMsgById`
- 请求体：

```protobuf
message GetMsgByIdReq {
  CommonReq commonReq = 1;
  optional string serverMsgId = 2;
  optional string clientMsgId = 3;
  bool push = 4;
}
```

- 响应体：

```protobuf
message GetMsgByIdResp {
  CommonResp commonResp = 1;
  MsgData msgData = 2;
}
```

## 4. batchGetConvSeq: 批量获取会话的消息seq

- 请求地址：`/v1/msg/batchGetConvSeq`
- 请求体：

```protobuf
message BatchGetConvSeqReq {
  CommonReq commonReq = 1;
  repeated string convIdList = 2;
}
```

- 响应体：

```protobuf
message BatchGetConvSeqResp {
  CommonResp commonResp = 1;
  message ConvSeq {
    string convId = 1;
    string minSeq = 2;
    string maxSeq = 3;
    string updateTime = 4;
  }
  map<string, ConvSeq> convSeqMap = 2;
}
```