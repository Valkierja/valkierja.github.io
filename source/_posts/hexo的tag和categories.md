---
title: hexo的tag和categories
poc: true
date: 2023-03-17 22:17:27
categories:
tags:
---

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
