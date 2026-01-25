<!--
 * [业务问题]: 不安全的反序列化是一种严重的安全风险，攻击者通过操纵序列化后的数据流，在应用程序重新构建对象时触发非预期的逻辑（如远程代码执行 RCE）。
 * [实现逻辑]: 本文档系统化地梳理了主流编程语言（Java, PHP, Python, Ruby, .NET 等）中反序列化漏洞的识别特征（Header）、利用工具（ysoserial, phpggc 等）及 POP 链构建思路。
 -->

# Insecure Deserialization (不安全的反序列化)

> 序列化是将对象转换为可以稍后恢复的数据格式的过程。人们通常为了将对象保存到存储器中，或作为通信的一部分进行发送而对对象进行序列化。反序列化则是该过程的逆过程 —— 从某种格式的结构化数据中提取信息，并将其重新构建为对象。 —— OWASP

## 概要 (Summary)

* [Deserialization Identifier](#deserialization-identifier)
* [POP Gadgets](#pop-gadgets)
* [Labs](#labs)
* [References](#references)


## Deserialization Identifier

Check the following sub-sections, located in other chapters :

* [Java deserialization : ysoserial, ...](Java.md)
* [PHP (Object injection) : phpggc, ...](PHP.md)
* [Ruby : universal rce gadget, ...](Ruby.md)
* [Python : pickle, ...](Python.md)
* [YAML : PyYAML, ...](YAML.md)
* [.NET : ysoserial.net, ...](DotNET.md)

| Object Type     | Header (Hex) | Header (Base64) |
|-----------------|--------------|-----------------|
| Java Serialized | AC ED        | rO              |
| .NET ViewState  | FF 01        | /w              |
| Python Pickle   | 80 04 95     | gASV            |
| PHP Serialized  | 4F 3A        | Tz              |


## POP 链 (POP Gadgets)

> POP（面向属性编程，Property Oriented Programming）Gadget 是应用程序类中实现的一段代码，可以在反序列化过程中被调用。

POP Gadgets 的特点：
* Can be serialized
* Has public/accessible properties
* Implements specific vulnerable methods
* Has access to other "callable" classes


## Labs

* [PortSwigger - Modifying serialized objects](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-objects)
* [PortSwigger - Modifying serialized data types](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-data-types)
* [PortSwigger - Using application functionality to exploit insecure deserialization](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-using-application-functionality-to-exploit-insecure-deserialization)
* [PortSwigger - Arbitrary object injection in PHP](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-arbitrary-object-injection-in-php)
* [PortSwigger - Exploiting Java deserialization with Apache Commons](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-exploiting-java-deserialization-with-apache-commons)
* [PortSwigger - Exploiting PHP deserialization with a pre-built gadget chain](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-exploiting-php-deserialization-with-a-pre-built-gadget-chain)
* [PortSwigger - Exploiting Ruby deserialization using a documented gadget chain](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-exploiting-ruby-deserialization-using-a-documented-gadget-chain)
* [PortSwigger - Developing a custom gadget chain for Java deserialization](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-developing-a-custom-gadget-chain-for-java-deserialization)
* [PortSwigger - Developing a custom gadget chain for PHP deserialization](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-developing-a-custom-gadget-chain-for-php-deserialization)
* [PortSwigger - Using PHAR deserialization to deploy a custom gadget chain](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-using-phar-deserialization-to-deploy-a-custom-gadget-chain)
* [NickstaDB - DeserLab](https://github.com/NickstaDB/DeserLab)


## References

- [ExploitDB Introduction - Abdelazim Mohammed(@intx0x80) - May 27, 2018](https://www.exploit-db.com/docs/english/44756-deserialization-vulnerability.pdf)
- [Exploiting insecure deserialization vulnerabilities - PortSwigger - July 25, 2020](https://portswigger.net/web-security/deserialization/exploiting)
- [Instagram's Million Dollar Bug - Wesley Wineberg - December 17, 2015](http://www.exfiltrated.com/research-Instagram-RCE.php)