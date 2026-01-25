<!--
 * [业务问题]: 不安全的源代码管理（Insecure SCM）指将版本控制系统的元数据目录（如 .git, .svn）暴露在生产环境中。攻击者可下载完整源代码、历史提交记录、敏感配置和硬编码凭证，深入了解应用逻辑以发现更多漏洞。
 * [实现逻辑]: 本文档介绍了如何发现泄露的 SCM 目录（.git, .svn, .hg, .bzr），如何绕过访问限制（如 403 Forbidden），以及利用自动化工具恢复源代码的方法。
 -->

# Insecure Source Code Management (不安全的源代码管理)

> 不安全的源代码管理（SCM）可能导致 Web 应用程序和服务中出现严重的漏洞。开发人员通常依赖 Git 和 Subversion（SVN）等 SCM 系统来管理其源代码版本。然而，糟糕的安全实践，例如将 .git 和 .svn 文件夹暴露在面向互联网的生产环境中，可能会带来重大风险。

## 概要 (Summary)


## Summary

* [Methodology](#methodology)
* [Bazaar](./Bazaar.md)
* [Git](./Git.md)
* [Mercurial](./Mercurial.md)
* [Subversion](./Subversion.md)
* [Labs](#labs)
* [References](#references)


## Methodology

Exposing the version control system folders on a web server can lead to severe security risks, including: 

- **Source Code Leaks** : Attackers can download the entire source code repository, gaining access to the application's logic.
- **Sensitive Information Exposure** : Embedded secrets, configuration files, and credentials might be present within the codebase.
- **Commit History Exposure** : Attackers can view past changes, revealing sensitive information that might have been previously exposed and later mitigated.
     

The first step is to gather information about the target application. This can be done using various web reconnaissance tools and techniques. 

* **Manual Inspection** : Check URLs manually by navigating to common SCM paths.
    * http://target.com/.git/
    * http://target.com/.svn/

* **Automated Tools** : Refer to the page related to the specific technology.

Once a potential SCM folder is identified, check the HTTP response codes and contents. You might need to bypass `.htaccess` or Reverse Proxy rules.

The NGINX rule below returns a `403 (Forbidden)` response instead of `404 (Not Found)` when hitting the `/.git` endpoint.

```ps1
location /.git {
  deny all;
}
```

For example in Git, the exploitation technique doesn't require to list the content of the `.git` folder (http://target.com/.git/), the data extraction can still be conducted when files can be read.

## Labs

* [Root Me - Insecure Code Management](https://www.root-me.org/fr/Challenges/Web-Serveur/Insecure-Code-Management)


## References

- [Hidden directories and files as a source of sensitive information about web application - Apr 30, 2017](https://github.com/bl4de/research/tree/master/hidden_directories_leaks)