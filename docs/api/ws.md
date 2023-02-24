# websocket gateway

## 长连接请求和响应协议

### 请求

```protobuf
// 客户端发送的消息体
message RequestBody {
  string reqId = 1; // 客户端生成的请求id，返回时会原样返回
  string method = 2; // 请求方法，类似于http中的path
  bytes data = 3; // 请求数据
}
```

### 响应

```protobuf
enum PushEvent {
  // 消息推送
  PushMsgDataList = 0;
  // 通知推送
  PushNoticeData = 1;
  // 响应返回
  PushResponseBody = 2;
}
// 服务端会推送的消息体
message PushBody {
  PushEvent event = 1; // 此时 event 为 PushResponseBody 
  bytes data = 2; // 此时 data 为 ResponseBody
}
// 服务端返回响应的消息体
message ResponseBody {
  enum Code {
    Success = 0;

    UnknownError = 1;
    InternalError = 2; // 服务端内部错误
    RequestError = 3; // 请求错误 可能 method 不正确
    AuthError = 4; // 鉴权错误 可能没有登录或者登录过期
    ToastError = 5; // toast错误 需要toast提示
    AlertError = 7; // alert错误 需要alert提示
    RetryError = 8; // 服务端临时错误，需要重试
  }
  string reqId = 1; // 客户端生成的请求id，返回时会原样返回
  string method = 2; // 请求方法，类似于http中的path
  Code code = 3; // 响应码
  bytes data = 4; // 响应数据
}

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

## 通讯流程

### 1. 连接

> 客户端连接websocket，url为`ws[s]://<host>:<port>/ws`

### 2. 服务端返回

> 服务端推送一条字符串类型的消息，内容为`connected`

### 3. 设置连接参数

> 发送一条二进制类型消息，内容为RequestBody的序列化数据。其中`method`为`/v1/conn/white/setCxnParams`，`data`
> 为`SetCxnParamsReq`的序列化数据

```protobuf
message SetCxnParamsReq {
  string packageId = 1; // 客户端包id 每次安装都会变化
  string platform = 2; // 客户端平台 ios/android/web/macos/windows/linux
  string deviceId = 3; // 客户端设备id
  string deviceModel = 4; // 客户端设备型号
  string osVersion = 5; // 客户端系统版本
  string appVersion = 6; // 客户端app版本
  string language = 7; // 客户端语言
  string networkUsed = 8; // 客户端网络类型
  bytes ext = 11; // 扩展字段
  optional bytes aesKey = 9; // 用于加密请求和响应的aes key; 他应该是随机字符串的rsa加密结果; 服务端拿到该字段后使用rsa私钥解密得到随机字符串; 然后md5(随机字符串)得到aes key
  optional bytes aesIv = 10; // 用于加密请求和响应的aes iv; 他应该是随机字符串的rsa加密结果; 服务端拿到该字段后使用rsa私钥解密得到随机字符串; 然后md5_16(随机字符串)得到aes iv
}

// 如果你的请求传入了aesKey aesIv，服务端校验成功。那么服务端之后所有的推送PushBody都会使用aes加密（包括本次响应）。客户端之后的每次请求都需要使用aes加密
message SetCxnParamsResp {}
```

### 4. 设置用户参数

> 发送一条二进制类型消息，内容为RequestBody的序列化数据。其中`method`为`/v1/conn/white/setUserParams`，`data`
> 为`SetUserParamsReq`的序列化数据

```protobuf
message SetUserParamsReq {
  string userId = 1;
  string token = 2;
  bytes ext = 11;
}

message SetUserParamsResp {}
```

### 5. 进行业务请求

发送一条二进制类型消息，内容为RequestBody的序列化数据。其中`method`为http api文档中的path，`data`为文档中的请求数据的序列化数据

### 注意

- 1.data中CommonReq不需要传递，此参数为服务端专用参数，客户端不需要传递
- 2.当ResponseBody中的code不为0时，data中的数据为CommonResp的序列化数据
- 3.CommonResp中的data的msg在每种code下的含义不同 如下

| code | msg含义            | msg数据结构                                                                                     |
|------|------------------|---------------------------------------------------------------------------------------------|
| 1    | 未知错误原因           | string                                                                                      |
| 2    | 内部错误原因           | string                                                                                      |
| 3    | 请求错误             | 无                                                                                           |
| 4    | 鉴权错误原因           | string                                                                                      |
| 5    | toast错误 需要吐丝     | string                                                                                      |
| 7    | alert错误 需要dialog | {title: "", msg: "", actions: [{action: 0, title: "确定", "jumpTo": "xxx"}]} //action=0代表取消动作 |
| 8    | 临时错误 需要重试        | 展示重试弹窗                                                                                      |

#### 示例1. 发送消息

> 根据http api中msg文档，发送一条二进制类型消息，内容为RequestBody的序列化数据。其中`method`为`/v1/msg/sendMsgList`，`data`
> 为`SendMsgListReq`的序列化数据

```protobuf
message SendMsgListReq {
  repeated MsgData msgDataList = 1;
  // options
  //  延迟时间（秒） 不得大于 864000秒 也就是10天 只有开启了Pulsar的延迟消息功能才有效
  optional int32 deliverAfter = 2;
}

message SendMsgListResp {}
```

#### 示例2. 使用golang请求流程

```go
package main

import (
	"context"
	"errors"
	"github.com/cherish-chat/xxim-server/common/pb"
	"github.com/cherish-chat/xxim-server/common/utils"
	"github.com/cherish-chat/xxim-server/common/xorm"
	"github.com/zeromicro/go-zero/core/mr"
	"google.golang.org/protobuf/proto"
	"log"
	"nhooyr.io/websocket"
	"strconv"
	"sync"
	"time"
)

var ctx = context.Background()
var respMap sync.Map

func url(path string) string { return "wss://api.cherish.chat" + path }

func Connect() *websocket.Conn {
	conn, response, err := websocket.Dial(ctx, url("/ws"), nil)
	if err != nil {
		log.Fatalf("failed to dial: %v", err)
	}
	// 打印响应头
	log.Printf("response.Header: %v", response.Header)
	// 打印响应状态码
	log.Printf("response.StatusCode: %v", response.StatusCode)
	// 返回conn
	go loopRead(conn)
	return conn
}

func loopRead(conn *websocket.Conn) {
	for {
		_, data, err := conn.Read(ctx)
		if err != nil {
			log.Fatalf("failed to read: %v", err)
		}
		pushBody := &pb.PushBody{}
		_ = proto.Unmarshal(data, pushBody)
		if pushBody.Event == pb.PushEvent_PushResponseBody {
			responseBody := &pb.ResponseBody{}
			_ = proto.Unmarshal(pushBody.Data, responseBody)
			value, ok := respMap.Load(responseBody.ReqId)
			if ok {
				ch := value.(chan *pb.ResponseBody)
				ch <- responseBody
			}
		}
	}
}

func RequestX(
	conn *websocket.Conn,
	method string,
	data proto.Message,
	response proto.Message,
) error {
	id := utils.GenId()
	dataBuff, _ := proto.Marshal(data)
	reqBody := &pb.RequestBody{
		ReqId:  id,
		Method: method,
		Data:   dataBuff,
	}
	dataBuff, _ = proto.Marshal(reqBody)
	ch := make(chan *pb.ResponseBody, 1)
	respMap.Store(id, ch)
	err := conn.Write(ctx, websocket.MessageBinary, dataBuff)
	if err != nil {
		return err
	}
	// 5s ctx
	ctx, _ := context.WithTimeout(context.Background(), 5*time.Second)
	for {
		select {
		case <-ctx.Done():
			return errors.New("timeout")
		case valueBody := <-ch:
			respMap.Delete(id)
			if valueBody.Code == pb.ResponseBody_Success {
				err = proto.Unmarshal(valueBody.Data, response)
				if err != nil {
					return err
				} else {
					return nil
				}
			} else {
				return errors.New(valueBody.Code.String())
			}
		}
	}

}

func Login(conn *websocket.Conn, req *pb.LoginReq) *pb.LoginResp {
	resp := &pb.LoginResp{}
	err := RequestX(conn, "/v1/user/white/login", req, resp)
	if err != nil {
		log.Fatalf("failed to login: %v", err)
	}
	log.Printf("login resp: %v", utils.AnyToString(resp))
	return resp
}

func SetConnParams(conn *websocket.Conn, req *pb.SetCxnParamsReq) {
	resp := &pb.SetCxnParamsResp{}
	err := RequestX(conn, "/v1/conn/white/setCxnParams", req, resp)
	if err != nil {
		log.Fatalf("failed to set_conn_params: %v", err)
	}
	log.Printf("set_conn_params resp: %v", utils.AnyToString(resp))
}

func SetUserParams(conn *websocket.Conn, req *pb.SetUserParamsReq) {
	resp := &pb.SetUserParamsResp{}
	err := RequestX(conn, "/v1/conn/white/setUserParams", req, resp)
	if err != nil {
		log.Fatalf("failed to setUserParams: %v", err)
	}
	log.Printf("setUserParams resp: %v", utils.AnyToString(resp))
}

func GetFriendList(conn *websocket.Conn, req *pb.GetFriendListReq) {
	resp := &pb.GetFriendListResp{}
	err := RequestX(conn, "/v1/relation/getFriendList", req, resp)
	if err != nil {
		log.Fatalf("failed to getFriendList: %v", err)
	}
	log.Printf("getFriendList resp: %v", utils.AnyToString(resp))
}

func SendMsg(conn *websocket.Conn, req *pb.SendMsgListReq) {
	resp := &pb.SendMsgListResp{}
	err := RequestX(conn, "/v1/msg/sendMsgList", req, resp)
	if err != nil {
		log.Fatalf("failed to sendMsgList: %v", err)
	}
	log.Printf("sendMsgList resp: %v", utils.AnyToString(resp))
}

func main() {
	conn := Connect()
	SetConnParams(conn, &pb.SetCxnParamsReq{
		PackageId:   "xxxx",
		DeviceId:    "xxxx",
		Platform:    "windows",
		NetworkUsed: "NetworkUsed",
		DeviceModel: "DeviceModel",
		OsVersion:   "OsVersion",
		AppVersion:  "AppVersion",
		Language:    "Language",
	})
	loginResp := Login(conn, &pb.LoginReq{
		CommonReq: nil,
		Id:        "test123456",
		Password:  utils.Md5("123456"),
	})
	SetUserParams(conn, &pb.SetUserParamsReq{
		UserId: "test123456",
		Token:  loginResp.Token,
		Ext:    nil,
	})
	GetFriendList(conn, &pb.GetFriendListReq{
		Page: &pb.Page{
			Page: 1,
			Size: 999999,
		},
		Opt: pb.GetFriendListReq_WithBaseInfo,
	})
	var num = 0
	var fs []func()
	for i := 0; i < 64; i++ {
		fs = append(fs, func() {
			for i := 0; i < 10000; i++ {
				num++
				SendMsg(conn, &pb.SendMsgListReq{
					MsgDataList: []*pb.MsgData{{
						ClientMsgId: utils.GenId(),
						ClientTime:  utils.AnyToString(time.Now().UnixMilli()),
						SenderId:    "test123456",
						SenderInfo: utils.AnyToBytes(xorm.M{
							"id":       "test123456",
							"nickname": "test123456",
							"avatar":   "https://fakeimg.pl/500x500/C71585/FFF/?font=noto&text=贺",
						}),
						ConvId:      "group:10134",
						AtUsers:     nil,
						ContentType: int32(pb.ContentType_TEXT),
						Content:     []byte(strconv.Itoa(num)),
						Options: &pb.MsgData_Options{
							StorageForServer:  true,
							StorageForClient:  true,
							NeedDecrypt:       false,
							OfflinePush:       false,
							UpdateConvMsg:     true,
							UpdateUnreadCount: true,
						},
						OfflinePush: &pb.MsgData_OfflinePush{
							Title:   "标题",
							Content: "内容",
							Payload: "",
						},
						Ext: nil,
					}},
					DeliverAfter: nil,
					CommonReq:    nil,
				})
			}
		})
	}
	start := time.Now()
	mr.FinishVoid(fs...)
	now := time.Now()
	millis := now.Sub(start).Milliseconds()
	// 统计 qps
	log.Printf("startTime: %v, endTime: %v, qps: %v", start, now, int64(num)*1000/millis)
	// 阻塞
	select {}
}
```