---
title: hexo博客搬迁小笔记
poc: true
date: 2023-03-17 22:17:27
categories:
tags: [悬而未决]
---


整体没有遇到太多问题, 记录几个不好搜索到的小技巧
https://i007it.com/2022/05/12/Hexo%E8%87%AA%E5%AE%9A%E4%B9%89%E9%A1%B5%E9%9D%A2%E8%B7%B3%E8%BF%87%E6%B8%B2%E6%9F%93/

## tag

```
tags:
  - 123
  - 456
```

or

```
tags: [123, 456]
```

## categories

```
categories: 123

categories: [123, 456]
```

```categories:
   - 123
   - 456
```

多标签写法，文章被分类到123、456以及123的自分类789这3个分类下面，官方指定写法。

```
categories:
   - [123]
   - [456]
   - [123, 789]
```

hexo博客支持latex需要修改渲染器设置, 跟随该文章修改 https://zhuanlan.zhihu.com/p/381508379

但是不知道为什么, 有一篇空文章被新渲染器报错了, 我博客里面空文章不少的, 唯独那篇报错了, 观察了以下没发现特别之处, 唯一特别的是他是草稿文章, 另外他的标题是中文

记得添加apt install到GitHub action 示例如下

```
    steps:
      - name: Install pandoc
        run: sudo apt-get update && sudo apt-get install -y pandoc
```
