# gitbook搭建

gitbook 是markdown 语法的一个不错的书籍工具，可以使用它来完成对markdown 文档的归档，将多个markdown 文档编排成一本有完整目录结构的电子书籍。

- 非常适合做接口文档的整理
- 同时提供在线浏览能力；
- 也可以结合 github page 来搭建自己的技术博客；

## **安装node.js**

去这个网址：[Previous Releases | Node.js](https://link.zhihu.com/?target=https%3A//nodejs.org/en/download/releases/)

> **node.js 使用的不是10.x版本，就会报错（尝试过12.x的也报错，所以这里推荐10.x）**
>
> 我这里安装的是 10.24.1 (x64)

检查版本

<img src="https://cdn.jsdelivr.net/gh/miiarms/typroa-image/moving/202302101144683.png" alt="image-20230210114439582" style="zoom:50%;" />

---

## **安装 gitbook**

命令行安装

```bash
npm install gitbook -g
gitbook -V
```

---

## gitbook常用命令

- 安装 GitBook：`npm i gitbook-cli -g`

- 初始化 GitBook 项目：`gitbook init`

- 安装 GitBook 依赖：`gitbook install`

- 开启 GitBook 服务：`gitbook serve`

- 打包 GitBook 项目：`gitbook build`

- GitBook 命令行查看：`gitbook -help`

- GitBook 版本查看：`gitbook -V`

---

## 初始化一本书籍

找个空文件夹，初始化一个 GitBook 项目：`gitbook init`，目录会生成一个 `README.md` 内容文件和一个 `SUMMARY.md` 目录文件。

<img src="https://cdn.jsdelivr.net/gh/miiarms/typroa-image/moving/202302101208849.png" alt="image-20230210120837782" style="zoom: 33%;float:left" />

- README.md

- SUMMARY.md

  >  表示目录，里面的格式是 Markdown 语法的链接格式，
  >
  >  即： `[链接](链接地址) `表示跳转链接，
  >
  >  如：`[introduction](readme.md)`

```
# Summary.md 例子

* [Introduction](README.md)
* [Easy](easy/README.md)
    * [1.Two Sum](easy/1.Two Sum.md)
    * [7.Reverse Integer](easy/7.Reverse Integer.md)
* [Medium](medium/README.md)
    * [2.Add Two Numbers](medium/2.Add Two Numbers.md)
    * [3.Longest Substring Without Repeating Characters](medium/3.Longest Substring Without Repeating Characters.md)
```

---

## 书籍预览

​	使用命令行执行

```shell
gitbook serve ./{book_name} 
# 若只执行gitbook build，会生成_book目录，但不能预览。
```

<img src="https://cdn.jsdelivr.net/gh/miiarms/typroa-image/moving/202302101228350.png" alt="image-20230210122855273" style="zoom:33%;" />

打开浏览器, 输入上面的`http://localhost:4000`即可预览书籍

<img src="https://cdn.jsdelivr.net/gh/miiarms/typroa-image/moving/202302101231741.png" alt="image-20230210123156653" style="zoom:33%;" />

这时候如果想将书籍提供给他人阅读，岂不是只需要将这个静态网站打包，再上传到服务器上即可？没错！就是这样。

---

## gitbook导出

gitbook创建的书籍还可以导出为电子书，比如pdf、epub和 mobi 格式。

```bash
gitbook pdf ./ ./mybook.pdf

gitbook epub ./ ./mybook.epub

gitbook mobi ./ ./mybook.mobi
```

其中，最后一个参数表示输出文件的文件名，可省略，默认输出为当前目录下的book文件。
再前面一个参数表示gitbook所在的目录。

> <font color="red">注：</font> 直接运行上述命令可能会报错，导出电子书之前，需先安装一款本地电子书管理工具：[Calibre](https://calibre-ebook.com/)
>
> **安装后记得将其安装根目录添加到环境变量PATH中**

然后就可以成功的导出为电子书了。

转成的pdf格式如下：

<img src="https://cdn.jsdelivr.net/gh/miiarms/typroa-image/moving/202302101244794.png" alt="导出pdf" style="zoom:50%;" />

可惜的是有个小问题，导出的pdf无论是左侧书签还是TOC目录，在跳转上存在着一些问题。但是导出的epub和mobi都非常棒

---

## gitbook 配置

所有的配置都以 JSON 格式存储在名为 `book.json` 的文件中。主要有以下字段：

| 字段          | 示例                                                         | 说明                                                         |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| gitbook       | { “gitbook”: “>=2.0.0” }                                     | 探测用来生成书本的GitBook的版本。格式是一个 [SEMVER](http://semver.org/) 条件。 |
| title         | {“title”: “Summary”}                                         | 书名，默认从README中提取                                     |
| authon        | {“author”: “miiarms”}                                        | 作者名                                                       |
| description   | { “description”: “This is my first book!” }                  | 定义了书本的描述，默认是从 **README**（第一段）中提取的。    |
| isbn          | { “isbn”: “978-7-115-32010-0” }                              | 定义了书本的ISBN                                             |
| language      | { “language”: “fr” }                                         | 定义了书本的语言，默认值是 `en`                              |
| direction     | { “direction”: “rtl” }                                       | 用来重新设置语言的文字方向的。rtl：从右至左。ltr：从左至右   |
| styles        | {“styles”: { “website”: “styles/website.css”, “pdf”: “styles/pdf.css” } } | 自定义书本的css                                              |
| plugins       | { “plugins”: [“myplugins”] }                                 | 插件列表                                                     |
| pluginsConfig | { “pluginsConfig”: { “myplugins”: { “message”: “Hello World” } } } | 插件配置                                                     |
| structure     | { “structure”: {“readme”: “INTRO.md” } }                     | 指定README，SUMMARY等文件的路径                              |
| variables     | { “variables”: { “myTest”: “Hello World” } }                 | 定义在 [模板](https://chrisniael.gitbooks.io/gitbook-documentation/content/format/templating.html) 中使用的变量值 |
| links         | “links” : { “sidebar” : { “Home” : “https://miiarms.github.io” } } | 在左侧导航栏添加链接信息                                     |

可供参考的 book.json:

```json
{
  "title": "java技术的文档库",
  "author": "miiarms ",
  "language": "zh-hans",
  "description":"java技术相关，netty、 elasticsearch、jvm、mySQL等",
  "styles": {
    "website": "./public-repertory/css/gitbook-configure.css"
  },
  "plugins": [
    "theme-comscore",
    "prism",
    "-highlight",
    "copy-code-button",
    "search-pro",
    "-search",
    "-lunr",
	"chapter-fold",
    "expandable-chapters",
    "splitter",
    "-sharing",
    "github-buttons",
    "donate",
    "baidu-tongji",
    "anchor-navigation-ex",
	"hide-element",
	"page-footer-ex",
	"mermaid-gb3",
	"favicon-absolute"
  ],
  "pluginsConfig": {
    "github-buttons": {
      "buttons": [
        {
          "user": "miiarms",
          "repo": "https://github.com/miiarms/miiarms.github.io", 
          "type": "star",
          "count": true,
          "size": "small"
        }, 
        {
          "user": "miiarms",
		  "repo": "https://github.com/miiarms/miiarms.github.io", 
          "width": "160", 
          "type": "follow", 
          "count": true,
          "size": "small"
        }
      ]
    },
    "donate": {
      "button": "打赏",
      "alipayText": "支付宝打赏",
      "wechatText": "微信打赏",
      "alipay": "/images/alipay.jpg",
      "wechat": "/images/wechatpay.png"
    },
    "prism": {
      "css": [
        "prismjs/themes/prism-solarizedlight.css"
      ],
      "lang": {
        "shell": "bash"
      }
    },
	"page-footer-ex": {
            "copyright": "[miiarms](https://github.com/miiarms/miiarms.github.io )",
            "markdown": true,
            "update_label": "<i>updated</i>",
            "update_format": "YYYY-MM-DD HH:mm:ss"
    },
    "baidu-tongji": {
      "token": "55e7dfe47f4dc1c018d4042fdfa62565"
    },
    "anchor-navigation-ex": {
      "showLevel": false
    },
	"hide-element": {
       "elements": [".gitbook-link"]
    },
	"favicon-absolute":{
		"favicon": "/images/icon/favicon.ico",
		"appleTouchIconPrecomposed152": "/images/icon/big_facion.png"
	}
  }
}

```

---

## gitbook 样式

有的时候，GitBook 会自带一些你不需要的样式，例如侧边栏的 **由 GitBook 提供支持** 等，我们可以通过设置 CSS 来让它隐藏掉：

```css
.gitbook-link {
    display: none
}
.sumary .divider{
    display: none
}
```

当然，也可以通过插件的方式来隐藏，比如 `hide-element` 插件

---

## gitbook插件

插件是扩展 GitBook 功能（电子书和网站）最好的方式。

**查找插件**

在nmp官网（https://www.npmjs.com/ ）上搜索关键词`gitbook-plugin`或`gitbook`即可

---

### **添加插件**

添加至 `book.json` 文件后，再执行命令`gitbook install`将插件下载至本地即可

```json
{
	"plugins": ["myPlugin", "anotherPlugin"]
}
```

---

### **删除插件**

如果不想使用自带的插件，在插件名称前面加 `-`：

```
{
	"plugins":[ "-search"]
}
```

如果不是自带的，将其从插件列表中去掉即可。

---

### **插件推荐**

- **折叠目录插件** [Expandable-chapters-small](https://github.com/chrisjake/gitbook-plugin-expandable-chapters-small)

```json
{    
    "plugins": ["expandable-chapters-small"] 
}
```



- **提供非官方的github按钮**（star, fork, sponsor, and follow ）[github-buttons](https://github.com/azu/gitbook-plugin-github-buttons)

```json
{
  "plugins": [
    "github-buttons"
  ],
  "pluginsConfig": {
    "github-buttons": {
      "buttons": [{
        "user": "azu",
        "repo": "JavaScript-Plugin-Architecture",
        "type": "star",
        "size": "large"
      }, {
        "user": "azu",
        "type": "follow",
        "width": "230",
        "count": false
      }]
    }
  }
}
```

| Option  | Description                                                  | 备注               |
| :------ | :----------------------------------------------------------- | :----------------- |
| `user`  | GitHub username that owns the repo/Username to sponsor       | 必须，用户名       |
| `repo`  | GitHub repository to pull the forks and watchers counts      | 必须，仓库名       |
| `type`  | Type of button to show: `watch`, `fork`, `sponsor`, or `follow` | 必须，4种类型之一  |
| `count` | Show the optional watchers or forks count: *none* by default or `true` | 可选，是否显示计数 |
| `size`  | Optional flag for using a larger button: *none* by default or `large` | 可选，按钮大小     |



- **添加好看的样式** theme-comscore

  为 GitBook 添加好看的样式，它会使 Table 表单等变得更加好看。

  ```json
  "plugins": [
    "theme-comscore"
  ]
  ```



- #### prism

  ​	为 GitBook 的 Code 添加更好看的样式，使用它的时候记得屏蔽 GitBook 默认的 `highlight` 插件，即通过 （`-highlight` 表示，下面出现 `-` 的插件也一样）

  ```json
  "plugins": [
    "prism",
    "-highlight"
  ],
  "pluginsConfig": {
    "prism": {
      "css": [
        "prismjs/themes/prism-solarizedlight.css"
      ],
      "lang": {
        "shell": "bash"
      }
    }
  }
  ```



- #### copy-code-button

  给 GitBook 的 Code 添加复制功能，可以一键复制代码块的所有代码。

  ```json
  "plugins": [
    "copy-code-button"
  ]
  ```



- #### search-pro

  由于 GitBook 支持的搜索，对于中文不太好。添加该插件后，对搜索结果能用高亮来显示，非常强大。当然，由于取缔了默认的搜索功能，所以需要屏蔽 `search` 和 `lunr`

  ```json
  "plugins": [
    "search-pro",
    "-search",
    "-lunr"
  ]
  ```



- #### expandable-chapters

  由于侧边栏标题是默认全部展开的，所以这个插件会让它具有展开收缩功能。

  ```json
  "plugins": [
    "expandable-chapters"
  ]
  ```



- #### splitter

  让侧边栏的宽度可以自行拖动。

  ```json
  "plugins": [
    "splitter"
  ]
  ```



- #### sharing

  `-sharing`：去掉 GitBook 默认的分享功能。由于它默认的一些推特，脸书都需要翻墙，而我们做的是中文站点，所以将分享功能全部关闭掉。

  ```json
  "plugins": [
    "-sharing"
  ]
  ```



- #### donate

  给底部配置一个打赏模块，用户可以点击进行支付宝、微信打赏等

  ```json
  "plugins": [
    "donate"
  ],
  "pluginsConfig": {
    "donate": {
      "button": "打赏",
      "alipayText": "支付宝打赏",
      "wechatText": "微信打赏",
      "alipay": "/images/alipay.jpg",
      "wechat": "/images/wechatpay.png"
    }
  }
  ```

- #### tbfed-pagefooter

  给 GitBook 每个页面添加页脚，这样就可以知道这些文件的 `copyright` 以及修改时间等。

  ```json
  "plugins": [
    "tbfed-pagefooter"
  ],
  "pluginsConfig": {
    "tbfed-pagefooter": {
      "copyright":"Copyright &copy miiarms.com 2023",
      "modify_label": "该文件修订时间：",
      "modify_format": "YYYY-MM-DD HH:mm:ss"
    }
  }
  ```

- #### baidu-tongji

  给 GitBook 的站点添加百度统计，这样用户的访问数量可以通过百度统计查看到

  ```json
  "plugins": [
    "baidu-tongji"
  ],
  "pluginsConfig": {
    "baidu-tongji": {
      "token": "55e7dfe47f4dc1c12345678fdfa62565"
    }
  }
  ```

- **锚点导航**  [anchor-navigation-ex](https://github.com/zq99299/gitbook-plugin-anchor-navigation-ex)

  ```json
  {
    "plugins": [
         "anchor-navigation-ex"
    ]
  }
  ```

- **hide-element**

  隐藏某些 gitbook 自带的元素，例如侧边栏的 "由 GitBook 提供支持"

  ```json
  "plugins": [
    "hide-element"
  ],
  "pluginsConfig": {
   "hide-element": {
         "elements": [".gitbook-link"]
      }
  }
  ```

---

## github page

> [Github Pages](http://pages.github.com/) 是面向用户、组织和项目开放的公共静态页面搭建托管服务，站点可以被免费托管在 Github 上，你可以选择使用 
>
> Github Pages 默认提供的域名 <font color="blue">github.io</font> 或者自定义域名来发布站点。
>
> 每个github账号有且只有一个主页站点(<username>.github.io)，无限多的项目站点(<username>.github.io/repo)。

[GitHub Actions](https://github.com/features/actions) 是 GitHub 的持续集成服务，于2018年10月推出。

> 持续集成由很多操作组成，比如抓取代码、运行测试、登录远程服务器，发布到第三方服务等等。GitHub 把这些操作就称为 actions。
>
> 很多操作在不同项目里面是类似的，完全可以共享。GitHub 注意到了这一点，想出了一个很妙的点子，允许开发者把每个操作写成独立的脚本文件，存放到代码仓库，使得其他开发者可以引用。
>
> 如果你需要某个 action，不必自己写复杂的脚本，直接引用他人写好的 action 即可，整个持续集成过程，就变成了一个 actions 的组合。这就是 GitHub Actions 最特别的地方。

<font color="blue">这里我们使用 github page + github action  实现提交代码自动构建发布</font>



### 一、创建仓库

创建 github page 的仓库，github page的仓库命名格式是 `<username>.github.io `，如 miiarms.github.io

![image-20230210162450684](https://cdn.jsdelivr.net/gh/miiarms/typroa-image/moving/202302101624815.png)

---

### 二、上传书籍到github

到 gitbook 书籍的目录下，新建 `.gitignore` 文件，排除不需要上传的文件

```
_book/
node_modules/
```

<img src="https://cdn.jsdelivr.net/gh/miiarms/typroa-image/moving/202302101634038.png" alt="image-20230210163458965" style="zoom:33%;" />

将书籍上传到github

```bas
cd gitbook-test
git init
git add .		#添加目录下的所有文件
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/miiarms/gitbookTest.git  #注意修改用户名
git push -u origin main
```

---

### 三、设置github page

进入仓库，`Settings -> Options -> Github Pages`选择好分支与目录，点击Save。

<font color="blue">我这里的 github page 分支 是 pages </font> <br>

<font color="red">书籍源码是上传到 main 分支， pages 分支不需要提交，是由后面的 github action自动提交的 </font>

<img src="https://cdn.jsdelivr.net/gh/miiarms/typroa-image/moving/202302101648582.png" alt="image-20230210164806487" style="zoom:33%;" />

---

### 四、配置 github action 

1. **新建Person Access Token** 在个人设置里生成一个新的个人令牌，权限仅选择`repo`即可。进入仓库，点击`Settiings -> Secrets -> New repository`，将其命名为`TOKEN`。

<img src="https://cdn.jsdelivr.net/gh/miiarms/typroa-image/moving/202302101700571.png" alt="test" style="zoom: 50%;" />

2. 在电脑本地书籍的目录下，新建目录`.github/workflows`，在该目录下新建文件 `generate-page.yml`

<img src="https://cdn.jsdelivr.net/gh/miiarms/typroa-image/moving/202302101653867.png" alt="image-20230210165333747" style="zoom:33%;" />

<img src="https://cdn.jsdelivr.net/gh/miiarms/typroa-image/moving/202302101653573.png" alt="image-20230210165352469" style="zoom:33%;" />

 `generate-page.yml` 文件内容如下，<font color="blue">内容看注释修改</font>

```yaml
# git提交时自动生产gitbook的静态文件到github pages
name: auto-generate-gitbook
#在main分支上进行push时触发 
on:                                  
  push:
    branches:
    - main
jobs:
  main-to-pages:
    runs-on: ubuntu-latest     
    steps:                          
    - name: checkout main
      uses: actions/checkout@v2
      with:
        ref: main
        
    - name: install nodejs
      uses: actions/setup-node@v1
      
    - name: configue gitbook
      run: |
        npm install -g gitbook-cli          
        gitbook install
                
    - name: generate _book folder
      run: |
        gitbook build
                
    - name: push _book to branch pages 
      env:
      	# 在个人设置里生成一个新的个人令牌，权限仅选择repo即可。进入仓库，点击Settiings -> Secrets -> New repository，将其命名为TOKEN
        TOKEN: ${{ secrets.TOKEN }} 
        REF: github.com/${{github.repository}}
        # ！！记得修改为自己github设置的邮箱
        MYEMAIL: xxx@gmail.com                  
        MYNAME: ${{github.repository_owner}}          
      run: |
        cd _book
        rm -rf .gitignore
        rm -rf .github/*
        git config --global user.email "${MYEMAIL}"
        git config --global user.name "${MYNAME}"
        git init
        git remote add origin https://${REF}
        git add . 
        git commit -m "Updated By Github Actions With Build ${{github.run_number}} of ${{github.workflow}} For Github Pages"
        git branch -M main
        git push --force --quiet "https://${TOKEN}@${REF}" main:pages
```

上面的最后一句，就是把生成的 `_book` 文件夹提交到 `pages` 分支 上

<font color="blue"> 最后，重新上传自己的书籍的改动到 github， github就会自动生成自己的github page 页面 </font>

```bas
git add .
git commit -m "github action yml"
git push
```

<font color="blue">稍等几分钟，访问 `https://<github用户名>.github.io` ，就能看到自己的书籍页面了</font>

---

## gitbook 避坑指北

 to be updated......