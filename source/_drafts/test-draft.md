---
title: Hexo草稿功能
tags:
  - Blog
date: 2019-05-08 15:15:01
---


当有一些未编辑好不想发布的文章，可以使用草稿功能。
<!--more-->

```bash
# 新建草稿
hexo new draft <title>

# 草稿预览
hexo server —drafts

# 草稿发布
hexo publish [layout] <title>

# 移动到草稿
# 将_posts下的文章和相应的文件夹移动到_drafts即可
mv _posts/<title>* _drafts 
```

