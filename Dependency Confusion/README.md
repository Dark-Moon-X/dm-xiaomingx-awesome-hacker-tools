<!--
 * [业务问题]: 依赖混淆（Dependency Confusion）是一种供应链攻击，攻击者通过在公共仓库（npm、pip、gem）中注册与企业内部私有包同名的恶意包，诱使安装脚本下载并执行恶意代码，导致远程代码执行或数据泄露。曾影响 Apple、Microsoft 等多家大型企业。
 * [实现逻辑]: 本文档介绍了依赖混淆攻击的完整方法论，包括如何发现私有包名称、在公共仓库注册同名包以及等待企业系统自动安装的攻击流程，并提供了检测工具（confused）和真实案例（Alex Birsan 的 $130,000+ 漏洞赏金）。
 -->

# Dependency Confusion (依赖混淆 / 供应链替换攻击)

> 依赖混淆攻击或供应链替换攻击发生在软件安装脚本被欺骗从公共仓库拉取恶意代码文件，而不是从内部仓库拉取同名的预期文件时。

## 概要 (Summary)

* [Tools](#tools)
* [Methodology](#methodology)
    * [NPM Example](#npm-example)
* [References](#references)


## Tools

* [visma-prodsec/confused](https://github.com/visma-prodsec/confused) - Tool to check for dependency confusion vulnerabilities in multiple package management systems 


## Methodology

Look for `npm`, `pip`, `gem` packages, the methodology is the same : you register a public package with the same name of private one used by the company and then you wait for it to be used.


### NPM Example

* List all the packages (ie: package.json, composer.json, ...)
* Find the package missing from https://www.npmjs.com/
* Register and create a **public** package with the same name
    * Package example : https://github.com/0xsapra/dependency-confusion-expoit


## References

- [Exploiting Dependency Confusion - Aman Sapra (0xsapra) - 2 Jul 2021](https://0xsapra.github.io/website//Exploiting-Dependency-Confusion)
- [Dependency Confusion: How I Hacked Into Apple, Microsoft and Dozens of Other Companies - Alex Birsan - 9 Feb 2021](https://medium.com/@alex.birsan/dependency-confusion-4a5d60fec610)
- [3 Ways to Mitigate Risk When Using Private Package Feeds - Microsoft - 29/03/2021](https://web.archive.org/web/20210210121930/https://azure.microsoft.com/en-gb/resources/3-ways-to-mitigate-risk-using-private-package-feeds/)
- [$130,000+ Learn New Hacking Technique in 2021 - Dependency Confusion - Bug Bounty Reports Explained - 22 févr. 2021](https://www.youtube.com/watch?v=zFHJwehpBrU)