# 我的Hugo博客配置

这是我的个人博客配置仓库，使用Hugo和PaperMod主题构建。

## 项目地址
```
https://github.com/chenyongda2018/my_hugo_blog_config
```

## 克隆项目（包含子模块）
由于项目包含子模块（themes/PaperMod和public），请使用以下命令克隆项目以确保子模块也被正确拉取：

```bash
git clone --recursive https://github.com/chenyongda2018/my_hugo_blog_config.git
```

## 如果已经克隆了项目但忘记拉取子模块
如果已经克隆了仓库但子模块内容未同步，可以运行以下命令：

```bash
git submodule update --init --recursive
```

## 子模块说明
本项目包含以下子模块：
- `themes/PaperMod`: Hugo博客主题
- `public`: 博客生成的静态文件