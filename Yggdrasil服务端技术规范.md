本文旨在为实现Yggdrasil服务端提供非官方的技术规范。
由于Mojang的服务端为黑盒，我们只能通过反向工程推测其逻辑，因此难免会有一些错误，所有API的表现都以Mojang的实现为准。

# 基本约定

## 请求与响应格式
若无特殊说明，请求与响应均为JSON格式（如果有body），`Content-Type`均为`application/json; charset=utf-8`。

所有API都应该使用https协议。

## 错误信息格式
```javascript
{
	"error":"错误的简要描述（机器可读）",
	"errorMessage":"错误的详细信息（人类可读）",
	"cause":"该错误的原因（可选）"
}
```

当遇到本文中已说明的异常情况时，返回的错误信息应符合对应的要求。

下表列举了常见异常情况下的错误信息。除特殊说明外，`cause`一般不包含。

**非标准**指由于无法使Mojang的Yggdrasil服务器触发对应异常，而只能推测该种情况下的错误信息。

**未定义**指该项并没有明确要求。

|异常情况|HTTP状态码|Error|Error Message|
|--------|----------|-----|------------|
|一般HTTP异常（非业务异常，如 _Not Found_、 _Method Not Allowed_）|_未定义_|_该HTTP状态对应的Reason Phrase（于[HTTP/1.1](https://tools.ietf.org/html/rfc2616#section-6.1.1)中定义）_|_未定义_|
|令牌无效|403|ForbiddenOperationException|Invalid token.|
|密码错误，或短时间内多次登录失败而被暂时禁止登录|403|ForbiddenOperationException|Invalid credentials. Invalid username or password.|
|试图向一个已经绑定了角色的令牌指定其要绑定的角色|400|IllegalArgumentException|Access token already has a profile assigned.|
|试图向一个令牌绑定不属于其对应用户的角色 _（非标准）_|403|ForbiddenOperationException|_未定义_|


## 数据格式
我们约定以下数据格式
 * **无符号UUID**: 指去掉所有`-`字符后的UUID字符串

## 模型

### 用户
一个系统中可以存在若干个用户，用户具有以下属性：
 * ID
 * 邮箱
 * 密码

其中ID为一个无符号UUID。邮箱可以变更，但需要保证唯一。

#### 用户信息的序列化
用户信息序列化后符合以下格式：
```javascript
{
	"id":"用户的ID",
	"properties":[ // 用户的属性（数组，每一元素为一个属性）
		{ // 一项属性
			"name":"属性的键",
			"value":"属性的值",
		}
		// ,...（可以有更多）
	]
}
```

用户属性中目前已知的键如下（并不一定会包含）：

|键|对应的值|
|--|--------|
|preferredLanguage|用户的偏好语言，[Java Locale格式](https://docs.oracle.com/javase/8/docs/api/java/util/Locale.html#toString--)(?)，例如`en`、`zh_CN`|
|twitch_access_token|[Twitch账户Token](https://dev.twitch.tv/docs/v5/guides/using-the-twitch-api/)，仅当绑定Twitch账户时才存在|

### 角色（Profile）
> Mojang当前不支持多角色，不保证多角色部分内容的正确性。

角色与账号为多对一关系。一个角色对应Minecraft中的一个实体玩家。角色具有以下属性：
 * UUID
 * 名称
 * 材质模型
   * 可选值：STEVE，ALEX
 * 材质
   * 类型为映射
   * key可选值有：SKIN，CAPE，ELYTRA
   * value类型为URL

UUID和名称均为全局唯一，但名称可变。应避免使用名称作为标识。

#### 材质URL规范
> 目前无法确定文件名的生成方法。从其用途来看应为文件内容的摘要，但其长度(61字符的HEX字符串)不属于任何已知Hash算法。
> 
> 如果使用SHA-256作为文件名的生成方法，由于长度不同，不会与Mojang的文件名发生冲突，因此是可行的。

> 实现时请注意：对于玩家上传的材质，服务端应将其读取后重新写入，以避免两幅内容相同的图像由于其中不影响显示的数据不同造成Hash不同。

材质格式为PNG，文件名应为材质的`SHA-256`校验和，如：
```
https://yggdrasil.example.com/textures/e051c27e803ba15de78a1d1e83491411dffb6d7fd2886da0a6c34a2161f7ca99
```

#### 角色信息的序列化
角色信息序列化后符合以下格式：
```javascript
{
	"id":"角色UUID（无符号）",
	"name":"角色名称",
	"properties":[ // 角色的属性（数组，每一元素为一个属性）（仅在特定情况下需要包含）
		{ // 一项属性
			"name":"属性的键",
			"value":"属性的值",
			"signature":"属性值的数字签名（仅在特定情况下需要包含）"
		}
		// ,...（可以有更多）
	]
}
```

`properties`及`signature`项目在无特殊说明的情况下不需要包含。

`signature`是一个Base64字符串，其中包含属性值（使用UTF-8编码）的数字签名（使用SHA1withRSA算法，见[PKCS #1](https://www.rfc-editor.org/rfc/rfc2437.txt)）。

角色属性中目前已知的键有`textures`（并不一定会包含）。它对应的值是一个Base64字符串，内容为JSON字符串，包含角色的材质信息，格式如下：
```javascript
{
	"timestamp":"该属性值被生成时的时间，为Java时间格式（自1970-01-01 00:00:00 UTC至今经过的毫秒数）",
	"profileId":"角色UUID（无符号）",
	"profileName":"角色名称",
	"textures":{ // 角色的材质
		"材质类型（如SKIN）":"材质的URL" // 若角色不具有该项材质，则不必包含
		// ,...（可以有更多）
	}
}
```

### 令牌（Token）
令牌与账号为多对一关系。令牌是一种登录凭证，具有时效性。令牌具有以下属性：
 * accessToken
 * clientToken
 * 绑定的角色
 * 颁发时间

其中`accessToken`和`clientToken`为无符号UUID。`accessToken`由服务端随机生成，`clientToken`由客户端提供。

绑定的角色可以为空。它代表了能使用该令牌进行游戏的角色。

一个用户可以同时有多个令牌，但服务端也应该对令牌数量加以限制。当令牌数量超出限制（如10个）时，则应先吊销最旧的令牌，之后再颁发新的令牌。

#### 令牌的有效时间
> 以下内容基于揣测，确实有点道理，但也很有可能不正确。

我们定义令牌具有三种状态：有效、暂时失效、无效。令牌状态可以通过颁发时间计算得出，其中有两个分界点，分别是有效和暂时失效的分界点，及暂时失效和无效的分界点。这两个分界点并没有硬性规定，一般前者为颁发后3天，后者为颁发后7天。

当令牌有效时，可以使用它进行各项操作，如用于进入服务器时的验证。

当令牌暂时失效时，它除了进行刷新操作外，无权进行任何操作。刷新操作会吊销原令牌，并颁发一个新的令牌。

当令牌无效时，它无权进行任何操作。服务端不需要存储已经处于无效状态的令牌。

定义这些状态是为了满足这样的需求：同一个令牌在分配后一段时间内可以多次使用；如果令牌距离颁发时间过去不是太久，则可以通过刷新动作获得一个新的令牌；如果距离颁发时间已经过去很久，则出于安全考虑吊销令牌。这样的话，玩家如果经常进行登录操作，除了第一次登录就不需要输入密码了。而当他长时间未登录时则需要重新输入密码。

# Yggdrasil API

## 用户部分

### 登录
`POST /authserver/authenticate`

使用邮箱和密码进行身份验证，并分配一个新的令牌。

请求格式：
```javascript
{
	"username":"邮箱",
	"password":"密码",
	"clientToken":"由客户端指定的令牌的clientToken（可选）",
	"requestUser":true/false, // 是否在响应中包含用户信息，默认false
	"agent":{
		"name":"Minecraft",
		"version":1
	}
}
```

若请求中未包含`clientToken`，服务端应该随机生成一个无符号UUID作为`clientToken`。但需要注意`clientToken`可以为任何字符串，即请求中提供任何`clientToken`都是可以接受的，不一定要为无符号UUID。

对于令牌要绑定的角色：若用户没有任何角色，则为空；若用户仅有一个角色，那么通常绑定到该角色；若用户有多个角色，通常为空，以便客户端进行选择。也就是说如果绑定的角色为空，则需要客户端进行角色选择。

响应格式：
```javascript
{
	"accessToken":"令牌的accessToken",
	"clientToken":"令牌的clientToken",
	"availableProfiles":[ // 用户可用角色列表
		// ,... 每一项为一个角色
	],
	"selectedProfile":{
		// ... 绑定的角色，若为空，则不需要包含
	},
	"user":{
			// ... 用户信息（仅当请求中requestUser为true时包含）
	}
}
```

**安全提示：** 该API可以被用于密码暴力破解，应受到速率限制。限制应针对用户，而不是客户端IP。

### 刷新
`POST /authserver/refresh`

吊销原令牌，并颁发一个新的令牌。

请求格式：
```javascript
{
	"accessToken":"令牌的accessToken",
	"clientToken":"令牌的clientToken（可选）",
	"requestUser":true/false, // 是否在响应中包含用户信息，默认false
	"selectedProfile":{
		// ... 要选择的角色（可选）
	}
}
```

当指定`clientToken`时，服务端应检查`accessToken`和`clientToken`是否有效，否则只需要检查`accessToken`。

颁发的新令牌的`clientToken`应与原令牌的相同。`selectedProfile`只有在原令牌绑定的角色为空时才能包含，新令牌应绑定到该项目指定的角色上。

该操作在令牌暂时失效时依然可以执行。若请求失败，原令牌依然有效。

返回格式：
```javascript
{
	"accessToken":"新令牌的accessToken",
	"clientToken":"新令牌的clientToken",
	"selectedProfile":{
		// ... 新令牌绑定的角色，若为空，则不需要包含
	},
	"user":{
		// ... 用户信息（仅当请求中requestUser为true时包含）
	}
}
```

### 验证令牌
`POST /authserver/validate`

检验令牌是否有效。

请求格式：
```javascript
{
	"accessToken":"令牌的accessToken",
	"clientToken":"令牌的clientToken（可选）"
}
```

当指定`clientToken`时，服务端应检查`accessToken`和`clientToken`是否有效，否则只需要检查`accessToken`。

若令牌有效，应返回HTTP状态码`204 No Content`，否则作为令牌无效的异常情况处理。

### 吊销令牌
`POST /authserver/invalidate`

吊销给定令牌。

请求格式：
```javascript
{
	"accessToken":"令牌的accessToken",
	"clientToken":"令牌的clientToken（可选）"
}
```

服务端只需要检查`accessToken`，即无论`clientToken`为何值都不会造成影响。

无论操作是否成功，服务端应返回HTTP状态码`204 No Content`。

### 登出
`POST /authserver/signout`

吊销用户的所有令牌。

请求格式：
```javascript
{
	"username":"邮箱",
	"password":"密码"
}
```

若操作成功，则服务端应返回HTTP状态码`204 No Content`。

**安全提示：** 该API也可用于判断密码的正确性，因此应受到和登录API一样的速率限制。

# 参见
 * [Authentication - wiki.vg](http://wiki.vg/Authentication)
 * [Mojang API - wiki.vg](http://wiki.vg/Mojang_API)
