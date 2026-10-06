# 个人主页工程说明

基于 [Quarto](https://quarto.org/) 搭建的静态个人主页，通过 GitHub Pages 发布。

- **线上地址**：https://zhiweizhou-think.github.io
- **技术栈**：Quarto（Pandoc）+ Bootstrap 5 + SCSS，无服务器、无数据库

## 1 目录与文件职责

### 1.1 文件引用关系

`_quarto.yml` 是配置枢纽，页面之间几乎不直接引用，而是靠_quarto配置，在渲染时统一"接线"：

```
                    ┌───────────────────────┐
                    │     _quarto.yml        │  配置枢纽
                    └───────────┬───────────┘
        ┌──────────────┬───────┼────────────────┬──────────────┐
        │ render 通配  │ navbar│                │ theme        │ favicon
        │ *.qmd/*.md   │ href  │                │              │
        ▼              ▼       ▼                ▼              ▼
  ┌───────────┐  ┌──────────┐ ┌──────────┐ ┌────────────┐ ┌──────────────────┐
  │ 所有 qmd/ │  │index.qmd │ │blog.qmd  │ │styles.scss │ │images/           │
  │ md 被收录  │  │ (Home)   │ │ (Blogs)  │ │  全站样式   │ │ site-logo.png    │
  └───────────┘  └────┬─────┘ └────┬─────┘ └──────┬─────┘ └──────────────────┘
                      │            │               │ @import
                      │ image      │ 外链(CSDN)    ▼
                      ▼            │          ┌─────────────────┐
              ┌──────────────────┐│          │ Google Fonts    │
              │images/           ││          │ (外部字体服务)   │
              │zhiweizhou2024.jpg││          └─────────────────┘
              └──────────────────┘│
                      │ 外链       │
                      ▼            ▼
        mailto 邮箱 / GitHub / CSDN（全部是站外链接）
```

### 1.2 源码区（日常维护）


| 文件          | 作用                                                   |
| ------------- | ------------------------------------------------------ |
| `_quarto.yml` | 全站配置：输出目录、导航栏、搜索、页脚、主题、右侧目录 |
| `index.qmd`   | 首页，`about: trestles` 头像简介卡片布局               |
| `blog.qmd`    | 博客页（当前为 CSDN 文章外链列表）                     |
| `about.qmd`   | 关于我页                                               |
| `styles.scss` | 自定义样式：配色变量、字体、行高、链接/标题/代码样式   |
| `images/`     | 图片素材（头像、logo）                                 |

### 1.3 产物区（自动生成）


| 路径                                   | 作用                                        |
| -------------------------------------- | ------------------------------------------- |
| `docs/*.html`                          | 渲染后的网页                                |
| `docs/site_libs/`                      | 前端依赖：Bootstrap、图标、搜索、代码高亮等 |
| `docs/search.json`、`docs/sitemap.xml` | 搜索索引、站点地图                          |

### 1.4 历史遗留（当前方案不使用）

- `.Rprofile`、`.hugo_build.lock`：早期 blogdown + Hugo 方案的残留
- `_extensions/coatless/webr/`：未使用的 WebR 扩展

## 2 _quarto.yml 文件结构

全站配置中心，顶层分为 `project`、`website`、`format` 三个块，分别管"怎么构建""网站长什么样""输出格式"。

### 2.1 project 块：构建行为

```yaml
project:
  type: website              # 工程类型：多页网站
  output-dir: docs           # 产物输出目录（对应 GitHub Pages 发布目录）
  render:                    # 参与渲染的文件白名单
    - "*.qmd"                #   所有 qmd
    - "*.md"                 #   所有 md
    - "!readme.md"           #   ! 表示排除：readme.md 仅作仓库文档，不发布
```

### 2.2 website 块：站点信息与公共元素

```yaml
website:
  page-navigation: true                # 页面底部上一页/下一页导航
  title: "Zhiwei Zhou"                 # 站点名称（导航栏/标题）
  site-url: https://zhiweizhou-think.github.io   # 正式域名（用于 sitemap）
  favicon: images/site-logo.png        # 浏览器标签页图标
  repo-url: https://github.com/...     # 仓库地址
  repo-actions: [edit, issue]          # 每页显示"编辑此页/提 issue"按钮

  page-footer:                         # 页脚（| 表示多行文本，可含 HTML）
    left: |
      © Zhiwei Zhou<br>
    right: |
      Made with [Quarto](https://quarto.org/)<br>

  navbar:                              # 导航栏
    background: "#A9CCE3"              #   背景色（十六进制颜色）
    search: true                       #   开启站内搜索（自动生成 search.json）
    right:                             #   右侧菜单项列表
      - text: "Home"
        href: index.qmd
      - text: "Blogs"
        href: blog.qmd
```

### 2.3 format 块：输出格式与页面功能

```yaml
format:
  html:                       # 输出 HTML
    theme: styles.scss        # 套用自定义样式文件
    toc: true                 # 开启右侧目录
    toc-position: right       # 目录位置
    toc-depth: 3              # 目录收录到几级标题（## 是 2 级，### 是 3 级）
```

### 2.4  高频修改场景


| 想做的事                   | 改哪里                                    |
| -------------------------- | ----------------------------------------- |
| 新增导航菜单项（如 About） | `navbar.right` 下加 `- text / href`       |
| 改网站名称、域名、图标     | `website` 的 `title / site-url / favicon` |
| 有文件不想发布             | `project.render` 加 `- "!文件名"`         |
| 改页脚版权文字             | `page-footer.left / right`                |
| 关闭右侧目录               | `toc: false`                              |
| 换全站样式                 | `format.html.theme` 指向新 scss           |

## 3. 日常更新流程

```bash
# 1. 修改根目录的 .qmd / .md / styles.scss / images
# 2. 本地预览（保存自动热更新）
quarto preview

# 3. 手动渲染（未使用 preview 时）
quarto render

# 4. 提交推送，几十秒后线上自动更新
git add .
git commit -m "更新内容"
git push
```

## 4.`.qmd` 文件结构

```markdown
---
title: "页面标题"          # YAML 头：页面级配置
---

::: {#some-id}            # Quarto 扩展块（渲染为 <div>）
## 标题                    # 以下为普通 Markdown 语法
- 列表项
:::
```

除 YAML 头和 `:::` 布局块外，其余与普通 Markdown 完全一致。

## 5. index.qmd 语法结构详解

首页源文件由 **YAML 配置头 + 标准 Markdown 正文 + Quarto 布局块** 三种语法组成。

### 5.1 YAML 配置头（`---` 之间）

以 `key: value` 形式书写，嵌套用空格缩进，列表用 `-` 开头（**缩进只能用空格，不能用 Tab**）：

```yaml
---
title: "Zhiwei Zhou"                    # 页面标题（浏览器标签页）
image: images/zhiweizhou2024.jpg        # 头像路径
about:                                  # 启用 about 简介卡片
  template: trestles                    # 布局模板：左图右文
  image-width: 12em                     # 头像宽度
  image-shape: round                    # 头像圆形裁剪
  links:                                # 图标按钮列表
    - icon: mailbox                     #   图标名（Bootstrap Icons）
      text: 1258806185@qq.com
      href: mailto:1258806185@qq.com    #   点击发邮件
---
```

### 5.2 标准 Markdown 正文

```markdown
## 教育与工作经历          ← 二级标题（# 数量代表层级）

- 2023.07 - 至今：联影...   ← 无序列表
- 2017.09 - 2023.06：华科...

我是一名物理算法工程师...     ← 普通段落，直接写文字
```

其他常用语法：链接 `[文字](网址)`、加粗 `**文字**`、图片 `![说明](路径)`。

### 5.3 Quarto 扩展块（Fenced Div）

```markdown
::: {#hero-heading}        ← 块开始，渲染为 <div id="hero-heading">
正文内容...
:::                        ← 块结束（必须成对出现）
```

`{#id}` 指定容器 id，`{.class}` 指定样式类，普通 Markdown 编辑器不识别此语法。

### 5.4  常用取值含义


| 写法                                  | 含义                                          |
| ------------------------------------- | --------------------------------------------- |
| `template: trestles`                  | Quarto 内置 about 布局模板                    |
| `image-shape: round`                  | 头像形状（round 圆形 / rounded 圆角矩形）     |
| `icon: mailbox` / `icon: brands csdn` | Bootstrap Icons 图标名，`brands` 为品牌图标集 |
| `12em`                                | CSS 长度单位                                  |
| `mailto:邮箱`                         | 邮件链接，点击唤起邮件客户端                  |

### 5.5 渲染对应关系

```
YAML 头      → 头像尺寸/形状、图标按钮、页面标题
Markdown 正文 → <h2> / <ul><li> / <p> 标准 HTML
::: 布局块    → <div id="hero-heading">
全部产物      → 套用 styles.scss + Bootstrap 生成最终卡片排版
```

## 6 blog.qmd 与 styles.scss 语法

### 6.1 blog.qmd：YAML 头 + Markdown 链接列表

```markdown
---
title: "Blogs"            # 页面标题
editor:                   # 编辑器行为配置（仅影响写作体验，不影响网页）
  markdown:
    wrap: 72              # 保存时自动按 72 字符折行
---

&emsp;                    # HTML 实体：一个全角空格，用于顶部留白

## 2026

  - zhiweizhou, 2026.02, [cmake基本使用](https://blog.csdn.net/...)
```

涉及的语法：


| 写法                            | 含义                                                  |
| ------------------------------- | ----------------------------------------------------- |
| `[文字](网址)`                  | Markdown 行内链接，渲染为`<a href="网址">文字</a>`    |
| `&emsp;`                        | HTML 实体字符，Pandoc 允许在 Markdown 中直接使用 HTML |
| 列表中`文字, [链接](...), 文字` | 列表项里可混排普通文字与多个链接                      |
| 年份`## 2026`                   | 用二级标题分组，配合右侧 toc 自动生成年份导航         |

新增博客条目时，照抄一行 `- 作者, 日期, [标题](链接)` 即可。

### 6.2 styles.scss：SCSS（CSS 超集）

Quarto 要求样式文件用两个特殊注释分段：

```scss
/*-- scss:defaults --*/    // 第一段：定义/覆盖 Quarto 主题变量
/*-- scss:rules --*/       // 第二段：写自定义 CSS 规则
```

**defaults 段**用到的语法：

```scss
$theme-darkcoffee1: #570e06;          // SCSS 变量：$变量名: 值;
$body-bg: $theme-white;               // 变量可引用其他变量
$navbar-bg: $theme-darkgreen1;        // Quarto 内置变量，赋值即覆盖默认主题

@import url('https://fonts.googleapis.com/css2?family=Poppins...');  // 导入外部字体
$font-family-sans-serif: 'Lato', sans-serif;                     // 覆盖全局字体
```

**rules 段**就是标准 CSS：

```scss
body {
  line-height: 1.7;        /* 行高 */
  color: #000000;          /* 正文黑色 */
}

a {                        /* 选择器：选中所有超链接 */
  color: #0066cc;
  text-decoration: none;   /* 去掉下划线 */
}

h1 { font-size: 40px; }    /* 一级标题字号 */
```

要点：

- `$` 开头的是 SCSS 变量，只在 defaults 段定义/覆盖
- 改颜色、字体优先改 defaults 段的变量，一处生效全站
- rules 段写普通 CSS 选择器即可（`body`、`h1`、`a`、`.类名`、`#id`）
- Quarto 构建时把 SCSS 编译成 CSS 注入每个页面，无需手动引入

### 6.3 两者与页面的关系

```
blog.qmd ──渲染──▶ blog.html（内容：链接列表）
styles.scss ──编译──▶ CSS（外观：颜色/字体/间距）──套用到──▶ 所有页面
```

内容和样式完全分离：改 blog.qmd 只影响博客页文字，改 styles.scss 会影响全站外观。

### 方案修订

如果只挑最重要的 5 个：

1. Markdown — 写内容的语言
2. YAML — 写配置的语言（[YAML 入门教程 | 菜鸟教程](https://www.runoob.com/w3cnote/yaml-intro.html)）
3. CSS/SCSS 基础 — 改样式的语言（选择器、盒模型、颜色）（[CSS 语法 | 菜鸟教程](https://www.runoob.com/css/css-syntax.html)）
4. Quarto 配置 —`_quarto.yml` + about 模板 + listing
5. Git + GitHub Pages — 发布上线

进阶知识点（想玩花样时学）

- Bootstrap 栅格 — 自定义复杂布局
- CSS Grid/Flexbox — 精细控制对齐
- JavaScript — 加交互（暗黑模式、动画）
- Quarto 扩展 — 装第三方模板/过滤器
- SEO —`description` 、`sitemap` 、`robots`

一句话总结 你项目的知识栈 = Markdown（内容）+ YAML（配置）+ CSS/SCSS（样式）+ Quarto（构建）+ Git/GitHub Pages（部署） 。前三个是"写"，后两个是"发"。核
## 
 doT
. 更新基于stm32的稳定光源1

o
待
心就这 5 块，其他都是锦上添花。

## 经典demo

[Deepanshu Mahto](https://mahtodeepanshu.github.io/)

[Sam Shanny-Csik](https://samanthacsik.github.io/)

[Chi Zhang – Hello, I’m Chi](https://chizapoth.github.io/)

[Valentina Giunchiglia](https://valegiunchiglia.github.io/personal_website/)

[Gang He – Dr. Gang He](https://drganghe.github.io/)
