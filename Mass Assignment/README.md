<!--
 * [业务问题]: 批量赋值（Mass Assignment）漏洞允许攻击者通过添加未授权的参数（如 isAdmin, role）来修改对象属性，导致权限提升、数据篡改或访问控制绕过。常见于使用 ORM 框架（Rails, Django, Laravel）的 Web 应用。
 * [实现逻辑]: 本文档通过实际案例说明了批量赋值漏洞的原理，展示了攻击者如何通过在请求中添加 isAdmin 等参数来获取管理员权限，并提供了 PentesterAcademy 实验室链接和防护建议。
 -->

# Mass Assignment (批量赋值)

> 批量赋值攻击是一种安全漏洞，当 Web 应用程序自动将用户提供的输入值分配给程序对象的属性或变量时就会发生。如果用户能够修改他们不应该访问的属性（如用户权限或管理员标志），这就会成为一个问题。

## 概要 (Summary)

* [Methodology](#methodology)
* [Labs](#labs)
* [References](#references)


## Methodology

Mass assignment vulnerabilities are most common in web applications that use Object-Relational Mapping (ORM) techniques or functions to map user input to object properties, where properties can be updated all at once instead of individually. Many popular web development frameworks such as Ruby on Rails, Django, and Laravel (PHP) offer this functionality.

For instance, consider a web application that uses an ORM and has a user object with the attributes `username`, `email`, `password`, and `isAdmin`. In a normal scenario, a user might be able to update their own username, email, and password through a form, which the server then assigns to the user object.

However, an attacker may attempt to add an `isAdmin` parameter to the incoming data like so:

```json
{
    "username": "attacker",
    "email": "attacker@email.com",
    "password": "unsafe_password",
    "isAdmin": true
}
```

If the web application is not checking which parameters are allowed to be updated in this way, it might set the `isAdmin` attribute based on the user-supplied input, giving the attacker admin privileges


## Labs

* [PentesterAcademy - Mass Assignment I](https://attackdefense.pentesteracademy.com/challengedetailsnoauth?cid=1964)
* [PentesterAcademy - Mass Assignment II](https://attackdefense.pentesteracademy.com/challengedetailsnoauth?cid=1922)


## References

- [Hunting for Mass Assignment - Shivam Bathla - August 12, 2021](https://blog.pentesteracademy.com/hunting-for-mass-assignment-56ed73095eda)
- [Mass Assignment Cheat Sheet - OWASP - March 15, 2021](https://cheatsheetseries.owasp.org/cheatsheets/Mass_Assignment_Cheat_Sheet.html)
- [What is Mass Assignment? Attacks and Security Tips - Yoan MONTOYA - June 15, 2023](https://www.vaadata.com/blog/what-is-mass-assignment-attacks-and-security-tips/)