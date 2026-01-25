<!--
 * [业务问题]: Web 应用程序通常具有未在用户界面中公开的隐藏或未记录的参数。发现这些参数可能揭示调试功能、管理接口或易受攻击的端点，导致未经授权的访问、信息泄露或权限提升。
 * [实现逻辑]: 本文档介绍了发现隐藏参数的两种主要方法：暴力破解参数（使用 Arjun、x8、param-miner 等工具和专业字典）以及通过 Wayback Machine 和 JS 文件分析发现旧参数的方法。
 -->

# HTTP Hidden Parameters (HTTP 隐藏参数发现)

> Web 应用程序通常具有未在用户界面中公开的隐藏或未记录的参数。模糊测试可以帮助发现这些参数，这些参数可能容易受到各种攻击。

## 概要 (Summary)

* [Tools](#tools)
* [Methodology](#methodology)
    * [Bruteforce Parameters](#bruteforce-parameters)
    * [Old Parameters](#old-parameters)
* [References](#references)


## Tools

* [PortSwigger/param-miner](https://github.com/PortSwigger/param-miner) - Burp extension to identify hidden, unlinked parameters.
* [s0md3v/Arjun](https://github.com/s0md3v/Arjun) - HTTP parameter discovery suite
* [Sh1Yo/x8](https://github.com/Sh1Yo/x8) - Hidden parameters discovery suite
* [tomnomnom/waybackurls](https://github.com/tomnomnom/waybackurls) - Fetch all the URLs that the Wayback Machine knows about for a domain
* [devanshbatham/ParamSpider](https://github.com/devanshbatham/ParamSpider) - Mining URLs from dark corners of Web Archives for bug hunting/fuzzing/further probing


## Methodology

### Bruteforce Parameters

* Use wordlists of common parameters and send them, look for unexpected behavior from the backend. 
    ```ps1
    x8 -u "https://example.com/" -w <wordlist>
    x8 -u "https://example.com/" -X POST -w <wordlist>
    ```

Wordlist examples: 

- [Arjun/large.txt](https://github.com/s0md3v/Arjun/blob/master/arjun/db/large.txt)
- [Arjun/medium.txt](https://github.com/s0md3v/Arjun/blob/master/arjun/db/medium.txt)
- [Arjun/small.txt](https://github.com/s0md3v/Arjun/blob/master/arjun/db/small.txt)
- [samlists/sam-cc-parameters-lowercase-all.txt](https://github.com/the-xentropy/samlists/blob/main/sam-cc-parameters-lowercase-all.txt)
- [samlists/sam-cc-parameters-mixedcase-all.txt](https://github.com/the-xentropy/samlists/blob/main/sam-cc-parameters-mixedcase-all.txt)


### Old Parameters

Explore all the URL from your targets to find old parameters.

* Browse the [Wayback Machine](http://web.archive.org/)
* Look through the JS files to discover unused parameters


## References

- [Hacker tools: Arjun – The parameter discovery tool - Intigriti - May 17, 2021](https://blog.intigriti.com/2021/05/17/hacker-tools-arjun-the-parameter-discovery-tool/)
- [Parameter Discovery: A quick guide to start - YesWeHack - April 20, 2022](http://web.archive.org/web/20220420123306/https://blog.yeswehack.com/yeswerhackers/parameter-discovery-quick-guide-to-start)