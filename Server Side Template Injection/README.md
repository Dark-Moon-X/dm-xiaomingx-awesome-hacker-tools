<!--
 * [业务问题]: 服务端模板注入（SSTI）是一种高危漏洞，攻击者通过在模板引擎中注入恶意代码，可以直接在服务器上执行任意命令，导致完全的系统沦陷、数据泄露或服务器接管。此漏洞常见于 PDF 生成、邮件模板和动态页面渲染场景。
 * [实现逻辑]: 本文档系统化地介绍了 SSTI 漏洞的检测与利用流程，包括识别漏洞输入点、注入模板语法、枚举模板引擎（Jinja2, Twig, Freemarker 等）以及提升为代码执行的完整攻击链，并提供了专业扫描工具（TInjA, tplmap, SSTImap）的使用方法。
 -->

# Server Side Template Injection (SSTI - 服务端模板注入)

> 模板注入允许攻击者将模板代码包含到现有（或不存在）的模板中。模板引擎通过使用静态模板文件使设计 HTML 页面变得更加容易，这些文件在运行时会将 HTML 页面中的变量/占位符替换为实际值。

## 概要 (Summary)

- [Tools](#tools)
- [Methodology](#methodology)
    - [Identify the Vulnerable Input Field](#identify-the-vulnerable-input-field)
    - [Inject Template Syntax](#inject-template-syntax)
    - [Enumerate the Template Engine](#enumerate-the-template-engine)
    - [Escalate to Code Execution](#escalate-to-code-execution)
- [Labs](#labs)
- [References](#references)


## Tools

* [Hackmanit/TInjA](https://github.com/Hackmanit/TInjA) - An effiecient SSTI + CSTI scanner which utilizes novel polyglots
  ```bash
  tinja url -u "http://example.com/?name=Kirlia" -H "Authentication: Bearer ey..."
  tinja url -u "http://example.com/" -d "username=Kirlia"  -c "PHPSESSID=ABC123..."
  ```

* [epinna/tplmap](https://github.com/epinna/tplmap) - Server-Side Template Injection and Code Injection Detection and Exploitation Tool
  ```powershell
  python2.7 ./tplmap.py -u 'http://www.target.com/page?name=John*' --os-shell
  python2.7 ./tplmap.py -u "http://192.168.56.101:3000/ti?user=*&comment=supercomment&link"
  python2.7 ./tplmap.py -u "http://192.168.56.101:3000/ti?user=InjectHere*&comment=A&link" --level 5 -e jade
  ```

* [vladko312/SSTImap](https://github.com/vladko312/SSTImap) - Automatic SSTI detection tool with interactive interface based on [epinna/tplmap](https://github.com/epinna/tplmap)
  ```powershell
  python3 ./sstimap.py -u 'https://example.com/page?name=John' -s
  python3 ./sstimap.py -u 'https://example.com/page?name=Vulnerable*&message=My_message' -l 5 -e jade
  python3 ./sstimap.py -i -A -m POST -l 5 -H 'Authorization: Basic bG9naW46c2VjcmV0X3Bhc3N3b3Jk'
  ```


## 方法论 (Methodology)

### 识别漏洞输入字段 (Identify the Vulnerable Input Field)

攻击者首先定位一个输入字段、URL 参数或应用程序的任何用户可控制部分，该部分在没有适当清理或转义的情况下被传递到服务端模板中。

例如，攻击者可能会识别一个 Web 表单、搜索栏或模板预览功能，这些功能似乎根据动态用户输入返回结果。

**提示**: 生成的 PDF 文件、发票和电子邮件通常使用模板。


### Inject Template Syntax

The attacker tests the identified input field by injecting template syntax specific to the template engine in use. Different web frameworks use different template engines (e.g., Jinja2 for Python, Twig for PHP, or FreeMarker for Java). 

Common template expressions:

* `{{7*7}}` for Jinja2 (Python).
* `#{7*7}` for Thymeleaf (Java).

Find more template expressions in the page dedicated to the technology (PHP, Python, etc).

![SSTI cheatsheet workflow](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/Images/serverside.png?raw=true)

In most cases, this polyglot payload will trigger an error in presence of a SSTI vulnerability:

```ps1
${{<%[%'"}}%\.
```

The [Hackmanit/Template Injection Table](https://github.com/Hackmanit/template-injection-table) is an interactive table containing the most efficient template injection polyglots along with the expected responses of the 44 most important template engines.


### Enumerate the Template Engine

Based on the successful response, the attacker determines which template engine is being used. This step is critical because different template engines have different syntax, features, and potential for exploitation. The attacker may try different payloads to see which one executes, thereby identifying the engine.

* **Python**: Django, Jinja2, Mako, ...
* **Java**: Freemarker, Jinjava, Velocity, ...
* **Ruby**: ERB, Slim, ...

[The post "template-engines-injection-101" from @0xAwali](https://medium.com/@0xAwali/template-engines-injection-101-4f2fe59e5756) summarize the syntax and detection method for most of the template engines for JavaScript, Python, Ruby, Java and PHP and how to differentiate between engines that use the same syntax.


### Escalate to Code Execution

Once the template engine is identified, the attacker injects more complex expressions, aiming to execute server-side commands or arbitrary code. 


## Labs

* [Root Me - Java - Server-side Template Injection](https://www.root-me.org/en/Challenges/Web-Server/Java-Server-side-Template-Injection)
* [Root Me - Python - Server-side Template Injection Introduction](https://www.root-me.org/en/Challenges/Web-Server/Python-Server-side-Template-Injection-Introduction)
* [Root Me - Python - Blind SSTI Filters Bypass](https://www.root-me.org/en/Challenges/Web-Server/Python-Blind-SSTI-Filters-Bypass)


## References

- [A Pentester's Guide to Server Side Template Injection (SSTI) - Busra Demir - December 24, 2020](https://www.cobalt.io/blog/a-pentesters-guide-to-server-side-template-injection-ssti)
- [Gaining Shell using Server Side Template Injection (SSTI) - David Valles - August 22, 2018](https://medium.com/@david.valles/gaining-shell-using-server-side-template-injection-ssti-81e29bb8e0f9)
- [Template Engines Injection 101 - Mahmoud M. Awali - November 1, 2024](https://medium.com/@0xAwali/template-engines-injection-101-4f2fe59e5756)
- [Template Injection On Hardened Targets - Lucas 'BitK' Philippe - September 28, 2022](https://youtu.be/M0b_KA0OMFw)