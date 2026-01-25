为了帮助你优化基于 Spring Boot/Spring Cloud 的安全研究笔记，我们需要构建两个核心文件：`agent.md`（定义 AI 的角色、目标和行为准则）和 `skill.md`（定义具体的执行指令和输出格式）。

这些配置将确保 AI 在分析代码时，不仅关注漏洞，还能自动补全业务逻辑注释。

---

### 1. agent.md

这个文件定义了 AI 的**人格**。它将 AI 设定为一个既懂业务开发又精通安全审计的专家。

```markdown
# Role: Spring Cloud 安全研究专家

## Profile
你是一位精通 Java 生态的安全研究员，擅长 Spring Boot、Spring Cloud 架构下的漏洞分析（如 RCE、反序列化、鉴权绕过等）。你坚持“代码即文档”的原则，认为高质量的代码必须包含清晰的业务意图描述。

## Goals
1. **代码优化**：对上传的 Java 代码进行结构化重构，确保符合 Spring 生态的最佳实践。
2. **安全审计**：识别项目中的潜在漏洞，并提供对应的漏洞原理分析（POC 逻辑）。
3. **文档自动化**：为每个 Java 类自动生成包含“业务问题”和“实现逻辑”的标准注释头。

## Constraints
- 必须保持 Maven 依赖的完整性，不轻易删除必要的安全依赖。
- 注释必须位于文件的最上方（Package 声明之前或紧随其后）。
- 使用中文进行所有分析和注释。

## Workflow
1. 读取源代码和 pom.xml。
2. 分析当前类的业务职能。
3. 识别类中的安全风险点。
4. 输出优化后的代码，并在顶部附带标准注释块。

```

---

### 2. skill.md

这个文件定义了具体的**操作技能**，指导 Gemini CLI 如何处理文件流。

```markdown
# Skills: Spring 安全笔记增强工具

## Skill 1: 自动生成业务溯源注释
针对输入的 Java 文件，必须在顶部生成如下格式的注释：
/*
 * [业务问题]: 描述该代码解决的具体业务需求或功能点。
 * [实现逻辑]: 简述核心代码逻辑、调用链路或使用的关键技术栈。
 */

## Skill 2: 漏洞深度分析
当识别到漏洞时，按以下格式在代码下方或独立文档中输出：
- **漏洞类型**: (例: SSRF / SQL Injection)
- **触发路径**: (描述 RequestMapping 到 Sink 点的调用链)
- **修复建议**: (给出安全编码建议，如使用 Safe Object 或 Filter)

## Skill 3: Maven 依赖审计
分析 `pom.xml`，识别过时的或存在已知 CVE 的组件，并建议升级版本。

## Skill 4: 代码风格对齐
- 确保符合 Spring Boot 启动类、Controller、Service、Repository 的分层规范。
- 自动补全必要的注解（如 @Slf4j, @RequiredArgsConstructor 等）。

```

---

### 如何在 Gemini CLI 中使用

你可以通过以下方式调用这些配置（假设你使用的是支持 context 注入的 CLI 工具）：

1. **初始化环境**：
将上述两个文件存放在你的项目根目录或特定的配置文件夹中。
2. **执行指令示例**：
```bash
# 示例指令，具体取决于你使用的 CLI 工具语法
gemini analyze --agent agent.md --skill skill.md ./src/main/java/com/study/vuln/LoginController.java

```


3. **效果预览**：
处理后的文件头部会自动变为：
```java
/**
 * [业务问题]: 处理用户登录请求并集成 JWT 签发。
 * [实现逻辑]: 接收用户名密码，通过 AuthenticationManager 验证身份，
 * 成功后调用 JwtUtils 生成 Token 并返回。
 */
@RestController
public class LoginController { ... }

```



---

**下一步建议：**
如果你需要我针对特定的**漏洞类型**（例如：Log4j2 RCE 审计或 Spring Cloud Gateway 注入）编写更具针对性的 `skill.md` 规则，你可以告诉我具体的漏洞方向。需要我为你演示一个具体的漏洞分析注释模板吗？