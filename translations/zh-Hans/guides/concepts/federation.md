---
title: 联邦协议
---

使用 Stellar 联邦(Federation)协议，您可以通过 Stellar 地址查询到给定用户的更多信息。这是 Stellar 的客户端软件将类似电子邮件的 Stellar 地址(例如 `name*yourdomain.com`)解析为账户 ID (例如 `GCCVPYFOHY7ZB7557JKENAX62LUAPLMGIWNZJAFV2MITK6T32V37KEJU`)的一种途径。通过使用不同域名和提供商都支持的约定语法，Stellar 地址为用户提供了一种简单的方式来共享付款详细信息。

Stellar 地址通过使用跨不同域名和提供商的互操作语法，为用户提供一种简单的共享支付详情的方式。

![使用联邦地址付款的模拟图](assets/mockup.png)

## Stellar 地址

Stellar 地址由用户名和域名组成，中间用 * 分隔开。

例如： `jed*stellar.org`：
* `jed` 是用户名
* `stellar.org` 是域名

域名可以是任何有效的 RFC 1035 域名。用户名必须是可打印的 UTF-8 字符串（空格包含在内），并且不包括以下字符：`<*,>` 。当然，域名管理员可以对其域名的用户名设置其他限制。

注意，用户名中允许使用 `@` 符号。 也就是说您可以将电子邮件地址作为用户名。例如: `maria@gmail.com*stellar.org`.

## 创建联邦域名服务

### 第一步: 创建一个 [stellar.toml](./stellar-toml.md) 文件

创建一个名为 stellar.toml 的文件，并将其置于 `https://YOUR_DOMAIN/.well-known/stellar.toml`。

### 第二步: 添加 federation_url

添加 `FEDERATION_SERVER` 到 stellar.toml 文件中，其他人可以通过它查询到联邦服务器地址。

例如： `FEDERATION_SERVER="https://api.yourdomain.com/federation"`

请注意：联邦服务器**必须**使用 `https` 协议。

### 第三步: 实现联邦查询服务

您已经在上一步指定了 `FEDERATION_SERVER`，用户可以向这个地址发起查询请求。

您可以自己实现一个联邦服务程序，也可以使用由 Stellar 开发基金会编写的[`联邦服务软件`](https://github.com/stellar/go/tree/master/services/federation)。

## 联邦服务查询请求
如果您有一个 Stellar 地址，则可以通过联邦地址服务查找账户 ID。您还可以通过账户 ID 或事务 ID 中查找 Stellar 地址，您可以通过这种方式查看谁向您发送了付款。

联邦服务查询请求是一个包含了如下参数的 `GET` 请求：

`?q=<需要查询的字符串>&type=<name,forward,id,txid>`

支持的类型：
 - **name**: 示例： `https://YOUR_FEDERATION_SERVER/federation?q=jed*stellar.org&type=name`
 - **forward**: 用于转发付款到不同的网络或不同的金融机构。根据支付的最终目标是哪种机构以及您作为转发锚所支持的内容，查询所需的其他参数将会有所不同。您应该在 [stellar.toml](./stellar-toml.md) 文件中指定 `forward` 请求中需要提供的参数。如果您无法转发该请求或是请求中的参数不正确，则应返回错误消息。示例：`https://YOUR_FEDERATION_SERVER/federation?type=forward&forward_type=bank_account&swift=BOPBPHMM&acct=2382376`
 - **id**: *没有被所有的联邦服务支持* 通过给定的账户 ID 查询 Stellar 地址。在某些情况下这种查询是不可靠的，例如锚点代替其用户提交事务，那么账户 ID 将是锚点账户，联邦服务器将无法解析出提交事务的特定用户。在这种情况下您应该使用 **txid**。示例：`https://YOUR_FEDERATION_SERVER/federation?q=GD6WU64OEP5C4LRBH6NK3MHYIA2ADN6K6II6EXPNVUR3ERBXT4AN4ACD&type=id`
 - **txid**: *没有被所有的联邦服务支持* 如果服务器记录了给定事务的提交者的联邦记录，那么服务器将会返回它。示例：`https://YOUR_FEDERATION_SERVER/federation?q=c1b368c00e9852351361e07cc58c54277e7a6366580044ab152b8db9cd8ec52a&type=txid`

### 联邦服务的响应
联邦服务器的响应包含 HTTP 状态码、HTTP 头和 JSON 消息体。

您必须在联邦服务器上启用 CORS，以便客户端可以从其他站点发送请求。 必须为联邦服务器响应设置以下 HTTP 响应头。

```
Access-Control-Allow-Origin: *
```

当一个记录被找到时，服务器应该返回 `200 OK` HTTP 状态码和下面的 JSON 消息体：

```
{
  "stellar_address": <username*domain.tld>,
  "account_id": <account_id>,
  "memo_type": <"text", "id" , or "hash"> *optional*
  "memo": <memo to attach to any payment. if "hash" type then will be base64 encoded> *optional*
}
```

如果需要重定向，联邦服务器应该返回 `3xx` HTTP状态码，并立即通过 `Location` 头将用户重定向到正确的 URL。

如果没有找到记录，应该返回 `404 Not Found` 状态码。

任何其它类型的 HTTP 状态码都应该视为有错误产生。消息体中应该包含错误详细信息：

```
{
   "detail": "extra details provided by the federation server"
}
```

## 通过账户主域名查询联邦服务提供商
账户可以设置一个[主域名](./accounts.md#home-domain)，通过该账户主域名可以查询到它的联邦服务提供商。

## 缓存
您不应该缓存来自联邦服务器的响应。一些组织可能会生成随机的账户 ID 来保护用户的隐私。 这些账户 ID 可能会随着时间的推移而改变。
