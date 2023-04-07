---
title: stable-diffusion 部署踩坑
poc: true
categories: 未分类
tags: 未分类
date: 2023-03-30 18:48:38
---

问题一 RuntimeError: Couldn't install open_clip.

解决 https://github.com/CompVis/stable-diffusion/issues/661

问题二 No matching distribution found for gradio==3.23

https://github.com/AUTOMATIC1111/stable-diffusion-webui/issues/8896



问题三 RuntimeError: self.size(-1) must be divisible by 4 to view Byte as Float (different element sizes), but got 16268507RuntimeError: self.size(-1) must be divisible by 4 to view Byte as Float (different element sizes), but got 16268507RuntimeError: self.size(-1) must be divisible by 4 to view Byte as Float (different element sizes), but got 16268507

https://github.com/AUTOMATIC1111/stable-diffusion-webui/issues/8653