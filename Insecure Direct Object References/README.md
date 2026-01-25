<!--
 * [业务问题]: 不安全的直接对象引用（IDOR）是一种访问控制漏洞，攻击者通过操纵 URL 或 API 请求中的参数（如用户 ID、订单号）直接访问或修改未经授权的资源，导致敏感数据泄露、账户接管或数据篡改。
 * [实现逻辑]: 本文档系统化地整理了 IDOR 漏洞的多种利用场景，包括数字型参数递增、常见标识符猜测、弱伪随机数生成器、哈希参数破解、通配符参数利用等，并提供了实战技巧（如 HTTP 方法变换、参数污染）和真实案例。
 -->

# Insecure Direct Object References (IDOR - 不安全的直接对象引用)

> 不安全的直接对象引用（IDOR）是一种安全漏洞，当应用程序允许用户基于用户提供的输入直接访问或修改对象（如文件、数据库记录或 URL）而没有足够的访问控制时就会发生。这意味着如果用户更改 URL 或 API 请求中的参数值（如 ID），他们可能能够访问或操纵他们无权查看或修改的数据。


## 概要 (Summary)

* [Tools](#tools)
* [Methodology](#methodology)
    * [Numeric Value Parameter](#numeric-value-parameter)
    * [Common Identifiers Parameter](#common-identifiers-parameter) 
    * [Weak Pseudo Random Number Generator](#weak-pseudo-random-number-generator) 
    * [Hashed Parameter](#hashed-parameter)
    * [Wildcard Parameter](#wildcard-parameter)
    * [IDOR Tips](#idor-tips)
* [Labs](#labs)
* [References](#references)


## Tools

- [PortSwigger/BApp Store > Authz](https://portswigger.net/bappstore/4316cc18ac5f434884b2089831c7d19e)
- [PortSwigger/BApp Store > AuthMatrix](https://portswigger.net/bappstore/30d8ee9f40c041b0bfec67441aad158e)
- [PortSwigger/BApp Store > Autorize](https://portswigger.net/bappstore/f9bbac8c4acf4aefa4d7dc92a991af2f)


## 方法论 (Methodology)

IDOR 代表不安全的直接对象引用。这是一种安全漏洞类型，当应用程序基于用户提供的输入提供对对象的直接访问时就会出现。因此，攻击者可以绕过授权并直接访问系统中的资源，可能导致未经授权的信息披露、修改或删除。

**Example of IDOR**

Imagine a web application that allows users to view their profile by clicking a link `https://example.com/profile?user_id=123`:

```php
<?php
    $user_id = $_GET['user_id'];
    $user_info = get_user_info($user_id);
    ...
```

Here, `user_id=123` is a direct reference to a specific user's profile. If the application doesn't properly check that the logged-in user has the right to view the profile associated with `user_id=123`, an attacker could simply change the `user_id` parameter to view other users' profiles:

```ps1
https://example.com/profile?user_id=124
```

![https://lh5.googleusercontent.com/VmLyyGH7dGxUOl60h97Lr57F7dcnDD8DmUMCZTD28BKivVI51BLPIqL0RmcxMPsmgXgvAqY8WcQ-Jyv5FhRiCBueX9Wj0HSCBhE-_SvrDdA6_wvDmtMSizlRsHNvTJHuy36LG47lstLpTqLK](https://raw.githubusercontent.com/swisskyrepo/PayloadsAllTheThings/master/Insecure%20Direct%20Object%20References/Images/idor.png)


### Numeric Value Parameter

Increment and decrement these values to access sensitive information.

* Decimal value: `287789`, `287790`, `287791`, ...
* Hexadecimal: `0x4642d`, `0x4642e`, `0x4642f`, ...
* Unix epoch timestamp: `1695574808`, `1695575098`, ...

**Examples** 

* [HackerOne - IDOR to view User Order Information - meals](https://hackerone.com/reports/287789)
* [HackerOne - Delete messages via IDOR - naaash](https://hackerone.com/reports/697412)

### Common Identifiers Parameter

Some identifiers can be guessed like names and emails, they might grant you access to customer data.

* Name: `john`, `doe`, `john.doe`, ...
* Email: `john.doe@mail.com`
* Base64 encoded value: `am9obi5kb2VAbWFpbC5jb20=`

**Examples** 

* [HackerOne - Insecure Direct Object Reference (IDOR) - Delete Campaigns - datph4m](https://hackerone.com/reports/1969141)


### Weak Pseudo Random Number Generator

* UUID/GUID v1 can be predicted if you know the time they were created: `95f6e264-bb00-11ec-8833-00155d01ef00`
* MongoDB Object Ids are generated in a predictable manner: `5ae9b90a2c144b9def01ec37`
    * a 4-byte value representing the seconds since the Unix epoch
    * a 3-byte machine identifier
    * a 2-byte process id
    * a 3-byte counter, starting with a random value

**Examples** 

* [HackerOne - IDOR allowing to read another user's token on the Social Media Ads service - a_d_a_m](https://hackerone.com/reports/1464168)
* [IDOR through MongoDB Object IDs Prediction](https://techkranti.com/idor-through-mongodb-object-ids-prediction/)


### Hashed Parameter

Sometimes we see websites using hashed values to generate a random user id or token, like `sha1(username)`, `md5(email)`, ...

* MD5: `098f6bcd4621d373cade4e832627b4f6`
* SHA1: `a94a8fe5ccb19ba61c4c0873d391e987982fbbd3`
* SHA2: `9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08`

**Examples** 

* [IDOR with Predictable HMAC Generation - DiceCTF 2022 - CryptoCat](https://youtu.be/Og5_5tEg6M0)


### Wildcard Parameter

Send a wildcard (`*`, `%`, `.`, `_`) instead of an ID, some backend might respond with the data of all the users.

* `GET /api/users/* HTTP/1.1`
* `GET /api/users/% HTTP/1.1`
* `GET /api/users/_ HTTP/1.1`
* `GET /api/users/. HTTP/1.1`


**Examples** 

* [TODO]()


### IDOR Tips

* Change the HTTP request: `POST → PUT`
* Change the content type: `XML → JSON`
* Transform numerical values to arrays: `{"id":19} → {"id":[19]}`
* Use Parameter Pollution: `user_id=hacker_id&user_id=victim_id`


## Labs

- [PortSwigger - Insecure Direct Object References](https://portswigger.net/web-security/access-control/lab-insecure-direct-object-references)


## References

- [From Christmas present in the blockchain to massive bug bounty - Jesse Lakerveld - March 21, 2018](http://web.archive.org/web/20180401130129/https://www.vicompany.nl/magazine/from-christmas-present-in-the-blockchain-to-massive-bug-bounty)
- [How-To: Find IDOR (Insecure Direct Object Reference) Vulnerabilities for large bounty rewards - Sam Houton - November 9, 2017](https://www.bugcrowd.com/blog/how-to-find-idor-insecure-direct-object-reference-vulnerabilities-for-large-bounty-rewards/)
- [Hunting Insecure Direct Object Reference Vulnerabilities for Fun and Profit (PART-1) - Mohammed Abdul Raheem - February 2, 2018](https://codeburst.io/hunting-insecure-direct-object-reference-vulnerabilities-for-fun-and-profit-part-1-f338c6a52782)
- [IDOR - how to predict an identifier? Bug bounty case study - Bug Bounty Reports Explained - September 21, 2023](https://youtu.be/wx5TwS0Dres)
- [Insecure Direct Object Reference Prevention Cheat Sheet - OWASP - July 31, 2023](https://www.owasp.org/index.php/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet)
- [Insecure direct object references (IDOR) - PortSwigger - December 25, 2019](https://portswigger.net/web-security/access-control/idor)
- [Testing for IDORs - PortSwigger - October 29, 2024](https://portswigger.net/burp/documentation/desktop/testing-workflow/access-controls/testing-for-idors)
- [Testing for Insecure Direct Object References (OTG-AUTHZ-004) - OWASP - August 8, 2014](https://www.owasp.org/index.php/Testing_for_Insecure_Direct_Object_References_(OTG-AUTHZ-004))
- [The Rise of IDOR - HackerOne - April 2, 2021](https://www.hackerone.com/company-news/rise-idor)
- [Web to App Phone Notification IDOR to view Everyone's Airbnb Messages - Brett Buerhaus - March 31, 2017](http://buer.haus/2017/03/31/airbnb-web-to-app-phone-notification-idor-to-view-everyones-airbnb-messages/)