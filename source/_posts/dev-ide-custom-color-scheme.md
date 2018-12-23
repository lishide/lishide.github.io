---
title: 自定义 IDE 配色主题
date: 2018-10-30 16:08:47
tags: [IDE, Color Scheme]
---
试用了很多款主题，个人比较喜欢 **Ladies Night 2** 和 **Sublime Text 2**。 Ladies Night 2 主题在 Color Themes 上下载量最多，Sublime Text 2 也有很多人推荐。目前在用的是：**Sublime Text 2**，当然，每一款主题都并非完美的，并不一定适合所有人的口味，自己喜欢才好。针对几处不完美的地方（某些关键词色值或背景色值不合适导致的关键词看不清楚）做了一些修改，以备后用。结合 **Darcula** 主题比较有用的提示功能，配置最适合自己的、赏心悦目的 Color Scheme。

<!--more-->

### Ladies Night 2 个人修改
#### 字体字号
Color Scheme Font，选择合适的字体字号。

#### General
- Errors and Warnings
	- Warning
        B：**[ ] -> 52503A**
        Error：**E3BF20 -> BE9117**
        Effects：**（取消勾选）**

#### Language Defaults
- Inline Parameter hints
	- Default
		F：**7A7A7A -> 787878**
        B：**EDEDED -> 3B3B3B**
	- Current：
		 F：**5B5B5B -> ACACAC**
         B：**BCDAF7 -> 305D78**
	- Highlighted
		 F：**5B5B5B -> ACACAC**
         B：**CCCCCC -> 4D4D4D**

- Braces and Operators
	- Semicolon&Comma
		F：**E8E2B7 -> CC7832**

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
	- Smart-cast implicit receiver(B)： **DBFFDB -> 223C23**
	- Smart-cast value(B)： **DBFFDB -> 223C23**
	- Smart constant(B)： **DBFFDB -> 223C23**

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
	- F： **000080 -> 8B999F**

### Sublime Text 2 个人修改
#### General
- Text
	- Default text
		F：**CFBFAD -> A9B7C6**
		B：**272822 -> 2B2B2B**
	- Folded text
		F：**404040 -> 8C8C8C**
        B：**CC9900 -> 3A3A3A**
	- Deleted text
		F：**272822 -> BBBBBB**
        B：**CFBFAD -> 450505**
		Effects：**[ ] -> C3C3C3**
	- Hyperlinks
		- Reference
			F：**0000FF -> 589DF6**
			Effects：**0000FF -> 589DF6 （Underscored）**
		- Followed
			Effects：**同 F （Underscored）**
		- Unfollowed
			Effects：**同 F （Underscored）**

- Errors and Warnings
	- Deprecated symbol
		Foreground **（关）**
		Effects：**[ ] -> C3C3C3**
	- Error（这个较有用，提示错误）
		E：**BC3F3C -> 9E2927**
		Effects：**3C3F3C -> BC3F3C （Underwaved）**
	- Unsed symbol
		Effects：**3C3F3C -> 808080 （Underwaved）**
	- Typo
		Effects：**3C3F3C -> 659C6B （Underwaved）**
	- Problem from server
		E：**F49810 -> B06100**
		Effects：**3C3F3C -> F49810 （Underscored）**
	- Weak warning
		E：**AEAE80 -> 756D56**
		Effects：**[ ] -> AEAE80 （Underwaved）**

- Search Results
	- Search Results
		- Search result
			B：**000000 -> 155221**
			E：**000000 -> 00530D**
		- Search result(write access)
			B：**000000 -> 532B2E**
            E：**000000 -> 8D4457**
		- Search result(write access)
			B：**D8D8D8 -> 32593D**
            E：**D8D8D8 -> 61936F**
            Effects：**[ ] -> 3C704B**
- Code
	- Identifier under caret
        B：**000000 -> 344134**
        E：**000000 -> 036B13**
	- Identifier under caret
        B：**000000 -> 40332B**
        E：**000000 -> B56277**
- Editor
	- Gutter background（编码区左侧信息区域背景）
		B：**272822 -> 313335**
	- Caret row（光标所在行）
		B：**5B5A4E -> 323232(383341)**
	- Selection background（选中行背景）
		B：**CC9900 -> 214283**
	- Selection foreground（选中行前景）
		F：**关闭，即显示原代码颜色**
    - Notification background（通知背景）
		F：**FFFFCC -> 5C5C42**

#### Language Defaults
- Inline Parameter hints
	- Default
		F：**7A7A7A -> 787878**
        B：**EDEDED -> 3B3B3B**
	- Current：
		 F：**5B5B5B -> ACACAC**
         B：**BCDAF7 -> 305D78**
	- Highlighted
		 F：**5B5B5B -> ACACAC**
         B：**CCCCCC -> 4D4D4D**

- Comments
    - Doc Comment
        - Tag： **8A826B -> 629755**
        - Tag value： **3D3D3D -> 8A653B**
- String
	- Escape Sequence
		- Invalid
			F：**FF0000 -> ECE47E**
            Effects：**[ ] -> FF0000 （Underwaved）**
- Bad character
	F：**（关）**
    Effects：**[ ] -> FF0000 （Underwaved）**
- Template language
	F：**（关）**
    B：**[ ] -> 232525**
- Number
	F：**C48CFF -> 6897BB**

#### Java
- Class Fields
	- Constant(static final field)： **660E7A -> 9876AA**
	- Instance field
		F：**CFBFAD -> 9876AA**
- Parameters
	- Implicit anonymous class parameter
		Effects：**[ ] -> 52E3F6 （Underscored）**

#### Kotlin
- Smart-casts
	- Smart-cast implicit receiver(B)： **DBFFDB -> 223C23**
	- Smart-cast value(B)： **DBFFDB -> 223C23**
	- Smart constant(B)： **DBFFDB -> 223C23**
- Annotation： **000080 -> CC7832**
- Parameters
	- Type parameter： **BFA4A4 -> 20999D**
- Properties and Variables
	- Instance property： **FEFFFB -> 9876AA**
	- Extension property： **CFBFAD -> 9876AA(Bold,Italic)**
	- Package-level property： **CFBFAD -> 9876AA(Bold,Italic)**
	- Synthetic extension property： **CFBFAD -> 9876AA**
	- Var(Effects)： **000000 -> BCA5C4**

#### Groovy
- Keyword： **000043 -> CC7832**
- Bad character
	F：**（关）**
    Effects：**[ ] -> FF0000 （Underwaved）**
- References
	- Unresolved reference
		F：**000080 -> 808080**

#### XML
- Namespace prefix： **CFBFAD -> 9876AA**
- Attribute name： **BFA4A4 -> BABABA**
- Attribute value： **ECE47E -> 4CD656**


#### VCS
- VCS Annotations
	- F： **000080 -> 8B999F**

> 备份很重要！！！记得把自己配置好的 Color Scheme 导出 jar，云盘备份保存。

