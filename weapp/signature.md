# 签名加密·小程序

https://mp.weixin.qq.com/debug/wxadoc/dev/api/signature.html

## 数据签名校验

为了确保开放接口返回用户数据的安全性，微信会对明文数据进行签名。开发者可以根据业务需要对数据包进行签名校验，确保数据的完整性。

1. 签名校验算法涉及用户的 `session\_key`，通过 `wx.login` 登录流程获取用户 `session\_key`，并自行维护与应用自身登录态的对应关系。
2. 通过调用接口（如 `wx.getUserInfo`）获取数据时，接口会同时返回 `rawData`、`signature`，其中 `signature = sha1( rawData + session_key )`
3. 开发者将 `signature`、`rawData` 发送到开发者服务器进行校验。服务器利用用户对应的 `session_key` 使用相同的算法计算出签名 `signature2` ，比对 `signature` 与 `signature2` 即可校验数据的完整性。

如 `wx.getUserInfo` 的数据校验：

接口返回的 `rawData`：

```json
{
  "nickName": "Band",
  "gender": 1,
  "language": "zh_CN",
  "city": "Guangzhou",
  "province": "Guangdong",
  "country": "CN",
  "avatarUrl": "http://wx.qlogo.cn/mmopen/vi_32/1vZvI39NWFQ9XM4LtQpFrQJ1xlgZxx3w7bQxKARol6503Iuswjjn6nIGBiaycAjAtpujxyzYsrztuuICqIM5ibXQ/0"
}
```

用户的 `session-key`:

```
HyVFkGl5F5OQWJZZaNzBBg==
```

> 注意：session_key 是微信服务器生成的针对用户数据加密签名的密钥，不应该传输到客户端。

所以，用于签名的字符串为：

```
{"nickName":"Band","gender":1,"language":"zh_CN","city":"Guangzhou","province":"Guangdong","country":"CN","avatarUrl":"http://wx.qlogo.cn/mmopen/vi_32/1vZvI39NWFQ9XM4LtQpFrQJ1xlgZxx3w7bQxKARol6503Iuswjjn6nIGBiaycAjAtpujxyzYsrztuuICqIM5ibXQ/0"}HyVFkGl5F5OQWJZZaNzBBg==
```

使用 sha1 得到的结果为：

```
75e81ceda165f4ffa64f4068af58c64b8f54b88c
```

## 加密数据解密算法

接口如果涉及敏感数据（如wx.getUserInfo当中的 openId 和unionId ），接口的明文内容将不包含这些敏感数据。开发者如需要获取敏感数据，需要对接口返回的加密数据( encryptedData )进行对称解密。 解密算法如下：

1. 对称解密使用的算法为 AES-128-CBC，数据采用PKCS#7填充。
2. 对称解密的目标密文为 Base64_Decode(encryptedData)。
3. 对称解密秘钥 aeskey = Base64_Decode(session_key), aeskey 是16字节。
4. 对称解密算法初始向量为 Base64_Decode(iv)，其中iv由数据接口返回。

微信官方提供了多种编程语言的示例代码（点击下载）。每种语言类型的接口名字均一致。调用方式可以参照示例。

另外，为了应用能校验数据的有效性，我们会在敏感数据加上数据水印( watermark )

watermark参数说明：

| 参数 | 类型 | 说明 |
| --- | --- | --- |
| watermark | OBJECT | 数据水印 |
| appid | String | 敏感数据归属 appid，开发者可校验此参数与自身appid是否一致 |
| timestamp | DateInt | 敏感数据获取的时间戳, 开发者可以用于数据时效性校验 |

如接口 wx.getUserInfo 敏感数据当中的 watermark：

```json
{
    "openId": "OPENID",
    "nickName": "NICKNAME",
    "gender": GENDER,
    "city": "CITY",
    "province": "PROVINCE",
    "country": "COUNTRY",
    "avatarUrl": "AVATARURL",
    "unionId": "UNIONID",
    "watermark":
    {
        "appid":"APPID",
        "timestamp":TIMESTAMP
    }
}
```

注：此前提供的加密数据（encryptData）以及对应的加密算法将被弃用，请开发者不要再依赖旧逻辑。