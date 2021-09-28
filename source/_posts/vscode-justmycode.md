---
title: "VSCode 中启用内部库代码调试"
date: 2021-09-26T08:48:37+08:00
description: ""
tags: ["VSCode", "调试", "库代码"]
categories: ["VSCode"]
---

但我们使用 VSCode 按 F5 调试代码时, 默认的设置只会加载我们自己的代码, 跳过加载库中符号. 如果需要步进到库中的代码, 我们需要修改 `lanuch.json`, 在 `configurations` 第一项中添加 `"justMyCode": false`.
<!-- more -->
