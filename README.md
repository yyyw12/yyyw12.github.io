# yyyw12 的博客

基于 [Hugo](https://gohugo.io/) + [PaperMod](https://github.com/adityatelange/hugo-PaperMod) 搭建的个人技术博客，
用于记录《Linux 基础及应用》课程的学习笔记。

- 本地地址：<http://192.168.100.128:1313/>
- 公网地址：<https://yyyw12.github.io/>

## 功能

| 页面 | 说明 |
| --- | --- |
| 首页 | 个人简介（`[params.homeInfoParams]`）+ 按时间倒序的文章卡片 |
| 关于 | 姓名、专业、研究方向、学校与个人照片 |
| 标签 | 按标签聚合文章（taxonomies 自动生成） |
| 分类 | 按分类聚合文章 |
| 归档 | 按年份/月份的时间轴归档 |
| 搜索 | 前端全文搜索，索引来自 `/index.json` |

## 目录结构

```
blog/
├── archetypes/default.md
├── content/
│   ├── about.md            # 关于页
│   ├── archives.md         # 归档页（layout: archives）
│   ├── search.md           # 搜索页（layout: search）
│   └── posts/              # 文章（5 篇，均为 draft: false）
├── static/images/          # 头像等静态图片
├── themes/PaperMod/        # 主题（git submodule）
├── .github/workflows/hugo.yaml
├── hugo.toml
└── .gitignore              # 含 public/
```

## 本地运行

```bash
# 克隆时连同主题一起拉取
git clone --recurse-submodules https://github.com/yyyw12/yyyw12.github.io.git
cd yyyw12.github.io

# 启动本地服务，--bind 0.0.0.0 允许宿主机通过虚拟机 IP 访问
hugo server --bind 0.0.0.0 -D      # http://localhost:1313/

# 只构建静态文件
hugo --gc --minify                  # 输出到 public/
```

## 部署

推送到 `main` 分支后，GitHub Actions 自动完成构建与发布：

1. `actions/checkout`（`submodules: recursive`，否则拉不到主题）
2. `peaceiris/actions-hugo` 安装 Hugo v0.168.0（standard 版）
3. `hugo --gc --minify`
4. `peaceiris/actions-gh-pages` 把 `public/` 推送到 `gh-pages` 分支
5. 仓库 Settings → Pages 的发布源设为 `gh-pages` 分支

## 遇到的问题与解决办法

1. **`GLIBC_2.33 not found`**：CentOS 7 的 glibc 只有 2.17，官方 extended 版 Hugo 无法运行。
   改用静态链接的 standard 版 v0.168.0；PaperMod 只使用纯 CSS，不需要 extended。
2. **`GLIBCXX_3.4.29 not found`**：系统 libstdc++ 过旧，同样通过静态链接版本解决。
3. **连接 github.com 超时**：默认解析到的 IP 的 443 端口被阻断，在 `/etc/hosts` 中指向可用地址后正常。
4. **宿主机无法访问 1313 端口**：残留的 firewalld 规则在 INPUT 链末尾 `REJECT`，
   停止 firewalld 并清空 iptables 规则后恢复。
5. **`key socialIcons already exists as a table`**：`[params.socialIcons]` 与 `[[params.socialIcons]]` 混用，
   统一改为数组写法。
6. **Actions 构建找不到主题**：`actions/checkout` 未加 `submodules: recursive`。
7. **部署后页面没有样式**：`baseURL` 与实际访问地址不一致，改为 `https://yyyw12.github.io/` 后重新构建。

## 说明

站点中的姓名 `yyyw12` 与头像为占位内容，可按需替换：

- 个人信息：`content/about.md`、`hugo.toml` 中的 `[params.homeInfoParams]`
- 头像：`static/images/profile.jpg`
