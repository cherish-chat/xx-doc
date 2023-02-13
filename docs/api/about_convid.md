# 关于convId的生成

## 生成规则

### 单聊

#### golang

```go
package main

import "strings"

func GenSingleConvId(id1 string, id2 string) string {
	if id1 < id2 {
		return "single:" + id1 + "-" + id2
	}
	return "single:" + id2 + "-" + id1
}

func IsSingleConvId(convId string) bool {
	return strings.HasPrefix(convId, "single:")
}

```

#### dart

```dart
String genSingleConvId(String id1, String id2) {
  if (id1.compareTo(id2) < 0) {
    return 'single:$id1-$id2';
  }
  return 'single:$id2-$id1';
}

bool isSingleConvId(String convId) {
  return convId.startsWith('single:');
}
```

#### java

```java
public static String genSingleConvId(String id1, String id2) {
  if (id1.compareTo(id2) < 0) {
    return "single:" + id1 + "-" + id2;
  }
  return "single:" + id2 + "-" + id1;
}

public static boolean isSingleConvId(String convId) {
  return convId.startsWith("single:");
}
```

#### swift

```swift
func genSingleConvId(_ id1: String, _ id2: String) -> String {
  if id1 < id2 {
    return "single:\(id1)-\(id2)"
  }
  return "single:\(id2)-\(id1)"
}

func isSingleConvId(_ convId: String) -> Bool {
  return convId.hasPrefix("single:")
}
```

#### cpp

```cpp
std::string genSingleConvId(const std::string& id1, const std::string& id2) {
  if (id1 < id2) {
    return "single:" + id1 + "-" + id2;
  }
  return "single:" + id2 + "-" + id1;
}

bool isSingleConvId(const std::string& convId) {
  return convId.find("single:") == 0;
}
```

#### javascript

```javascript
function genSingleConvId(id1, id2) {
    if (id1 < id2) {
        return 'single:' + id1 + '-' + id2;
    }
    return 'single:' + id2 + '-' + id1;
}

function isSingleConvId(convId) {
    return convId.startsWith('single:');
}
```

### 群聊

#### golang

```go
package main

import "strings"

func GenGroupConvId(groupId string) string {
	return "group:" + groupId
}

func IsGroupConvId(convId string) bool {
	return strings.HasPrefix(convId, "group:")
}

```

#### dart

```dart
String genGroupConvId(String groupId) {
  return 'group:$groupId';
}

bool isGroupConvId(String convId) {
  return convId.startsWith('group:');
}
```

#### java

```java
public static String genGroupConvId(String groupId) {
  return "group:" + groupId;
}

public static boolean isGroupConvId(String convId) {
  return convId.startsWith("group:");
}
```

#### swift

```swift
func genGroupConvId(_ groupId: String) -> String {
  return "group:\(groupId)"
}

func isGroupConvId(_ convId: String) -> Bool {
  return convId.hasPrefix("group:")
}
```

#### cpp

```cpp
std::string genGroupConvId(const std::string& groupId) {
  return "group:" + groupId;
}

bool isGroupConvId(const std::string& convId) {
  return convId.find("group:") == 0;
}
```

#### javascript

```javascript
function genGroupConvId(groupId) {
    return 'group:' + groupId;
}

function isGroupConvId(convId) {
    return convId.startsWith('group:');
}
```
