---
title: 如何以及为何设置您的 Stellar.toml
---

>*如果您想要但却还没有在 Stellar 网络上发布令牌，请*[*在此处查阅*](./custom-assets.md)*创建自定义令牌的详细步骤。*

您好！我们制作了这个简短的指南指导您如何使发行的令牌取得成功。

我们希望确保您知道如何向**网络提供**有关您自己和您的令牌的信息，以便潜在的买家和应用（如交易所和钱包）信任您的资产。您可以通过填写 **stellar.toml** 文件来提供此必要信息。

Stellar 上最成功的那些令牌已经遵循以下指南，*应用程序和买家将期望您的令牌也能这样做*。

为什么您应该设置您的 stellar.toml
----------------------------------------------

那些成功的令牌发行方往往向交易所和潜在的购买者提供了大量关于他们的信息。在 Stellar 中，他们可以将这些信息配置在 **stellar.toml** 中。如果您提供了足够多的信息可能会带来以下好处：

* 您的令牌能在*更多*的交易所上架

* 令牌的持有者*更加*信任您的令牌

* 您的项目很有可能会取得*更大*的成功

举例来说，Stellar 应用程序 [StellarX](http://stellarx.com/) 通过 stellar.toml 文件来决定如何在市场视图中向交易者展示令牌。如果您没有提供足够的信息，您的令牌可能会被隐藏。stellarport.io 和 stellarterm.com 等其它的 Stellar 交易所也采取了类似的措施。

*如果您没有提供一个完善的 stellar.toml，大多数交易所都不会列出您的令牌。*

stellar.toml 文件非常重要，因此第一个 Stellar 生态系统提案致力于概述它应该包含的内容。您可以在[这里](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0001.md)找到完整的 SEP 0001，我们将在下面总结其中重要的部分。

什么是 stellar.toml？
--------------------------

stellar.toml 是以 [TOML](https://github.com/toml-lang/toml) 格式（一种简单的配置文件格式）编写到一个文件，它会被发布到 https://YOUR_DOMAIN/.well-known/stellar.toml。任何人都可以查看它，该文件*证明*了您拥有托管 stellar.toml 文件的域名，也声明了您对其中列出的账户和令牌*负责*。您可以通过这个文件使您的令牌更加符合标准，也可以在这里提供关于您的组织和令牌的更多信息。**如果您提供了多种令牌，您可以将它们在一个 stellar.toml 文件中列出。**

如何完成您的 stellar.toml
---------------------------------

[SEP 0001](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0001.md) 规定了可以添加到 stellar.toml 中的五个部分：账户信息、发行人信息、联系人信息、令牌信息和验证节点信息。在这些部分中，一些字段只适用于特定的令牌，但是许多字段适用于*所有*令牌，下文中会注明哪些信息的必须提供的，哪些信息是建议提供的。

* **必需提供**：如果要在交易所上市，所有令牌发行人*必须*在其 stellar.toml 中填写此信息。

* **建议提供**：任何希望其令牌更加被用户信任的令牌发行人都应填写这些字段。

### 账户信息

*所有*令牌发行者*都*需要填写“帐户信息”部分中的一个字段：

* `ACCOUNTS`: 与令牌关联的所有 Stellar 帐户的**公钥**列表。

列出您的公钥可以让用户确认您实际上拥有它们。例如 https://google.com 托管了一个 stellar.toml 文件，用户可以确保*只有*其中列出的帐户属于 Google。如果有人说：“您需要在本月支付您的 Google 账单，请把付款发送到 GIAMGOOGLEIPROMISE”，用户查看 Google 的 stellar.toml 后发现该地址并不在其中，便明白了这个人是不可信的。

“帐户信息”部分中指定的大多数其他信息仅对验证节点和金融机构有用。

下面是一个完整的 `ACCOUNTS` 字段的示例，它列出了三个公钥：

	ACCOUNTS=[
	"GAOO3LWBC4XF6VWRP5ESJ6IBHAISVJMSBTALHOQM2EZG7Q477UWA6L7U",
	"GAENZLGHJGJRCMX5VCHOLHQXU3EMCU5XWDNU4BGGJFNLI2EL354IVBK7",
	"GB6REF5GOGGSEHZ3L2YK6K4T4KX3YDMWHDCPMV7MZJDLHBDNZXEPRBGM"
	]

### 发行方信息

将您的组织的基本信息放在 TOML 文件的 `[DOCUMENTATION]` 表中。您可以在这里向交易所和买家展示您的业务信息，也可以在这里向用户证明您的业务合法且值得信任。

您提供的信息越多，用户就越有可能信任您发行的令牌。

**必需提供**：所有发行方必须填写以下信息：

* 您的组织的法定名称 (`org_name`)，如果您的公司有一个经营名称(`org_dba`)，那么您也需要添加它。

* 您的组织的官方网站的 URL(`org_url`)。为了证明该网站是您的网站，*您必须在此处列出的相同域名中托管您的 stellar.toml。* 这样，交易所和买家就可以在您的网站上查看 SSL 证书，并可以以此判断您的身份。

* 组织 Logo 的 URL(`org_logo`) ，它将在交易所中显示在您的组织旁边。如果您未能提供此参数，在许多交易所中您的组织 Logo 将在显示为空白。

* 您的组织的实际地址(`org_physical_address`)。我们了解您可能希望将您的工作地址保密。但您至少应该把您所在的*国家*和*城市*填写在这里。如果能详细到街道地址就更好了，这可以让潜在用户更加信任您的组织。

* 您的组织的官方电话号码(`org_phone_number`)。

* 您的组织的官方 Twitter 帐号(`org_twitter`)。

* 您的组织的官方电子邮箱地址(`org_official_email`)。邮箱的邮箱域应该和您官方网站的域名相同。

**建议提供**：提供以下信息可以使您的令牌更加被用户信任：

* 您的组织的官方 GitHub 帐号(`org_github`)。

* 您的组织的官方 Keybase 帐号 (`org_keybase`)。您的 Keybase 帐户应包含您在此处列出的所有公共在线帐户和组织域名所有权的证明。

* 介绍您的组织 (`org_description`)。您可以根据自己的需要填写该值，比如在这里描述您的组织正在做什么。

交易所或许需要额外的可验证信息来决定如何向客户展示您的令牌，包含以下信息的令牌可能会获得更高的优先级：

* 证明上述的实际地址是可信的 (`org_physical_address_attestation`)。该值是一个链接到图片的 URL，这个图片是由第三方机构提供的文档（例如公用事业账单），文档中应该包含了您的组织的名字和地址。

* 证明上述的联系电话是可信的 (`org_physical_address_attestation`)。Attestation of the phone number listed above (`org_phone_number_attestation`)。该值是一个链接到图片的 URL，这个图片显示的是包含您的电话号码和组织名称的电话帐单。

以下是一份完整的发行方信息示例：

    [DOCUMENTATION]
    ORG_NAME="Organization Name"
    ORG_DBA="Organization DBA"
    ORG_URL="https://www.domain.com"
    ORG_LOGO="https://www.domain.com/awesomelogo.jpg"
    ORG_DESCRIPTION="Description of issuer"
    ORG_PHYSICAL_ADDRESS="123 Sesame St., New York, NY, 12345"
    ORG_PHYSICAL_ADDRESS_ATTESTATION="https://www.domain.com/address_attestation.jpg"
    ORG_PHONE_NUMBER="1 (123)-456-7890"
    ORG_PHONE_NUMBER_ATTESTATION="https://www.domain.com/phone_attestation.jpg"
    ORG_KEYBASE="accountname"
    ORG_TWITTER="orgtweet"
    ORG_GITHUB="orgcode"
    ORG_OFFICIAL_EMAIL="support@domain.com"

### 联系人信息

将您的组织的主要联系人的信息放置在  TOML 文件的 `[[ PRINCIPALS ]]` [表列](https://github.com/toml-lang/toml#array-of-tables)中。您至少需要输入组织中的一个人的联系信息。如果您不这样做，交易所就无法核实您的令牌，买家也可能不会对您的令牌感兴趣。多个联系人可以放置多个`[[ PRINCIPALS ]]`。

**必需提供**：所有的发行方都应该提供以下信息：

* 主要联系人的名字 (`name`)。

* 首要联系人的官方电子邮箱地址(`email`)。邮箱的邮箱域应该和您官方网站的域名相同。

* 首要联系人的 Twitter 帐号 (`twitter`)。

**建议提供**：如果主要联系人有以下信息，我们建议您填写它们：

* 首要联系人的个人 GithHb 帐号(`github`)。

* 首要联系人的个人 Keybase 帐号(`keybase`)。此帐户应包括上述电子邮件地址的所有权证明。

您提供的信息越多越好。交易所或许需要额外的可验证信息来决定如何向客户展示您的令牌，包含以下信息的令牌可能会获得更高的优先级：

* 政府签发的身份证明的照片的 SHA-256 hash (`id_photo_hash`)。

* 手持包含签名、日期和手写的 SEP 0001 信息的照片的 SHA-256 hash (`verification_photo_hash`)。

交易所和钱包可以通过 hash 确认您的联系人的身份。这些服务提供商可以私下与您联系以获取身份证和验证照片，然后根据此处列出的 hash 检查这些照片，以确保它们匹配。如果哈希值匹配，他们会让客户知道您的联系人信息已经经过了验证。

以下是一份完整的主要联系人信息示例：

    [[PRINCIPALS]]
    name="Jane Jedidiah Johnson"
    email="jane@domain.com"
    twitter="@crypto_jane"
    keybase="crypto_jane"
    github="crypto_jane"
    id_photo_hash="5g249e170f4f134b18ab3de069c5a13e5c3ef3ef90f3643afa15a1603c34cf38"
    verification_photo_hash="693687f6abd594366a09cfe6b380e58f9023867a851cc9fa71f302ab4889e48"


### 令牌信息

将您的组织的主要联系人的信息放置在 TOML 文件的 `[[ CURRENCIES ]]` [表列](https://github.com/toml-lang/toml#array-of-tables)中。如果您发行了多种令牌，您可以将这些令牌放置在一个 stellar.toml 文件中，每种令牌都有单独的`[[CURRENCIES]]` 条目。

**必需提供**：所有的发行方都需要为它们发行的令牌提供以下信息：

* 资产代码(`code`)。这是识别令牌的两个关键信息之一。如果没有它，您的令牌就不能在任何地方列出。

* 发行账户的 Stellar 公钥 (`issuer`)。这是识别令牌的第二个关键信息。如果没有它，您的令牌就不能在任何地方列出。

* 令牌的状态(`status`)：*live(存活)*、*dead(死亡)*或 *test(测试)*。如果您已经准备好了将它在交易所中上市，那您需要将它的状态设置为 *live*。如果您的令牌已准备好交易，而您未能表明其状态，则它可能不会出现在交易所中。

* 指示您的令牌是是否是锚定资产(`is_asset_anchored`)：如果您的令牌可以在 Stellar 网络之外赎回其它资产，则为 `true`；如果不能，则为`false`。交易所使用此信息按清单中的类型对令牌进行排序。如果您未能提供它，则您的令牌可能不会出现在筛选后的市场视图中。

* 客户端显示货币余额时应精确到小数点后几位(`display_decimals`)。

* 令牌的简称(`name`)。如果您没有提供该信息，那么交易所中可能无法正确显示该令牌。

您还需要通过填写以下互斥字段中的*一个*来描述您的**令牌发行政策**：

* `fixed_number`，如果您一次性发行了一定数量的令牌，且不会增发，那么您应该配置这个值。

* `max_number`，允许发行的资产数量的上限。

* `is_unlimited`，该资产是否允许增发。

**建议提供**：提供以下信息可以使您的令牌更加被用户信任：

* 您应该介绍您的令牌(`desc`)，包括它有什么用处，人们为什么会想要拥有它。

* 使用该令牌在 Stellar 网络之外赎回其它资产的条件(`conditions`)。

* 表示该令牌的图像的 URL(`image`)。没有它的话，在许多交易所中您的令牌图标将显示为空白。

以下是一份完整的令牌信息示例：

	[[CURRENCIES]]
	code="GOAT"
	issuer="GD5T6IPRNCKFOHQWT264YPKOZAWUMMZOLZBJ6BNQMUGPWGRLBK3U7ZNP"
	status="live"
	display_decimals=2
	name="goat share"
	desc="1 GOAT token entitles you to a share of revenue from Elkins Goat Farm."
	conditions="There will only ever be 10,000 GOAT tokens in existence. We will distribute the revenue share annually on Jan. 15th"
	image="https://pbs.twimg.com/profile_images/666921221410439168/iriHah4f.jpg"
	fixed_number=10000

### 锚定令牌的要求：

锚定令牌是 Stellar 生态系统中的特殊资产，因为它们可以在网络外部兑换为其他资产。如果您要发布锚定令牌，则需要提供有关这些资产的附加信息，以及介绍如何将这些令牌兑换为其它的资产。

除上面文档中提到的信息外，锚定令牌还**需要**以下字段：

* 令牌锚定的资产类型(`anchor_asset_type`)。可能的类别是*政府发行的货币*，*数字货币*，*股票*，*债券*，*大宗商品*，*房地产*及*其他*。

* 锚定的资产(`anchor_asset`)。

* 兑换这些令牌的说明(`redemption_instructions`)。

如果令牌锚定的是数字货币的话，在上市前交易所还需要**核实**以下信息：

* 持有锚定的数字货币的地址 (`collateral_addresses`)。

* 证明您确实控制这这些地址 (`collateral_address_signatures`)，[SEP 0001](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0001.md) 包含了签名数据的模板，您可以根据您的令牌的实际情况使用这个模板。

交易所使用 `collateral_address_signatures` 来验证您确实持有 `collateral_addresses`，它还会检查这些地址中的余额，如果您不能提供 100% 的资金储备，它将不会列出您的令牌。

	[[CURRENCIES]]
	code="BTC"
	issuer="GAOO3LWBC4XF6VWRP5ESJ6IBHAISVJMSBTALHOQM2EZG7Q477UWA6L7U"
	status="live"
	display_decimals=7
	name="Bitcoin"
	desc="Organization promises to purchase each BTC token from any holder for the value of 1 Bitcoin"
	conditions="Withdrawal fees apply"
	image="https://domain.com/img/Bitcoin-100x100.png"
	anchor_asset_type="crypto"
	anchor_asset="BTC"
	redemption_instructions="Use SEP6 with our federation server"
	collateral_addresses=["2C1mCx3ukix1KfegAY5zgQJV7sanAciZpv"]
	collateral_address_signatures=["304502206e21798a42fae0e854281abd38bacd1aeed3ee3738d9e1446618c4571d10"]

如何部署您的 stellar.toml 文件
--------------------------------
按照上面的步骤完成 stellar.toml 之后，将它发布到以下位置：

* https://YOUR_DOMAIN/.well-known/stellar.toml

启用 [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) 以便人们可以从其他站点访问该文件，并为 /.well-known/stellar.toml 文件请求的 HTTP 响应设置以下 HTTP 头部：

* Access-Control-Allow-Origin: *

您已经完成最后一个步骤了。现在，应用程序和买家可以通过简单的 HTTP 请求访问您提供的所有信息。

### 优秀的 stellar.toml 示例：Stronghold

如果您想要一个优秀的 stellar.toml 示例作为参考，那么去看看 [Stronghold](https://stronghold.co/.well-known/stellar.toml) 的吧。您可以轻松找到有关公司、Stellar 帐户、联系人及其令牌的所有信息，您也可以验证这些信息。

如果您的 stellar.toml 和 Stronghold 的一样完善，交易所和买家将会注意到您的令牌。

*****
