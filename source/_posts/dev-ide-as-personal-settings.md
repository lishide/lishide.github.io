---
title: 个人 IDEA（Android Studio）生产工具个性化配置
date: 2018-09-05 11:51:29
tags: [IDE, Android Studio]
---
记录个人 IDEA（Android Studio）生产工具个性化配置，常用插件等。
<!--more-->
# 个性化配置
## 基础配置
字体、字号、行号、导包等相关的配置。

## 代码配色主题下载
### 下载安装
在 [Color Themes](http://color-themes.com) 有很多的代码配色主题，应该能找到自己喜欢的。

步骤：
1. 下载主题—xxx.jar（Ps：如果我们下载下来的 jar 名字如果有空格，一定要把空格去掉，同时文件路径中不要含有中文）；
2. 选择 File—>Import Settings—>选择 jar 包导入；
3. 重启 IDEA（Android Studio）；
4. 修改个别不喜欢的地方（比如字体、某种类型字体颜色）

希望大家都找到自己喜欢的配色方案。其实，自带的 Darcula 主题就很棒了，官方出品，适配很好。

### Ladies Night 2 个人修改
试用了很多款主题，目前在用的是：**Ladies Night 2**，此主题在 Color Themes 上下载量最多。当然，每一款主题都并非完美的，并不一定适合所有人的口味，自己喜欢才好。针对几处不完美的地方（某些关键词色值或背景色值不合适导致的关键词看不清楚）做了一些修改，以备后用。

#### 字体字号
Color Scheme Font，选择合适的字体字号。

#### Language Defaults
- Inline Parameter hints
	- Default： **F：7A7A7A -> 787878；B：EDEDED -> 3B3B3B**
	- Current： **F：5B5B5B -> ACACAC；B：BCDAF7 -> 305D78**
	- Highlighted： **F：5B5B5B -> ACACAC；B：CCCCCC -> 4D4D4D**

- Braces and Operators
	- Semicolon&Comma： **E8E2B7 -> CC7832**

#### Java
- Comments
	- JavaDoc
		- Tag： **E0E2E4 -> 629755**
    	- Tag value： **[ ? ] -> 8A653B**
    	- Text： **7D8C93 -> 629755**

- Parameters
	- Type parameter： **E0E2E4 -> 507874**

#### Kotlin
- Smart-casts
	- Smart-cast implicit receiver(B)： **[ ? ] -> 223C23**
	- Smart-cast value(B)： **[ ? ] -> 223C23**
	- Smart constant(B)： **[ ? ] -> 223C23**

- KDoc
	- KDoc comment： **[ ? ] -> 629755**
	- tag： **E0E2E4 -> 629755**
	- KDoc Link in KDoc tag： **3D3D3D -> 8A653B**

- Properties and Variables
	- Instance property： **FEFFFB -> 9876AA**

#### Groovy
- Keyword： **000043 -> CC7832**

#### XML
- Namespace prefix： **FEFFFB -> 9876AA**

#### VCS
- VCS Annotations
	- Foreground： **000080 -> 8B999F**

#### General
- Errors and Warnings
	- Warning
        B：[ ] -> 52503A
        Error：E3BF20 -> BE9117
        Effects （取消勾选）

## Logcat 配色
Android Logcat 默认主题的配色只有红白两种颜色，不太便于我们区分 Log 的类型。
建议采用鲜明的配色，按照下面的色值修改配置。

| Log 级别 | 颜色   |
|-------- |--------|
| Assert  | AA66CC |
| Debug   | 33B5E5 |
| Error   | FF4444 |
| Info    | 99CC00 |
| Verbose | FFFFFF |
| Warning | FFBB33 |

## AS 插件

1. **GsonFormat**：快速将 json 字符串转换成一个 Java Bean。
2. **Android ButterKnife Zelezny**：自动生成 butterknife 注解。
3. **ADB WIFI**：使用 wifi 无线调试你的 app，无需 root 权限。
4. **Android WiFi ADB**：无线调试应用
5. **WakaTime**：记录你在 IDE 上的工作时间（推荐）。
6. **String Manipulation**：一个为文本操作提供操作的插件。
7. **postfix**：此插件可以快速进行 Log、Toast、isEmpty 的代码书写。
8. **Alibaba Java Coding Guidelines**：《阿里巴巴Java开发规约》的扫描插件。
9. **IconViewer**：图标预览
10. **TinyPic**：压缩图片资源
11. **Statistic**：统计代码量
12. **.ignore**：过滤 git 忽略文件
13. **Translation**：翻译插件
14. **Exynap**：一个帮助开发者自动生成样板代码的 AndroidStudio 插件。
15. **Lombok**：在项目中使用 Lombok 可以减少很多重复代码的书写。比如说 getter/setter/toString 等方法的编写。



