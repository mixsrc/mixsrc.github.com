---
title: Java 中 File 的各种 getPath
categories: 开发
toc: false
permalink: /java-file-get-path.html
---

## getPath、getAbsolutePath、getAbsolutePath

先说 getPath 和 getAbsolutePath，当输入为绝对路径时，返回的都是绝对路径。当输入为相对路径时，getPath 返回的是 File 构造方法里的路径，getAbsolutePath 返回的其实是user.dir + getPath 的内容。

getCanonicalPath 返回的也是绝对路径，而且能够将符号链接、`..`和`.`这样的符号解析出来。

例如，MacOS 下

 | getPath                   | getAbsolutePath            | getCanonicalPath             |
 | ------------------------- | -------------------------- | ---------------------------- |
 | 1.txt                     | /tmp/dir1/dir2/1.txt       | /private/tmp/dir1/dir2/1.txt |
 | ../../1.txt               | /tmp/dir1/dir2/../../1.txt | /private/tmp/1.txt           |
 | /tmp/1.txt                | /tmp/1.txt                 | /private/tmp/1.txt           |
 | /tmp/dir1/dir2/.././1.txt | /tmp/dir1/dir2/.././1.txt  | /private/tmp/dir1/1.txt      |

## 鉴权问题

某个鉴权功能是基于文件路径和配置的字符串比较判断的，但直接使用但是 getAbsolutePath，但通过在路径中使用`..`和`.`这样的字符，就可以绕过鉴权机制，并获取到文件，所以，应该使用 getCanonicalPath 获取标准化后的路径。
