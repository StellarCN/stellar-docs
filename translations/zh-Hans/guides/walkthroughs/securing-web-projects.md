---
title: 保障 Web 项目安全
---

对于任何管理加密货币的应用程序来说，遵循最佳的安全实践来构建应用程序是至关重要的。加密货币相关的应用程序一直是攻击者的目标，因为他们能够立即从漏洞中获得金钱收益。下面的检查表为最常见的漏洞提供了指导。但即使遵循了一下每一条建议，安全性也不能得到绝对保证。网络安全是不断发展的，我们需要持续不断地在里面投入精力。

# SSL/TLS

确保你的站点启用了 TLS（你可以通过 Letsencrypt 来免费启用它），并将 http 请求重定向到 https。这能有效的抵挡中间人攻击，保障敏感数据在服务器与浏览器之间安全的传输。请点击[这里](https://letsencrypt.org/getting-started/)来了解如何获取免费的 SSL 证书。

*如果你还没有启用 SSL/TLS，请立刻启用它。*

# 内容安全策略（Content Security Policy, CSP）头部

CSP 头会告诉浏览器它可以从哪里下载静态资源。比如说你正在访问的
astralwallet.io 站点向 myevilsite.com 请求了一个 JavaScript 文件，如果后者不在 CSP 头部的白名单的话，你的浏览器将会禁止这个行为。您可以在[此处](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)了解有关如何实现 CSP 头部的信息。

大部分的 Web 框架都支持通过配置文件或拓展来配置 CSP 策略。举例来说，如果你使用的是 Node.js，那你可以试试 [Helmet](https://www.npmjs.com/package/helmet)。

这可以防止像 [Blackwallet
Hack](https://www.ccn.com/yet-another-crypto-wallet-hack-causes-users-lose-400000/) 这样的黑客事件发生。

# HTTP 严格安全传输（HTTP Strict-Transport-Security, HSTS）Headers

这个 HTTP 头会告诉浏览器：未来载入这个网站时需要使用 HTTPS。要实现这点，你需要将这个[头部](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security)添加到你的站点。一些 Web 框架内置了这个功能，比如 [Django](https://docs.djangoproject.com/en/2.0/topics/security/#ssl-https)。

这可以防止像 [MyEtherWallet DNS
hack](https://bitcoinmagazine.com/articles/popular-ether-wallet-mew-hijacked-dns-attack/) 这样的黑客事件发生。


# 储存敏感数据

我们并不建议你存储太多的用户敏感数据。但如果你一定要储存他们的话，请务必小心行事。在存储敏感数据方面有许多策略：首先确保敏感数据使用经过验证的方法进行加密，比如 AES-256，并与应用程序数据分开存储，始终选择 AEAD 模式。应用服务器和私密服务器之间的任何通信都应该在私有网络中进行，并且/或者通过 HMAC 进行身份验证。密码策略将根据是否通过网络多次发送密文而变化。最后，备份任何离线使用的加密密钥，并仅在内存中储存它们。

咨询一位优秀的密码学家并阅读最佳实践。查看你喜欢的 Web 框架的文档或许是一个好的起点。

使用你自己的加密算法绝不是一个好注意，你应该选择那些被广泛使用并且经过了严格测试的加密库。[NaCl](https://en.wikipedia.org/wiki/NaCl_(software))是一个优秀的库。

# 监控
攻击者经常需要花费一些时间在你的网站上寻找漏洞。防御性地检查日志可以帮助你理解他们试图实现的目标，并确保你受到保护。至少，您可以阻止他们的 IP，或者根据您发现的可疑行为自动屏蔽。

最后，配置错误报告是非常有价值的（如 [Sentry](https://sentry.io/welcome/)）：攻击者在尝试入侵时通常会触发各种奇怪的 Bug。

# 脆弱的身份认证

如果您有用户登录系统，那么安全地构建身份验证系统是至关重要的。当然，最好的方法就是使用现成的东西。Ruby on Rails 和 Django 都有健壮的内置身份验证方案。

许多 JSON Web Token 的实现都很糟糕，所以请务必对使用到的第三方库进行审计。

使用经过时间考验的哈希密码方案。最新一次哈希密码竞赛的获胜者是 Argon2，Balloon Hashing 也值得一看。

强烈建议启用二步验证（2FA），需要启用 U2F 或 [TOTP](https://tools.ietf.org/html/rfc6238) 2FA 才能进行敏感操作。

2FA 真的非常重要。电子邮件帐户通常不是非常安全。拥有第二个身份验证因素可确保无意中保持登录状态或密码泄漏的用户仍然受到保护。

最后，一个高强度的密码是必须的。常见的和简短的密码可以很容易被暴力破解。Dropbox 推出了一个[很棒的开源工具](https://blogs.dropbox.com/tech/2012/04/zxcvbn-realistic-password-strength-estimation/)，它可以很快地测量密码的强度。

# 分布式拒绝服务攻击攻击（Denial of Service Attacks, DOS）

拒绝服务攻击通常是通过向 Web 服务器发送超出它能承受的流量来实现的。为了降低此风险，请限制来自单个 IP 和浏览器指纹的流量。有时攻击者会使用代理来绕过 IP 速率限制，总之，恶意攻击者总能找到掩盖自己身份的方法。因此阻止DOS攻击的最可靠方法是在客户端进行检查，或使用像 [Cloudflare](https://www.cloudflare.com/ddos/) 这样的托管服务。

# 关闭未使用的端口

攻击者通常会通过扫描你的服务器的端口查看是否有漏洞。像 Heroku 这样的服务已经帮你关闭了端口，你也可以阅读[这篇文章](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/authorizing-access-to-an-instance.html)以了解如何在 AWS 上关闭端口。

# 网络钓鱼与社会工程攻击

网络钓鱼攻击能够摧毁任何良好的安全基础架构。在你的网站上发布明确的政策，并在用户注册时将这些信息展示出来（比如：你永远不会要求他们提供密码等）。向用户发送消息时对消息进行签名。提示用户检查他们所在网站的域名。

# 扫描你的网站与库以查找漏洞

使用诸如 [Snyk](https://snyk.io/) 这样的工具来检查你使用的第三方包是否有漏洞。确保您的第三方库保持最新——开发者通常会在发现漏洞之后释出一个新版本以修复漏洞。

你也可以使用 [Mozilla Observatory](https://observatory.mozilla.org/) 来检查你的站点的 HTTP 安全性。

# 可轻易实现的目标：跨站请求伪造（Cross-Site Request Forgery Protection, CSRF），SQL 注入

大多数现代 Web 和移动框架都能有 CSRF 保护和防 SQL 注入。请确保启用了 CSRF 保护，并且使用 ORM 对数据库进行操作，而不是直接使用用户输入的原始 SQL。请参阅 [Ruby on Rails 文档](http://guides.rubyonrails.org/security.html#sql-injection)对 SQL 注入的说明。

# 总结

我们希望这份指南能使 Web 环境更为安全。本指南并不是面面俱到的，但是如果你还没有在安全性方面付出行动的话，它是一个很好的起点。请记住，安全性的强弱取决于它最薄弱的环节。如果你的密码很糟糕，黑客可以猜到你的 AWS 密码，那么所有这些都将变得毫无意义。