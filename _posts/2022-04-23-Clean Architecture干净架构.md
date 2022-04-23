---
layout: post
title: 'Clean Architecture干净架构'
date: 2022-04-23
author: smakry
tags: Clear Architecture 干净架构
---

> 外部依赖内部，内部不能依赖外部；外层中命名和数据格式不能影响内圈

```sh
--------------------------------------------------
|｜ Devices、Web、Ui、DB 、External Interfaces	||
|｜                   ⬇️                       	||
|｜     GateWays、Presenters、Controllers			||
|｜                   ⬇️                       	||
|｜                User Cases					||
|｜                   ⬇️                       	||
|｜                Entities                	 	||
--------------------------------------------------
```
