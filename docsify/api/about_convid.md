# 关于convId的生成

## golang实现
```go
package pb

import (
	"strconv"
	"strings"
)

// IdSeparator 会话id之间的分隔符
const IdSeparator = "-"
const SinglePrefix = "single:"
const GroupPrefix = "group:"
const SubPrefix = "sub:"

func (x *MsgData) IsSingleConv() bool {
	return strings.HasPrefix(x.ConvId, SinglePrefix)
}

func (x *MsgData) ReceiverUid() string {
	split := strings.Split(strings.TrimPrefix(x.ConvId, SinglePrefix), IdSeparator)
	if len(split) == 2 {
		if split[0] == x.SenderId {
			return split[1]
		}
		return split[0]
	}
	return ""
}

func (x *MsgData) ReceiverGid() string {
	return strings.TrimPrefix(x.ConvId, GroupPrefix)
}

func (x *MsgData) IsGroupConv() bool {
	return strings.HasPrefix(x.ConvId, GroupPrefix)
}

func (x *MsgData) IsSubConv() bool {
	return strings.HasPrefix(x.ConvId, SubPrefix)
}

func ServerMsgId(convId string, seq int64) string {
	return convId + IdSeparator + strconv.FormatInt(seq, 10)
}

func SingleConvId(id1 string, id2 string) string {
	if id1 < id2 {
		return SinglePrefix + id1 + IdSeparator + id2
	}
	return SinglePrefix + id2 + IdSeparator + id1
}

func GroupConvId(groupId string) string {
	return SinglePrefix + groupId
}

func SubConvId(subId string) string {
	return SinglePrefix + subId
}

func ParseConvServerMsgId(serverMsgId string) (convId string, seq int64) {
	arr := strings.Split(serverMsgId, IdSeparator)
	if len(arr) == 2 {
		convId = arr[0]
		seq, _ = strconv.ParseInt(arr[1], 10, 64)
	} else if len(arr) == 3 {
		convId = arr[0] + IdSeparator + arr[1]
		seq, _ = strconv.ParseInt(arr[2], 10, 64)
	}
	return
}
```

## dart实现
```dart
class ConvId {
  static const String separator = "-";
  static const String singlePrefix = "single:";
  static const String groupPrefix = "group:";
  static const String subPrefix = "sub:";

  static String singleConvId(String id1, String id2) {
    if (id1.compareTo(id2) < 0) {
      return singlePrefix + id1 + separator + id2;
    }
    return singlePrefix + id2 + separator + id1;
  }

  static String groupConvId(String groupId) {
    return groupPrefix + groupId;
  }

  static String subConvId(String subId) {
    return subPrefix + subId;
  }

  static String parseConvId(String serverMsgId) {
    var arr = serverMsgId.split(separator);
    if (arr.length == 2) {
      return arr[0];
    } else if (arr.length == 3) {
      return arr[0] + separator + arr[1];
    }
    return "";
  }

  static int parseSeq(String serverMsgId) {
    var arr = serverMsgId.split(separator);
    if (arr.length == 2) {
      return int.parse(arr[1]);
    } else if (arr.length == 3) {
      return int.parse(arr[2]);
    }
    return 0;
  }
}
```