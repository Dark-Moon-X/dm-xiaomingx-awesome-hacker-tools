<!--
 * [业务问题]: DNS 重绑定（DNS Rebinding）通过动态改变攻击者控制的域名的 IP 地址，将其指向目标应用的 IP，从而绕过浏览器的同源策略（SOP），允许攻击者读取内部应用的响应、窃取敏感数据或执行未经授权的操作。
 * [实现逻辑]: 本文档详细介绍了 DNS 重绑定攻击的完整流程，包括使用 Singularity 框架进行自动化攻击、绕过 DNS 保护的多种技术（0.0.0.0、CNAME、localhost）以及如何检测服务是否易受攻击的方法。
 -->

# DNS Rebinding (DNS 重绑定攻击)

> DNS 重绑定通过将攻击者控制的机器名称的 IP 地址更改为目标应用程序的 IP 地址，绕过[同源策略](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)，从而允许浏览器向目标应用程序发起任意请求并读取其响应。

## 概要 (Summary)

* [Tools](#tools)
* [Methodology](#methodology)
* [Protection Bypasses](#protection-bypasses)
    * [0.0.0.0](#0000)
    * [CNAME](#CNAME)
    * [localhost](#localhost)
* [References](#references)


## Tools

- [nccgroup/singularity](https://github.com/nccgroup/singularity) - A DNS rebinding attack framework. 
- [rebind.it](http://rebind.it/) - Singularity of Origin Web Client.


## Methodology

First, we need to make sure that the targeted service is vulnerable to DNS rebinding.
It can be done with a simple curl request:

```bash
curl --header 'Host: <arbitrary-hostname>' http://<vulnerable-service>:8080
```

If the server returns the expected result (e.g. the regular web page) then the service is vulnerable.
If the server returns an error message (e.g. 404 or similar), the server has most likely protections implemented which prevent DNS rebinding attacks.

Then, if the service is vulnerable, we can abuse DNS rebinding by following these steps:

1. Register a domain.
2. [Setup Singularity of Origin](https://github.com/nccgroup/singularity/wiki/Setup-and-Installation).
3. Edit the [autoattack HTML page](https://github.com/nccgroup/singularity/blob/master/html/autoattack.html) for your needs.
4. Browse to "http://rebinder.your.domain:8080/autoattack.html".
5. Wait for the attack to finish (it can take few seconds/minutes).


## Protection Bypasses

> Most DNS protections are implemented in the form of blocking DNS responses containing unwanted IP addresses at the perimeter, when DNS responses enter the internal network. The most common form of protection is to block private IP addresses as defined in RFC 1918 (i.e. 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16). Some tools allow to additionally block localhost (127.0.0.0/8), local (internal) networks, or 0.0.0.0/0 network ranges.

In the case where DNS protection are enabled (generally disabled by default), NCC Group has documented multiple [DNS protection bypasses](https://github.com/nccgroup/singularity/wiki/Protection-Bypasses) that can be used.

### 0.0.0.0

We can use the IP address 0.0.0.0 to access the localhost (127.0.0.1) to bypass filters blocking DNS responses containing 127.0.0.1 or 127.0.0.0/8.

### CNAME

We can use DNS CNAME records to bypass a DNS protection solution that blocks all internal IP addresses.
Since our response will only return a CNAME of an internal server,
the rule filtering internal IP addresses will not be applied.
Then, the local, internal DNS server will resolve the CNAME.

```bash
$ dig cname.example.com +noall +answer
; <<>> DiG 9.11.3-1ubuntu1.15-Ubuntu <<>> example.com +noall +answer
;; global options: +cmd
cname.example.com.            381     IN      CNAME   target.local.
```

### localhost

We can use "localhost" as a DNS CNAME record to bypass filters blocking DNS responses containing 127.0.0.1.

```bash
$ dig www.example.com +noall +answer
; <<>> DiG 9.11.3-1ubuntu1.15-Ubuntu <<>> example.com +noall +answer
;; global options: +cmd
localhost.example.com.            381     IN      CNAME   localhost.
```


## References

- [How Do DNS Rebinding Attacks Work? - nccgroup - Apr 9, 2019](https://github.com/nccgroup/singularity/wiki/How-Do-DNS-Rebinding-Attacks-Work%3F)
