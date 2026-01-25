<!--
 * [业务问题]: HTTP 参数污染（HPP）是一种 Web 攻击规避技术，攻击者通过在同一参数名称的多个实例之间分割攻击向量（?param1=value&param1=value）来绕过基于模式的安全机制（WAF）、操纵 Web 逻辑或检索隐藏信息。
 * [实现逻辑]: 本文档详细介绍了 HPP 攻击的原理和不同 Web 技术对重复参数的解析行为差异表（ASP.NET 读取所有、PHP 读取最后一个、JSP 读取第一个等），展示了如何利用这种差异绕过 WAF 并执行注入攻击。
 -->

# HTTP Parameter Pollution (HTTP 参数污染)

> HTTP 参数污染（HPP）是一种 Web 攻击规避技术，允许攻击者精心构造 HTTP 请求以操纵 Web 逻辑或检索隐藏信息。这种规避技术基于在具有相同名称的参数的多个实例之间分割攻击向量（?param1=value&param1=value）。由于没有正式的 HTTP 参数解析方式，各个 Web 技术都有自己独特的解析和读取同名 URL 参数的方式。有些读取第一次出现，有些读取最后一次出现，有些将其读取为数组。攻击者滥用这种行为以绕过基于模式的安全机制。

## 概要 (Summary)

* [Tools](#tools)
* [How to test](#how-to-test)
    * [Table of reference](#table-of-reference)
* [References](#references)


## Tools

No tools needed. Maybe Burp or OWASP ZAP.

## How to test

HPP allows an attacker to bypass pattern based/black list proxies or Web Application Firewall detection mechanisms. This can be done with or without the knowledge of the web technology behind the proxy, and can be achieved through simple trial and error. 

```
Example scenario.
WAF - Reads first param
Origin Service - Reads second param. In this scenario, developer trusted WAF and did not implement sanity checks.

Attacker -- http://example.com?search=Beth&search=' OR 1=1;## --> WAF (reads first 'search' param, looks innocent. passes on) --> Origin Service (reads second 'search' param, injection happens if no checks are done here.)
```

### Table of reference

When ?par1=a&par1=b

| Technology                                      | Parsing Result          |outcome (par1=)|
| ------------------                              |---------------          |:-------------:|
| ASP.NET/IIS                                     |All occurrences          |a,b            |
| ASP/IIS                                         |All occurrences          |a,b            |
| PHP/Apache                                      |Last occurrence          |b              |
| PHP/Zues                                        |Last occurrence          |b              |
| JSP,Servlet/Tomcat                              |First occurrence         |a              |
| Perl CGI/Apache                                 |First occurrence         |a              |
| Python Flask                                    |First occurrence         |a              |
| Python Django                                   |Last occurrence          |b              |
| Nodejs                                          |All occurrences          |a,b            |
| Golang net/http - `r.URL.Query().Get("param")`  |First occurrence         |a              |
| Golang net/http - `r.URL.Query()["param"]`      |All occurrences in array |['a','b']      |
| IBM Lotus Domino                                |First occurrence         |a              |
| IBM HTTP Server                                 |First occurrence         |a              |
| Perl CGI/Apache                                 |First occurrence         |a              |
| mod_wsgi (Python)/Apache                        |First occurrence         |a              |
| Python/Zope                                     |All occurrences in array |['a','b']      |
| Ruby on Rails                                   |Last occurrence          |b              |


## References

- [How to Detect HTTP Parameter Pollution Attacks - Acunetix - January 9, 2024](https://www.acunetix.com/blog/whitepaper-http-parameter-pollution/)
- [HTTP Parameter Pollution - Itamar Verta - December 20, 2023](https://www.imperva.com/learn/application-security/http-parameter-pollution/)
- [HTTP Parameter Pollution in 11 minutes - PwnFunction - January 28, 2019](https://www.youtube.com/watch?v=QVZBl8yxVX0&ab_channel=PwnFunction)