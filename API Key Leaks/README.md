<!--
 * [业务问题]: API 密钥和令牌泄露是现代云原生应用中极其严重的安全隐患。攻击者利用泄露的密钥可直接获取云资源控制权、访问私有数据库或劫持第三方服务（如 Telegram, AWS, GCP），导致严重的财务损失和数据泄露。
 * [实现逻辑]: 本文档整理了 API 密钥泄露的常见原因（硬编码、Docker 镜像、公开仓库等），提供了主流扫描工具（TruffleHog, Trivy 等）的使用方法，并汇总了针对不同服务（AWS, Telegram 等）的密钥有效性验证（Keyhacks）实战技巧。
 -->

# API Key and Token Leaks (API 密钥与令牌泄露)

> API 密钥和令牌是用于管理公共和私有服务访问权限的常见身份验证形式。泄露这些敏感数据可能导致未经授权的访问、安全沦陷以及潜在的数据泄露风险。

## 概要 (Summary)

- [Tools](#tools)
- [Methodology](#exploit)
    - [Common Causes of Leaks](#common-causes-of-leaks)
    - [Validate The API Key](#validate-the-api-key)
- [References](#references)


## Tools

- [aquasecurity/trivy](https://github.com/aquasecurity/trivy) - General purpose vulnerability and misconfiguration scanner which also searches for API keys/secrets
- [blacklanternsecurity/badsecrets](https://github.com/blacklanternsecurity/badsecrets) - A library for detecting known or weak secrets on across many platforms
- [d0ge/sign-saboteur](https://github.com/d0ge/sign-saboteur) - SignSaboteur is a Burp Suite extension for editing, signing, verifying various signed web tokens
- [mazen160/secrets-patterns-db](https://github.com/mazen160/secrets-patterns-db) - Secrets Patterns DB: The largest open-source Database for detecting secrets, API keys, passwords, tokens, and more.
- [momenbasel/KeyFinder](https://github.com/momenbasel/KeyFinder) - is a tool that let you find keys while surfing the web
- [streaak/keyhacks](https://github.com/streaak/keyhacks) - is a repository which shows quick ways in which API keys leaked by a bug bounty program can be checked to see if they're valid
- [trufflesecurity/truffleHog](https://github.com/trufflesecurity/truffleHog) - Find credentials all over the place
- [projectdiscovery/nuclei-templates](https://github.com/projectdiscovery/nuclei-templates) - Use these templates to test an API token against many API service endpoints
    ```powershell
    nuclei -t token-spray/ -var token=token_list.txt
    ```


## 方法论 (Methodology)

* **API 密钥 (API Keys)**: 用于验证与您的项目或应用程序关联的请求的唯一标识符。
* **令牌 (Tokens)**: 授予对受保护资源访问权限的安全令牌（如 OAuth 令牌）。
     
### Common Causes of Leaks

* **Hardcoding in Source Code**: Developers may unintentionally leave API keys or tokens directly in the source code.

    ```py     
    # Example of hardcoded API key
    api_key = "1234567890abcdef"
    ```

* **Public Repositories**: Accidentally committing sensitive keys and tokens to publicly accessible version control systems like GitHub.

    ```ps1
    ## Scan a Github Organization
    docker run --rm -it -v "$PWD:/pwd" trufflesecurity/trufflehog:latest github --org=trufflesecurity
    
    ## Scan a GitHub Repository, its Issues and Pull Requests
    docker run --rm -it -v "$PWD:/pwd" trufflesecurity/trufflehog:latest github --repo https://github.com/trufflesecurity/test_keys --issue-comments --pr-comments
    ```

* **Hardcoding in Docker Images**: API keys and credentials might be hardcoded in Docker images hosted on DockerHub or private registries.

    ```ps1
    # Scan a Docker image for verified secrets
    docker run --rm -it -v "$PWD:/pwd" trufflesecurity/trufflehog:latest docker --image trufflesecurity/secrets
    ```

* **Logs and Debug Information**: Keys and tokens might be inadvertently logged or printed during debugging processes.

* **Configuration Files**: Including keys and tokens in publicly accessible configuration files (e.g., .env files, config.json, settings.py, or .aws/credentials.).


### 验证 API 密钥 (Validate The API Key)

If assistance is needed in identifying the service that generated the token, [mazen160/secrets-patterns-db](https://github.com/mazen160/secrets-patterns-db) can be consulted. It is the largest open-source database for detecting secrets, API keys, passwords, tokens, and more. This database contains regex patterns for various secrets.

```yaml
patterns:
  - pattern:
      name: AWS API Gateway
      regex: '[0-9a-z]+.execute-api.[0-9a-z._-]+.amazonaws.com'
      confidence: low
  - pattern:
      name: AWS API Key
      regex: AKIA[0-9A-Z]{16}
      confidence: high
```

Use [streaak/keyhacks](https://github.com/streaak/keyhacks) or read the documentation of the service to find a quick way to verify the validity of an API key.

* **Example**: Telegram Bot API Token

    ```ps1
    curl https://api.telegram.org/bot<TOKEN>/getMe
    ```


## References

* [Finding Hidden API Keys & How to Use Them - Sumit Jain - August 24, 2019](https://web.archive.org/web/20191012175520/https://medium.com/@sumitcfe/finding-hidden-api-keys-how-to-use-them-11b1e5d0f01d)
* [Introducing SignSaboteur: Forge Signed Web Tokens with Ease - Zakhar Fedotkin - May 22, 2024](https://portswigger.net/research/introducing-signsaboteur-forge-signed-web-tokens-with-ease)
* [Private API Key Leakage Due to Lack of Access Control - yox - August 8, 2018](https://hackerone.com/reports/376060)
* [Saying Goodbye to My Favorite 5 Minute P1 - Allyson O'Malley - January 6, 2020](https://www.allysonomalley.com/2020/01/06/saying-goodbye-to-my-favorite-5-minute-p1/)