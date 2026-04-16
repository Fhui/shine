# Shine

> `ShineF` 的静态博客发布仓库，部署域名为 [`shine666.club`](http://shine666.club)。

![site](https://img.shields.io/badge/site-shine666.club-blue)
![stack](https://img.shields.io/badge/stack-Hexo%20%2B%20NexT-black)
![content](https://img.shields.io/badge/posts-13-brightgreen)

## 项目简介

这个仓库保存的是 **ShineF 个人技术博客的静态生成结果**，可以直接用于 GitHub Pages 一类的静态托管。

从页面源码可以看出，本项目基于：

- `Hexo`
- `NexT` 主题
- 自定义域名 `shine666.club`

博客描述为：

> 口中有德, 目中有人, 心中有爱, 行中带善

## 内容方向

当前收录的文章主要集中在以下主题：

- Android 开发
- Gradle 配置
- RxJava 入门
- MVP 设计模式
- BLE 4.0
- EventBus 使用经验
- Hexo + GitHub 博客搭建

其中包含的代表性文章有：

- `Volley以及自定义Request详解`
- `探寻Android中MVP设计模式`
- `Gradle多环境配置`
- `RXJava入门`
- `Mac上使用Hexo+github搭建自己的博客`

## 目录结构

```text
.
├── index.html          # 博客首页
├── 2016/               # 2016 年文章页面
├── 2017/               # 2017 年文章页面
├── archives/           # 归档页
├── tags/               # 标签页
├── css/                # 编译后的样式文件
├── js/                 # 前端脚本
├── lib/                # 第三方静态资源库
├── images/             # 主题图片与站点资源
├── uploads/            # 上传附件
├── search.xml          # 站内搜索索引
└── CNAME               # 自定义域名配置
```

## 本地预览

由于仓库中保存的是静态文件，直接启动一个静态服务器即可预览：

```bash
python3 -m http.server 4000
```

然后访问 [http://localhost:4000](http://localhost:4000)。

也可以使用：

```bash
npx serve .
```

## 部署方式

这个仓库采用的是典型的静态站点发布方式：

- 生成后的页面文件直接提交到仓库根目录
- `CNAME` 用于绑定自定义域名 `shine666.club`
- 历史提交信息多为 `Site updated: ...`，说明它更像是博客构建产物仓库，而不是 Hexo 源码仓库

## 现状说明

- 当前仓库中 **不包含完整的 Hexo 源码**
- 常见的 Hexo 源码目录和配置，如 `_config.yml`、`source/`、`themes/`，在这里并不存在
- 如果后续要继续写文章、换主题或重新生成页面，更适合在原始 Hexo 源码仓库中操作

## License

当前仓库里没有明确的许可证文件；如果你希望明确文章、图片和代码片段的复用范围，建议补充 `LICENSE`。
