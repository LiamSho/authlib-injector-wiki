目录
=================

   * [概述](#概述)
   * [基本约定](#基本约定)
      * [请求与响应格式](#请求与响应格式)
      * [错误信息格式](#错误信息格式)
      * [数据格式](#数据格式)
      * [模型](#模型)
         * [用户](#用户)
            * [用户信息的序列化](#用户信息的序列化)
         * [角色（Profile）](#角色profile)
            * [角色信息的序列化](#角色信息的序列化)
            * [材质URL规范](#材质url规范)
         * [令牌（Token）](#令牌token)
            * [令牌的状态](#令牌的状态)
   * [Yggdrasil API](#yggdrasil-api)
      * [用户部分](#用户部分)
         * [登录](#登录)
         * [刷新](#刷新)
         * [验证令牌](#验证令牌)
         * [吊销令牌](#吊销令牌)
         * [登出](#登出)
      * [会话部分](#会话部分)
         * [客户端进入服务器](#客户端进入服务器)
         * [服务端验证客户端](#服务端验证客户端)
      * [角色部分](#角色部分)
         * [查询单个角色](#查询单个角色)
         * [批量查询角色](#批量查询角色)
   * [扩展API](#扩展api)
      * [服务端信息获取](#服务端信息获取)
   * [参见](#参见)

# 概述
本文旨在为实现Yggdrasil服务端提供非官方的技术规范。

本规范中所描述的服务端行为不一定与Mojang服务端的相同。
这客观上是因为Mojang的服务端是闭源的，我们只能推测其内部逻辑，而所作出的推测难免会与实际存在出入。
但事实上只要客户端能够正确理解并处理服务端的响应，那么其行为是否与Mojang服务端的相同，也就无关紧要了。

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
|试图使用一个错误的角色加入服务器|403|ForbiddenOperationException|Invalid token.|


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

`signature`是一个Base64字符串，其中包含属性值（使用UTF-8编码）的数字签名（使用SHA1withRSA算法，见[PKCS #1](https://www.rfc-editor.org/rfc/rfc2437.txt)）。关于签名密钥的详细介绍，见[签名密钥对](https://github.com/to2mbn/authlib-injector/wiki/%E7%AD%BE%E5%90%8D%E5%AF%86%E9%92%A5%E5%AF%B9)。

角色属性中目前已知的键有`textures`（并不一定会包含）。它对应的值是一个Base64字符串，内容为JSON字符串，包含角色的材质信息，格式如下：
```javascript
{
	"timestamp":该属性值被生成时的时间戳（Java时间戳格式，即自1970-01-01 00:00:00 UTC至今经过的毫秒数）,
	"profileId":"角色UUID（无符号）",
	"profileName":"角色名称",
	"textures":{ // 角色的材质
		"材质类型（如SKIN）":{ // 若角色不具有该项材质，则不必包含
			"url":"材质的URL",
			"metadata":{ // 材质的元数据，若没有则不必包含
				"键":"值"
				// ,...（可以有更多）
			}
		}
		// ,...（可以有更多）
	}
}
```
材质元数据中目前已知的键有`model`。
`model`只出现在`SKIN`材质中，用于表示该角色的模型。它的值可以为`default`或`slim`，其中`default`代表STEVE，`slim`代表ALEX。

#### 材质URL规范
Minecraft将材质hash作为材质的标识。每当客户端下载一个材质后，便会将其缓存在本地，以后若需要相同hash的材质，则会直接使用缓存。
而这个hash并不是由客户端计算的。Yggdrasil服务端应先计算好材质hash，将其作为材质URL的文件名，即从URL最后一个`/`（不包括）开始一直到结尾的这一段子串。
而客户端会直接将URL的文件名作为材质的hash。

例如下面这个URL，它所代表的材质的hash为`e051c27e803ba15de78a1d1e83491411dffb6d7fd2886da0a6c34a2161f7ca99`：
```
https://yggdrasil.example.com/textures/e051c27e803ba15de78a1d1e83491411dffb6d7fd2886da0a6c34a2161f7ca99
```

由于PNG格式图像包含与显示无关的数据，因此即使是图像尺寸与内容完全相同的PNG文件，它们的hash值也可能不同。
为此，需要使用一个仅与图像内容有关的方法来计算材质的hash。规定这个方法如下：
 1. 首先创建一个长度为`(width * height * 4 + 8)`字节的缓冲区，其中`width`和`height`为图像的长和宽
 2. 填充该缓冲区
     1. `0~3`字节为`width`，以大端序存储
     2. `4~7`字节为`height`，以大端序存储
     3. 对于每一个像素，设其坐标为`(x, y)`，其首地址`offset`为`((y + x * height) * 4 + 8)`
         1. 第`(offset + 0)`、`(offset + 1)`、`(offset + 2)`、`(offset + 3)`个字节分别为该像素的Alpha、Red、Green、Blue分量
	     2. 若Alpha分量为`0x00`（透明），则RGB分量皆作为`0x00`处理
 3. 计算以上缓冲区内数据的`SHA-256`，作为材质的hash

> 目前无法确定Mojang所使用的hash算法，其输出长度（61个hex字符）不属于任何已知hash算法。
> 如果使用SHA-256作为hash算法，由于输出长度不同，不会与Mojang的hash算法发生冲突，因此是可行的。

<details>
<summary>Java实现示例</summary>

```java
public static String textureHash(BufferedImage img) throws Exception {
	MessageDigest digest = MessageDigest.getInstance("SHA-256");
	int width = img.getWidth();
	int height = img.getHeight();
	byte[] buf = new byte[4096];

	putInt(buf, 0, width); // 0~3: width(big-endian)
	putInt(buf, 4, height); // 4~7: height(big-endian)
	int pos = 8;
	for (int x = 0; x < width; x++) {
		for (int y = 0; y < height; y++) {
			// pos+0: alpha
			// pos+1: red
			// pos+2: green
			// pos+3: blue
			putInt(buf, pos, img.getRGB(x, y));
			if (buf[pos + 0] == 0) {
				// the pixel is transparent
				buf[pos + 1] = buf[pos + 2] = buf[pos + 3] = 0;
			}
			pos += 4;
			if (pos == buf.length) {
				// buffer is full
				pos = 0;
				digest.update(buf, 0, buf.length);
			}
		}
	}
	if (pos > 0) {
		// flush
		digest.update(buf, 0, pos);
	}

	byte[] sha256 = digest.digest();
	return String.format("%0" + (sha256.length << 1) + "x", new BigInteger(1, sha256)); // to hex
}

// put an int into the array in big-endian
private static void putInt(byte[] array, int offset, int x) {
	array[offset + 0] = (byte) (x >> 24 & 0xff);
	array[offset + 1] = (byte) (x >> 16 & 0xff);
	array[offset + 2] = (byte) (x >> 8 & 0xff);
	array[offset + 3] = (byte) (x >> 0 & 0xff);
}
```

</details>

<details>
<summary>JavaScript实现示例</summary>

> 本实现使用了[pngjs](https://github.com/lukeapage/pngjs)。

```javascript
let crypto = require("crypto");
let PNG = require("pngjs").PNG;
let fs = require("fs");

function computeTextureHash(image) {
	const bufSize = 8192;
	let hash = crypto.createHash("sha256");
	let buf = Buffer.allocUnsafe(bufSize);
	let width = image.width;
	let height = image.height;
	buf.writeUInt32BE(width, 0);
	buf.writeUInt32BE(height, 4);
	let pos = 8;
	for (let x = 0; x < width; x++) {
		for (let y = 0; y < height; y++) {
			let imgidx = (width * y + x) << 2;
			let alpha = image.data[imgidx + 3];
			buf.writeUInt8(alpha, pos + 0);
			if (alpha === 0) {
				buf.writeUInt8(0, pos + 1);
				buf.writeUInt8(0, pos + 2);
				buf.writeUInt8(0, pos + 3);
			} else {
				buf.writeUInt8(image.data[imgidx + 0], pos + 1);
				buf.writeUInt8(image.data[imgidx + 1], pos + 2);
				buf.writeUInt8(image.data[imgidx + 2], pos + 3);
			}
			pos += 4;
			if (pos === bufSize) {
				pos = 0;
				hash.update(buf);
			}
		}
	}
	if (pos > 0) {
		hash.update(buf.slice(0, pos));
	}
	return hash.digest("hex");
}

console.info(computeTextureHash(PNG.sync.read(fs.readFileSync("texture-hash-test.png"))));
```

</details>

<details>
<summary>测试样例</summary>

> 样例输入：[texture-hash-test.png](https://raw.githubusercontent.com/wiki/to2mbn/authlib-injector/texture-hash-test.png)
>
> 样例输出：`47a4c518f80f94ad8737713e0325a98e1f2647f962b9a646f58cd0bbd5afe683`

</details>


建议所有Yggdrasil服务端实现都应将上述算法作为材质hash的计算方法。
这样可以确保即使相同的材质来自不同的Yggdrasil服务端，它们的hash也是相同的，进而避免客户端不必要的重复下载和存储。

**安全提示：** 无论如何，服务端都不应直接存储用户上传的材质。服务端应当将用户上传的材质读入后，去除其中任何与显示无关的信息（如[PNG中的辅助区块](https://tools.ietf.org/html/rfc2083#page-19)），并重新以PNG格式写入。

### 令牌（Token）
令牌与账号为多对一关系。令牌是一种登录凭证，具有时效性。令牌具有以下属性：
 * accessToken
 * clientToken
 * 绑定的角色
 * 颁发时间

其中`accessToken`和`clientToken`为无符号UUID。`accessToken`由服务端随机生成，`clientToken`由客户端提供。

介于`accessToken`的随机性，它可以被作为主键。而`clientToken`不具有唯一性。

绑定的角色可以为空。它代表了能使用该令牌进行游戏的角色。

一个用户可以同时有多个令牌，但服务端也应该对令牌数量加以限制。当令牌数量超出限制（如10个）时，则应先吊销最旧的令牌，之后再颁发新的令牌。

#### 令牌的状态
令牌有以下三种状态：
 * **有效**
   * 处于该状态的令牌可以进行各项操作，如[进服验证](#会话部分)、[刷新](#刷新)。
   * 新颁发的令牌（即通过[登录](#登录)、[刷新](#刷新)颁发的令牌）处于该状态。
 * **暂时失效**
   * 处于该状态的令牌除了进行[刷新操作](#刷新)外，无权进行任何操作。
   * _该状态并不是必须要实现的（详细介绍见下）。_
 * **无效**
   * 处于该状态的令牌无权进行任何操作。
   * 令牌被吊销后处于该状态。这里的吊销包括[显式吊销](#吊销令牌)、[登出](#登出)、[刷新](#刷新)后吊销原令牌、令牌过期。

令牌的状态只能由有效变为无效，或是由有效变为暂时失效再变为无效，这个过程是不可逆的。
刷新操作仅颁发一个新的令牌，并不能使原令牌重新回到有效状态。

令牌应当有一个过期时限（如15天）。当自颁发起所经过的时间超过该时限时，令牌过期。

> **关于暂时失效状态**
>
> Mojang对暂时失效状态的实现是这样的：
> 对启动器而言，若令牌处于暂时失效状态，则会刷新令牌，获得一个新的处于有效状态的令牌；
> 对Yggdrasil服务端而言，仅最后颁发的令牌才是有效的，先前颁发的其它令牌都处于暂时失效状态。
>
> Mojang之所以这么做，可能是为了防止用户多地同时登录（仅使最后一个session有效）。但事实上，即使服务端没有实现暂时失效状态，启动器的逻辑也是可以正常工作的。
>
> 当然，就算我们要实现暂时失效状态，也并不需要以Mojang的实现为范本。只需要启动器能够正确处理，任何实现都是可以的。下面给出一个不同于Mojang的实现的例子：
>> 取一个短于令牌过期时限的时间段作为有效和暂时失效的分界点。若自颁发起经过的时间在该时限内，则令牌有效；若超过该时限，但仍在过期时限内，则令牌暂时失效。
>>
>> 这种做法实现了这样的功能：玩家如果经常进行登录操作，除了第一次登录就不需要输入密码了。而当他长时间未登录时则需要重新输入密码。


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
		// ,... 每一项为一个角色（格式见 §角色信息的序列化）
	],
	"selectedProfile":{
		// ... 绑定的角色，若为空，则不需要包含（格式见 §角色信息的序列化）
	},
	"user":{
		// ... 用户信息（仅当请求中requestUser为true时包含，格式见 §用户信息的序列化）
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
		// ... 要选择的角色（可选，格式见 §角色信息的序列化）
	}
}
```

当指定`clientToken`时，服务端应检查`accessToken`和`clientToken`是否有效，否则只需要检查`accessToken`。

颁发的新令牌的`clientToken`应与原令牌的相同。`selectedProfile`只有在原令牌绑定的角色为空时才能包含，新令牌应绑定到该项目指定的角色上。

该操作在令牌暂时失效时依然可以执行。若请求失败，原令牌依然有效。

响应格式：
```javascript
{
	"accessToken":"新令牌的accessToken",
	"clientToken":"新令牌的clientToken",
	"selectedProfile":{
		// ... 新令牌绑定的角色，若为空，则不需要包含（格式见 §角色信息的序列化）
	},
	"user":{
		// ... 用户信息（仅当请求中requestUser为true时包含，格式见 §用户信息的序列化）
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

若令牌有效，服务端应返回HTTP状态`204 No Content`，否则作为令牌无效的异常情况处理。

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

无论操作是否成功，服务端应返回HTTP状态`204 No Content`。

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

若操作成功，服务端应返回HTTP状态`204 No Content`。

**安全提示：** 该API也可用于判断密码的正确性，因此应受到和登录API一样的速率限制。

## 会话部分
该部分用于角色进入服务器时的验证。验证流程如下：

 1. **Minecraft服务端**随机生成一段字符串，发送给**Minecraft客户端**
 2. **Minecraft客户端**将该字符串及令牌发送给**Yggdrasil服务端**（要求令牌有效）
 3. **Minecraft服务端**请求**Yggdrasil服务端**检查客户端会话的有效性，即是否成功进行第2步

### 客户端进入服务器
`POST /sessionserver/session/minecraft/join`

记录服务端发送给客户端的随机字符串，以备服务端检查。

请求格式：
```javascript
{
	"accessToken":"令牌的accessToken",
	"selectedProfile":"该令牌绑定的角色的UUID（无符号）",
	"serverId":"服务端发送给客户端的随机字符串"
}
```

仅当`accessToken`有效，且`selectedProfile`与令牌所绑定的角色一致时，操作才成功。

服务端应记录以下信息：
 * serverId
 * accessToken
 * 发送该请求的客户端IP

实现时请注意：以上信息应记录在内存数据库中（如Redis），且应该设置过期时间（如30秒）。
介于`serverId`的随机性，可以将其作为主键。

若操作成功，服务端应返回HTTP状态`204 No Content`。

### 服务端验证客户端
`GET /sessionserver/session/minecraft/hasJoined?username={username}&serverId={serverId}&ip={ip}`

检查客户端会话的有效性，即数据库中是否存在该`serverId`的记录，且信息正确。

请求参数：

|参数|值|
|----|--|
|username|角色的名称|
|serverId|服务端发送给客户端的随机字符串|
|ip _（可选）_|在Minecraft服务端处获取到的客户端IP|

`username`需要与`serverId`对应的令牌所绑定的角色的名称相同。

若`ip`参数存在，仅当其值与先前发送[进入服务器请求](#客户端进入服务器)的客户端IP相同时，操作才成功。

响应格式：
```javascript
{
	// ... 令牌所绑定的角色（包含角色属性及数字签名，格式见 §角色信息的序列化）
}
```

若操作失败，服务端应返回HTTP状态`204 No Content`。

## 角色部分
该部分用于角色信息的查询。

### 查询单个角色
`GET /sessionserver/session/minecraft/profile/{uuid}?unsigned={unsigned}`

查询指定角色的信息。

请求参数：

|参数|值|
|----|--|
|uuid|角色的UUID（无符号）|
|unsigned _（可选）_|`true`或`false`。是否在响应中**不包含**数字签名，默认为`true`|

响应格式：
```javascript
{
	// ... 角色信息（包含角色属性。若unsigned为true，还需要包含数字签名。格式见 §角色信息的序列化）
}
```

若角色不存在，服务端应返回HTTP状态`204 No Content`。

### 批量查询角色
`POST /api/profiles/minecraft`

查询多个角色的信息。

请求格式：
```javascript
[
	"角色名称1"
	// ,... 还可以有更多
]
```

服务端查询各个角色名称对应的角色信息，并包含在响应中。对于不存在的角色，不需要包含。响应中角色的先后次序无要求。

响应格式：
```javascript
[
	{
		// 角色信息（注意：不包含角色属性。格式见 §角色信息的序列化）
	}
	// ,...（可以有更多）
]
```

**安全提示：** 为防止CC攻击，需要为单次查询的角色数目设置最大值，该值至少为2。

# 扩展API
以下API并不属于Yggdrasil，它们是为了方便authlib-injector进行自动配置而设计的。如果服务端实现了以下API，authlib-injector只需要API URL就可以自动配置其他的项目。

## 服务端信息获取
`GET /`

响应格式：
```javascript
{
	"meta":{
		// 服务端的元数据，内容任意
	},
	"skinDomains":[ // 加入皮肤白名单的域名后缀
		"皮肤域名后缀1"
		// ,...（可以有更多）
	],
	"signaturePublickey":"用于验证数字签名的公钥"
}
```

关于`skinDomains`和`signaturePublickey`的详细介绍，可以见[authlib-injector.example.yaml](https://github.com/to2mbn/authlib-injector/blob/master/authlib-injector.example.yaml)

> **`signaturePublickey`的格式**：
> 其内容可以为多行，但行首与行末，以及除头尾两行（`-----BEGIN PUBLIC KEY...`）外每行的中间，都不允许出现空格。

尽管`meta`中内容没有强制要求，但我们建议您包含以下属性：

|Key|Value|
|---|-----|
|serverName|服务器名称|
|implementationName|服务端实现的名称|
|implementationVersion|服务端实现的版本|

下面给出一段响应的示例：
```javascript
{
	"meta":{
		"serverName":"to2mbn Minecraft Server",
		"implementationName":"akir",
		"implementationVersion":"1.0.0"
	},
	"skinDomains":[
		".to2mbn.org"
	],
	"signaturePublickey":"-----BEGIN PUBLIC KEY-----\nMIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEAylB4B6m5lz7jwrcFz6Fd\n/fnfUhcvlxsTSn5kIK/2aGG1C3kMy4VjhwlxF6BFUSnfxhNswPjh3ZitkBxEAFY2\n5uzkJFRwHwVA9mdwjashXILtR6OqdLXXFVyUPIURLOSWqGNBtb08EN5fMnG8iFLg\nEJIBMxs9BvF3s3/FhuHyPKiVTZmXY0WY4ZyYqvoKR+XjaTRPPvBsDa4WI2u1zxXM\neHlodT3lnCzVvyOYBLXL6CJgByuOxccJ8hnXfF9yY4F0aeL080Jz/3+EBNG8RO4B\nyhtBf4Ny8NQ6stWsjfeUIvH7bU/4zCYcYOq4WrInXHqS8qruDmIl7P5XXGcabuzQ\nstPf/h2CRAUpP/PlHXcMlvewjmGU6MfDK+lifScNYwjPxRo4nKTGFZf/0aqHCh/E\nAsQyLKrOIYRE0lDG3bzBh8ogIMLAugsAfBb6M3mqCqKaTMAf/VAjh5FFJnjS+7bE\n+bZEV0qwax1CEoPPJL1fIQjOS8zj086gjpGRCtSy9+bTPTfTR/SJ+VUB5G2IeCIt\nkNHpJX2ygojFZ9n5Fnj7R9ZnOM+L8nyIjPu3aePvtcrXlyLhH/hvOfIOjPxOlqW+\nO5QwSFP4OEcyLAUgDdUgyW36Z5mB285uKW/ighzZsOTevVUG2QwDItObIV6i8RCx\nFbN2oDHyPaO5j1tTaBNyVt8CAwEAAQ==\n-----END PUBLIC KEY-----"
}
```

# 参见
 * [Authentication - wiki.vg](http://wiki.vg/Authentication)
 * [Mojang API - wiki.vg](http://wiki.vg/Mojang_API)
 * [Protocol - wiki.vg](http://wiki.vg/Protocol)
