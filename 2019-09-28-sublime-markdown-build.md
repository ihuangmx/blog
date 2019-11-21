# Sublime 快速修正 Markdown 文档

使用Sublime Text 编辑器进行创作的时候，经常需要校验 Markdown 是否满足规范，比如 [中文文案排版指北](https://github.com/mzlogin/chinese-copywriting-guidelines/blob/Simplified/README.md)，利用 Sublime Text 自带的 Build 功能可快速实现。

首先安装 [hustcc/lint-md](https://github.com/hustcc/lint-md) 

```bash
$ npm i -g lint-md
$ npm i -g lint-md-cli
```

该工具可自动校验 `markdown` 文档，用法很简单

```bash
$ lint-md README.md Document.md
```

在 Sublime 的用户目录下新建 `Markdown.sublime-build` 文件

```json
{	
	"cmd": [
		"sh", "-c", "lint-md $file --fix $file"
    ],
    "selector" : "text.html.markdown"
}
```

在编辑 `markdown` 文件时，只需要使用快捷键   `command` + `b` 即可自动校验 `markdown` 文档。

该工具能够满足大多数需求，部分需求可自行添加，比如对引号的替换

```json
{	
	"cmd": [
		"sh", "-c", "lint-md $file --fix && sed -i  's/”/」/g' $file && sed -i  's/“/「/g' $file"
    ],
    "selector" : "text.html.markdown"
}
```


