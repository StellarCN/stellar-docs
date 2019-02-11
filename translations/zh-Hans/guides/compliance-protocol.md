---
title: 合规协议
---

# 合规协议

反洗钱法(AML)要求金融机构(FIs)不仅要知道客户向谁汇款，还要知道收款人是谁。在某些司法管辖区，银行可信赖其他持牌银行的反洗钱程序。但在另外一些司法管辖区，每家银行都必须对汇款人和收款人进行检查。合规协议用于处理上述场景。

金融机构之间可以自己协定交换哪些客户信息，但一般需要以下信息：
 - 全名
 - 生日
 - 住址

执行合规协议是执行[联邦协议](https://stellar-docs.overcat.me/guides/concepts/federation.html)之后的另一个步骤。在此步骤中，发送机构联系接收机构以获得发送事务的许可。为此，接收机构创建一个 `AUTH_SERVER` 并将其端点地址添加到自己的 [stellar.toml](https://stellar-docs.overcat.me/guides/concepts/stellar-toml.html) 中。

您可以自己实现合规协议服务，也可以使用我们编写好的一个[简单的合规协议服务软件](https://github.com/stellar/bridge-server/blob/master/readme_compliance.md)。

## AUTH_SERVER(认证服务器)

`AUTH_SERVER` 提供一个端点，该端点由发送方调用，该端点会告知发送机构这笔付款是否被批准。`AUTH_SERVER` 的端点地址应该在您的 [stellar.toml](https://www.stellar.org/developers/guides/concepts/stellar-toml.html) 中声明。

#### 请求

要向接收方发送事务数据，请将数据用 `Content-Type`: `application/x-www-form-urlencoded` 编码后以 HTTP POST 的方式发送给认证服务器：
```
data=<data value>&sig=<sig value>
```

**数据**是包含如下字段的 JSON 体：

字段名 | 数据类型 | 描述 
-----|-----------|------------
`sender` | string | 发送方的地址。例如：`bob*bank.com` 
`need_info` | boolean | 是否需要提供收款方的反洗钱信息。 
`tx` | string: base64 encoded [xdr.Transaction](https://github.com/stellar/stellar-core/blob/4961b8bb4a64c68838632c5865389867e9f02840/src/xdr/Stellar-transaction.x#L297-L322) | 发送机构希望以 XDR 格式提交的事务。此事务是未签署的，它的序列号应该为 0。 
`attachment` | string | 附件全文。这个附件的 hash 被包含在事务的备注(Memo)中。该字段的内容应该符合 [Stellar 附件公约](./attachment.md)，并且应该提供足够的信息以供收款机构能对发送方进行制裁检查。 

**sig** 是发送机构对发送数据的签名。接送机构需要使用发送机构的签名公钥对签名进行检查，以确认数据的确是由发送方签署的。发送机构的签名公钥可以在 [stellar.toml](https://www.stellar.org/developers/guides/concepts/stellar-toml.html) 的 `SIGNING_KEY` 字段中找到。

请求体示例 (请注意这里面包含了 `data` 和 `sig`)：
```
data=%7B%22sender%22%3A%22aldi%2AbankA.com%22%2C%22need_info%22%3Atrue%2C%22tx%22%3A%22AAAAAEhAArfpmUJYq%2FQ9SFAH3YDzNLJEBI9i9TXmJ7s608xbAAAAZAAMon0AAAAJAAAAAAAAAAPUg1%2FwDrMDozn8yfiCA8LLC0wF10q5n5lo0GiFQXpPsAAAAAEAAAAAAAAAAQAAAADdvkoXq6TXDV9IpguvNHyAXaUH4AcCLqhToJpaG6cCyQAAAAAAAAAAAJiWgAAAAAA%3D%22%2C%22attachment%22%3A%22%7B%5C%22nonce%5C%22%3A%5C%221488805458327055805%5C%22%2C%5C%22transaction%5C%22%3A%7B%5C%22sender_info%5C%22%3A%7B%5C%22address%5C%22%3A%5C%22678+Mission+St%5C%22%2C%5C%22city%5C%22%3A%5C%22San+Francisco%5C%22%2C%5C%22country%5C%22%3A%5C%22US%5C%22%2C%5C%22first_name%5C%22%3A%5C%22Aldi%5C%22%2C%5C%22last_name%5C%22%3A%5C%22Dobbs%5C%22%7D%2C%5C%22route%5C%22%3A%5C%221%5C%22%2C%5C%22note%5C%22%3A%5C%22%5C%22%2C%5C%22extra%5C%22%3A%5C%22%5C%22%7D%2C%5C%22operations%5C%22%3Anull%7D%22%7D&sig=KgvyQTZsZQoaMy8jdwCUfLayfgfFMUdZJ%2B0BIvEwiH5aJhBXvhV%2BipRok1asjSCUS%2FUaGeGKDoizS1%2BtFiiyAA%3D%3D
```

在对 `data` 解码后您可以获得以下数据：

```json
{
  "sender": "aldi*bankA.com",
  "need_info": true,
  "tx": "AAAAAEhAArfpmUJYq/Q9SFAH3YDzNLJEBI9i9TXmJ7s608xbAAAAZAAMon0AAAAJAAAAAAAAAAPUg1/wDrMDozn8yfiCA8LLC0wF10q5n5lo0GiFQXpPsAAAAAEAAAAAAAAAAQAAAADdvkoXq6TXDV9IpguvNHyAXaUH4AcCLqhToJpaG6cCyQAAAAAAAAAAAJiWgAAAAAA=",
  "attachment": "{\"nonce\":\"1488805458327055805\",\"transaction\":{\"sender_info\":{\"address\":\"678 Mission St\",\"city\":\"San Francisco\",\"country\":\"US\",\"first_name\":\"Aldi\",\"last_name\":\"Dobbs\"},\"route\":\"1\",\"note\":\"\",\"extra\":\"\"},\"operations\":null}"
}
```

请注意，事务的备注(Memo) 是 `tx` 的 sha256 hash。

#### 回应

`AUTH_SERVER` 会返回一个包含以下字段的 JSON 体：

字段名 | 数据类型 | 描述 
-----|-----------|------------
`info_status` | `ok`, `denied`, `pending` | 该金融机构是否愿意分享反洗钱信息。 
`tx_status` | `ok`, `denied`, `pending` | 该金融机构是否愿意接收该交易。 
`dest_info` | string | *(只有当 `info_status`为`ok` 时才包含该参数)* 编码处理过的反洗钱信息。 
`pending` | integer | *(只有当 `info_status` 或 `tx_status` 为 `pending` 时才包含该参数)* 发送机构应该在多少秒后再次检查回应的状态。发送机构应该在给定的秒数之后重新提交这个请求。 

*回应示例：*

```json
{
    "info_status": "ok",
    "tx_status": "pending",
    "dest_info": "{\"name\": \"John Doe\"}",
    "pending": 3600
}
```

----



## 流程示例：
在这个例子中，Aldi `aldi*bankA.com` 想要付款给 `bogart*bankB.com`：

**1) BankA 获取与 BankB 交互所需的信息**

这是通过查询 BankB `stellar.toml` 文件完成的。

BankA -> 获取 `https://bankB.com/.well-known/stellar.toml`

从 .toml 文件中可以获取 BankB 的以下信息：
 - `FEDERATION_SERVER`
 - `AUTH_SERVER`

**2) BankA 获取 Bogart 的路由信息，这样它就可以开始构建事务了**

这里是通过 BankB 的联邦服务(FEDERATION_SERVER)查询 `bogart*bankB.com` 完成的。

BankA -> `FEDERATION_SERVER?type=name&q=bogart*bankB.com`

查阅[联邦服务](https://www.stellar.org/developers/guides/concepts/federation.html)了解更多的信息，联邦服务返回了以下信息：
 - Bogart 在机构中的账户 ID
 - Bogart 的路径信息

联邦服务的回应示例：
```json
{
  "stellar_address": "bogart*bankB.com",
  "account_id": "GDJ2GYMIQRIPTJZXQAVE5IM675ITLBAMQJS7AEFIWM4HZNGHVXOZ3TZK",
  "memo_type": "id",
  "memo": 1
}
```

**3) BankA 向 BankB 提出授权请求**

向 BankB 请求 Bogart 的反洗钱信息及向 Bogart 发送付款的权限。

BankA -> `AUTH_SERVER`

请求主体示例(请注意它包含参数`数据`和 `sig`) ：
```
data=%7B%22sender%22%3A%22aldi%2AbankA.com%22%2C%22need_info%22%3Atrue%2C%22tx%22%3A%22AAAAAEhAArfpmUJYq%2FQ9SFAH3YDzNLJEBI9i9TXmJ7s608xbAAAAZAAMon0AAAAJAAAAAAAAAAPUg1%2FwDrMDozn8yfiCA8LLC0wF10q5n5lo0GiFQXpPsAAAAAEAAAAAAAAAAQAAAADdvkoXq6TXDV9IpguvNHyAXaUH4AcCLqhToJpaG6cCyQAAAAAAAAAAAJiWgAAAAAA%3D%22%2C%22attachment%22%3A%22%7B%5C%22nonce%5C%22%3A%5C%221488805458327055805%5C%22%2C%5C%22transaction%5C%22%3A%7B%5C%22sender_info%5C%22%3A%7B%5C%22address%5C%22%3A%5C%22678+Mission+St%5C%22%2C%5C%22city%5C%22%3A%5C%22San+Francisco%5C%22%2C%5C%22country%5C%22%3A%5C%22US%5C%22%2C%5C%22first_name%5C%22%3A%5C%22Aldi%5C%22%2C%5C%22last_name%5C%22%3A%5C%22Dobbs%5C%22%7D%2C%5C%22route%5C%22%3A%5C%221%5C%22%2C%5C%22note%5C%22%3A%5C%22%5C%22%2C%5C%22extra%5C%22%3A%5C%22%5C%22%7D%2C%5C%22operations%5C%22%3Anull%7D%22%7D&sig=KgvyQTZsZQoaMy8jdwCUfLayfgfFMUdZJ%2B0BIvEwiH5aJhBXvhV%2BipRok1asjSCUS%2FUaGeGKDoizS1%2BtFiiyAA%3D%3D
```

在解码 `data` 之后，您可以看到它包含了以下参数：

```json
{
  "sender": "aldi*bankA.com",
  "need_info": true,
  "tx": "AAAAAEhAArfpmUJYq/Q9SFAH3YDzNLJEBI9i9TXmJ7s608xbAAAAZAAMon0AAAAJAAAAAAAAAAPUg1/wDrMDozn8yfiCA8LLC0wF10q5n5lo0GiFQXpPsAAAAAEAAAAAAAAAAQAAAADdvkoXq6TXDV9IpguvNHyAXaUH4AcCLqhToJpaG6cCyQAAAAAAAAAAAJiWgAAAAAA=",
  "attachment": "{\"nonce\":\"1488805458327055805\",\"transaction\":{\"sender_info\":{\"address\":\"678 Mission St\",\"city\":\"San Francisco\",\"country\":\"US\",\"first_name\":\"Aldi\",\"last_name\":\"Dobbs\"},\"route\":\"1\",\"note\":\"\",\"extra\":\"\"},\"operations\":null}"
}
```

请注意，事务的备注(Memo)值是 `tx` 的 sha256 hash，付款地址是由联邦服务提供的。您可以使用 [XDR 浏览器](https://www.stellar.org/laboratory/#xdr-viewer?input=AAAAAEhAArfpmUJYq%2FQ9SFAH3YDzNLJEBI9i9TXmJ7s608xbAAAAZAAMon0AAAAJAAAAAAAAAAPUg1%2FwDrMDozn8yfiCA8LLC0wF10q5n5lo0GiFQXpPsAAAAAEAAAAAAAAAAQAAAADdvkoXq6TXDV9IpguvNHyAXaUH4AcCLqhToJpaG6cCyQAAAAAAAAAAAJiWgAAAAAA%3D&type=Transaction&network=test) 查验上述的事务。

**4) BankB 处理授权请求**

 - BankB 将发送方的地址(`aldi*bankA.com`)分成两部分：`aldi` 和  `bankA.com`
 - BankB -> 从 `https://bankA.com/.well-known/stellar.toml`
   中获取 BankA 的 `SIGNING_KEY`
 - BankB 使用 BankA 的 `SIGNING_KEY` 对签名进行校验，以确认这个请求确实是由 BankA 发起的。
 - BankB 对 Aldi 进行制裁检查。检查结果会决定 `tx_status` 的值。
 - BankB 通过以下条件决定是否披露鲍嘉的反洗钱信息:
   - Bogart 公开了他的信息
   - Bogart 允许 BankA 查看他的信息
   - Bogart 允许 Aldi 查看他的信息
   - BankB 允许 BankA 查看用户信息
 - 如果上述条件都不成立，BankB 需要询问 Bogart 时候愿意向 BankA 披露自己的信息并接收付款。在这种情况下，BankB 的授权服务器将会回应 `info_status: "pending"`，并给 Borgart 一定的时间决定是否应该披露信息。
 - 如果 BankB 认为可以与 BankA 共享反洗钱信息，它会将这些信息包含在 `dest_info` 中，然后发送给 BankA。

查阅[认证服务器的回应](#response)了解其它可能的返回值。

回应示例：

```json
{
    "info_status": "ok",
    "tx_status": "ok",
    "dest_info": "{\"name\": \"Bogart Doe\"}",
}
```

**5) BankA 处理授权服务器的回应**

如果 AUTH_SERVER 返回的是 `pending`，BankA 需要在一段时间后再次发起请求。

**6) BankA 进行制裁检查**

一旦 BankA 收到了 BankB 提供的 `dest_info` ，BankA 将会使用反洗钱信息对 Bogart 进行制裁检查。一旦检查通过，BankA 会对事务进行签署并将它提交到 Stellar 网络。

**7) BankB 处理收款**

 - 它根据缓存检查事务 hash，或者重新执行对发送方的制裁检查。
 - 它将收到的付款记录在 Bogart 的账户中，或者退回这笔付款。
