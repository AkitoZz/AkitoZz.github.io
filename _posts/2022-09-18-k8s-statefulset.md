---
layout: post
title: k8s控制器之有状态服务
categories: [k8s, statefulset]
description: k8s, statefulset
keywords: k8s, 控制器, statefulset
---

---
    k8s常见控制器类别有deployment、daemonset、statefulset，deployment认为所有pod互不影响，deployment直接管理的是replicaset，因为他认为所有pod都是一样的，而statefulset管理的则是pod，每个pod都有对应的细微处理
---