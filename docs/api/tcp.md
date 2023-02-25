# tcp gateway

## 长连接请求和响应协议

### tcp请求

| 字段      | 长度  | 类型     | 说明                 |
|---------|-----|--------|--------------------|
| dataLen | 4   | uint32 | 表示变长data的长度        |
| reqId   | 4   | uint32 | 1表示text; 2表示binary |
| data    | 变长  | bytes  | 请求数据               |

### tcp响应

| 字段      | 长度  | 类型     | 说明                 |
|---------|-----|--------|--------------------|
| dataLen | 4   | uint32 | 表示变长data的长度        |
| reqId   | 4   | uint32 | 1表示text; 2表示binary |
| data    | 变长  | bytes  | 请求数据               |

### 业务请求

```protobuf
// 客户端发送的消息体
message RequestBody {
  string reqId = 1; // 客户端生成的请求id，返回时会原样返回
  string method = 2; // 请求方法，类似于http中的path
  bytes data = 3; // 请求数据
}
```

### 业务响应

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
```

## 通讯流程

### 1. 连接

> 客户端连接tcp，`<host>:<port>`

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
  bytes aesKey = 9; // 用于加密请求和响应的aes key; 他应该是随机字符串的rsa加密结果; 服务端拿到该字段后使用rsa私钥解密得到随机字符串; 然后md5(随机字符串)得到aes key
  bytes aesIv = 10; // 用于加密请求和响应的aes iv; 他应该是随机字符串的rsa加密结果; 服务端拿到该字段后使用rsa私钥解密得到随机字符串; 然后md5_16(随机字符串)得到aes iv
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
	"bytes"
	"context"
	"encoding/binary"
	"errors"
	"github.com/aceld/zinx/ziface"
	"github.com/aceld/zinx/znet"
	"github.com/cherish-chat/xxim-server/common/pb"
	"github.com/cherish-chat/xxim-server/common/utils"
	"github.com/cherish-chat/xxim-server/common/xorm"
	"github.com/zeromicro/go-zero/core/mr"
	"google.golang.org/protobuf/proto"
	"io"
	"log"
	"net"
	"strconv"
	"sync"
	"time"
)

var dp = znet.NewDataPack()

func pack(msg *znet.Message) ([]byte, error) {
	//创建一个存放bytes字节的缓冲
	dataBuff := bytes.NewBuffer([]byte{})

	//写dataLen
	if err := binary.Write(dataBuff, binary.LittleEndian, msg.GetDataLen()); err != nil {
		return nil, err
	}

	//写msgID
	if err := binary.Write(dataBuff, binary.LittleEndian, msg.GetMsgID()); err != nil {
		return nil, err
	}

	//写data数据
	if err := binary.Write(dataBuff, binary.LittleEndian, msg.GetData()); err != nil {
		return nil, err
	}
	return dataBuff.Bytes(), nil
}

// Unpack 拆包方法(解压数据)
func unpack(binaryData []byte) (ziface.IMessage, error) {
	//创建一个从输入二进制数据的ioReader
	dataBuff := bytes.NewReader(binaryData)

	//只解压head的信息，得到dataLen和msgID
	msg := &znet.Message{}

	//读dataLen
	if err := binary.Read(dataBuff, binary.LittleEndian, &msg.DataLen); err != nil {
		return nil, err
	}

	//读msgID
	if err := binary.Read(dataBuff, binary.LittleEndian, &msg.ID); err != nil {
		return nil, err
	}

	//这里只需要把head的数据拆包出来就可以了，然后再通过head的长度，再从conn读取一次数据
	return msg, nil
}

var respMap sync.Map

func Connect() net.Conn {
	conn, err := net.Dial("tcp", "api.cherish.chat:8999")
	if err != nil {
		log.Fatalf("failed to dial: %v", err)
	}
	// 返回conn
	go loopRead(conn)
	return conn
}

func loopRead(conn net.Conn) {
	for {
		headData := make([]byte, dp.GetHeadLen())
		_, err := io.ReadFull(conn, headData)
		if err != nil {
			log.Fatalf("failed to read head: %v", err)
		}
		msgHead, err := unpack(headData)
		if err != nil {
			log.Fatalf("failed to unpack head: %v", err)
		}
		if msgHead.GetDataLen() > 0 {
			msg := msgHead.(*znet.Message)
			msg.Data = make([]byte, msg.GetDataLen())
			_, err := io.ReadFull(conn, msg.Data)
			if err != nil {
				log.Fatalf("failed to read data: %v", err)
			}
			pushBody := &pb.PushBody{}
			_ = proto.Unmarshal(msg.Data, pushBody)
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
}

func RequestX(
	conn net.Conn,
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
	msg := &znet.Message{
		DataLen: uint32(len(dataBuff)),
		ID:      2,
		Data:    dataBuff,
	}
	dataBuff, _ = pack(msg)
	_, err := conn.Write(dataBuff)
	if err != nil {
		return err
	}
	// 5s ctx
	ctx, _ := context.WithTimeout(context.Background(), 5*time.Second)
	for {
		select {
		case <-ctx.Done():
			log.Printf("request timeout, method: %v, reqId: %v", method, id)
			return ctx.Err()
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

func Login(conn net.Conn, req *pb.LoginReq) *pb.LoginResp {
	resp := &pb.LoginResp{}
	err := RequestX(conn, "/v1/user/white/login", req, resp)
	if err != nil {
		log.Fatalf("failed to login: %v", err)
	}
	log.Printf("login resp: %v", utils.AnyToString(resp))
	return resp
}

func SetConnParams(conn net.Conn, req *pb.SetCxnParamsReq) {
	resp := &pb.SetCxnParamsResp{}
	err := RequestX(conn, "/v1/conn/white/setCxnParams", req, resp)
	if err != nil {
		log.Fatalf("failed to set_conn_params: %v", err)
	}
	log.Printf("set_conn_params resp: %v", utils.AnyToString(resp))
}

func SetUserParams(conn net.Conn, req *pb.SetUserParamsReq) {
	resp := &pb.SetUserParamsResp{}
	err := RequestX(conn, "/v1/conn/white/setUserParams", req, resp)
	if err != nil {
		log.Fatalf("failed to setUserParams: %v", err)
	}
	log.Printf("setUserParams resp: %v", utils.AnyToString(resp))
}

func GetFriendList(conn net.Conn, req *pb.GetFriendListReq) {
	resp := &pb.GetFriendListResp{}
	err := RequestX(conn, "/v1/relation/getFriendList", req, resp)
	if err != nil {
		log.Fatalf("failed to getFriendList: %v", err)
	}
	log.Printf("getFriendList resp: %v", utils.AnyToString(resp))
}

func SendMsg(conn net.Conn, req *pb.SendMsgListReq) {
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