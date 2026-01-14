---
title: '用 n8n 打造微信公众号自动发布神器'
cover: /Users/luckyme/Documents/quickstart/assets/pics/a1.png
---

+++
date = '2026-01-14T13:34:03+08:00'
draft = true
title = '用n8n打造微信公众号自动发布神器'
+++

### 介绍n8n
理论上可以采集任何支持rss订阅的文章,ai会根据指定的提示词进行数据处理以及润色并生成公众号文章草稿以及发布.

### 技术栈
- n8n
- rsshub
- gemini2.5 pro
- n8n WeChat Official Account 

### 效果演示
演示视频: /Users/luckyme/Documents/quickstart/assets/video/1月14日.mp4

### 原理以及步骤
1. 设置定时触发器
2. 设置采集数据源(任何支持rss订阅的数据源都可)
3. 采集数据处理
4. ai对数据进行再处理生成需要的格式
5. 对输出的数据进行格式转换支持公众号格式
6. 保存为草稿并可自动发布

### 关注公众号,私信回复“用 n8n 打造微信公众号自动发布神器”领取n8n工作流源文件~