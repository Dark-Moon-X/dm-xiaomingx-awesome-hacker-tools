# CONTRIBUTING

PayloadsAllTheThings' Team :heart: pull requests.

Feel free to improve with your payloads and techniques !

You can also contribute with a :beers: IRL, or using the [sponsor](https://github.com/sponsors/swisskyrepo) button.

## Pull Requests Guidelines

In order to provide the safest payloads for the community, the following rules must be followed for **every** Pull Request.

- Payloads must be sanitized
    - Use `id`, and `whoami`, for RCE Proof of Concepts
    - Use `[REDACTED]` when the user has to replace a domain for a callback. E.g: XSSHunter, BurpCollaborator etc.
    - Use `10.10.10.10` and `10.10.10.11` when the payload require IP addresses
    - Use `Administrator` for privileged users and `User` for normal account
    - Use `P@ssw0rd`, `Password123`, `password` as default passwords for your examples
    - Prefer commonly used name for machines such as `DC01`, `EXCHANGE01`, `WORKSTATION01`, etc
- References must have an `author`, a `title`, a `link` and a `date`
    - Use [Wayback Machine](wayback.archive.org) if the reference is not available anymore.
    - The date must be following the format `Month Number, Year`, e.g: `December 25, 2024`
    - References to Github repositories must follow this format: `[author/tool](https://github.com/URL) - Description`

Every pull request will be checked with `markdownlint` to ensure consistent writing and Markdown best practices. You can validate your files locally using the following Docker command:

```ps1
docker run -v $PWD:/workdir davidanson/markdownlint-cli2:v0.15.0 "**/*.md" --config .github/.markdownlint.json --fix
```

Every section should contains the following files, you can use the `_template_vuln` folder to create a new technique folder. 

**特别要求：** 所有新增或修改的内容必须包含中文注释，并符合“安全研究专家”的角色规范。

- **README.md**: 漏洞描述以及如何利用它，包括若干 Payload。
    - **必须在文件顶部包含业务溯源注释：**
      ```markdown
      <!--
       * [业务问题]: 描述该代码/Payload集解决的具体业务需求或功能点。
       * [实现逻辑]: 简述核心代码逻辑、调用链路或使用的关键技术栈。
      -->
      ```
- **Intruder**: 为 Burp Intruder 提供的一组文件
- **Images**: README.md 使用的图片
- **Files**: README.md 引用的文件

## README.md 格式规范

使用示例文件夹 [_template_vuln/](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/_template_vuln/) 创建新的漏洞文档。主页面是 [README.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/_template_vuln/README.md)。它按章节组织，包括漏洞标题、描述以及链接到文档主要部分的目录摘要。

- **工具 (Tools)**: 列出相关工具，并附上仓库链接和简要描述。
- **方法论 (Methodology)**: 提供所用方法的快速概述，并附上代码片段演示利用步骤。
- **实验室 (Labs)**: 引用可以练习类似漏洞的在线平台。
- **参考资料 (References)**: 列出外部资源，如博客文章或文章，提供与漏洞相关的附加背景或案例研究。
