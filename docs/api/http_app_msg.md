# http api 接口-消息

## 1. SendMsg: 发送消息

- 请求地址：`/v1/msg/sendMsgList`
- 请求体：

```protobuf
message SendMsgListReq {
  repeated MsgData msgDataList = 1;
  // options
  //  延迟时间（秒） 不得大于 864000秒 也就是10天 只有开启了Pulsar的延迟消息功能才有效
  optional int32 deliverAfter = 2;
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
}
```

- 响应体：

```protobuf
message GetMsgListResp {
  repeated MsgData msgDataList = 1; // 如果是websocket推送的方式，这里是空的
}
```

## 3. getMsgById: 通过id获取消息

- 请求地址：`/v1/msg/getMsgById`
- 请求体：

```protobuf
message GetMsgByIdReq {
  optional string serverMsgId = 1;
  optional string clientMsgId = 2;
}
```

- 响应体：

```protobuf
message GetMsgByIdResp {
  MsgData msgData = 1;
}
```

## 4. batchGetConvSeq: 批量获取会话的消息seq

- 请求地址：`/v1/msg/batchGetConvSeq`
- 请求体：

```protobuf
message BatchGetConvSeqReq {
  repeated string convIdList = 1;
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
}
```