---
title: 使用Cloudflare AI Gateway监控、控制和优化 AI 应用
published: 2025-01-05
description: 'Cloudflare AI Gateway 为你的 AI 应用提供集中管理和控制。只需一行代码即可连接您的应用，监控使用情况、成本和错误。'
image: 'https://i.111666.best/image/JhGgrQ53hB0Bx0OcFK5Ssy.png'
tags: [cloudflare, ai]
category: 'Guides'
draft: false 
lang: ''
---
## 使用 Cloudflare AI Gateway 监控、控制和优化 AI 应用

### 为什么要用 Cloudflare AI Gateway？

众所周知，AI 应用开发在调用各种 AI 服务时，往往会面临一些挑战，如：不能有效监控和控制成本、难以管理多个供应商等。现在大多数的解决方案都是通过oneapi或者newapi来解决这些问题，但是这些基本上都要自己再去部署一套服务，有时候我们就是单纯的想去监控一下key的使用和日志情况，这时候使用Cloudflare AI Gateway就是一个不错的选择。

## 核心功能

### 1. 分析和监控

- 实时查看请求数量、token 使用情况和成本估算
- 详细的日志记录，支持故障排查和审计
- 自定义过滤器，按时间和提供商类型筛选数据

### 2. 性能优化

- 智能缓存机制，提升响应速度
- 请求重试和模型回退策略
- 支持 WebSocket API，实现持续通信

### 3. 成本控制

- 统一的成本监控仪表板
- 速率限制功能，防止过度使用
- 自定义缓存策略，降低 API 调用成本

### 4. 多供应商支持

AI Gateway 支持市面上主流的 AI 服务提供商，包括：

- [Amazon Bedrock](https://aws.amazon.com/bedrock/)
- [Anthropic](https://www.anthropic.com/)
- [Azure OpenAI](https://azure.microsoft.com/products/cognitive-services/openai-service/)
- [Cohere](https://cohere.com/)
- [DeepSeek AI](https://deepseek.com/)
- [Google AI Studio](https://makersuite.google.com/)
- [Google Vertex AI](https://cloud.google.com/vertex-ai)
- [Grok](https://grok.x.ai/)
- [Groq](https://groq.com/)
- [HuggingFace](https://huggingface.co/)
- [Mistral AI](https://mistral.ai/)
- [OpenAI](https://openai.com/)
- [OpenRouter](https://openrouter.ai/)
- [Perplexity](https://www.perplexity.ai/)
- [Replicate](https://replicate.com/)
- [Workers AI](https://workers.cloudflare.com/ai)
- Universal Endpoint

## 快速入门指南

### 1. 创建网关

1. 登录 Cloudflare 控制台
2. 导航至 AI > AI Gateway
3. 点击 "Create Gateway"
4. 设置网关名称（限制64字符）
![image](https://i.111666.best/image/spMZxwdf5tHGdVXPwP0WAP.png)

### 2. API 集成
其实这一步就是将Cloudflare AI Gateway的API集成到你的项目中，这样就可以实现对你的项目进行监控和控制了。AI Gateway 提供两种集成方式：CURL 和 JS。
就是把你的api的请求地址换成Cloudflare AI Gateway的api地址，token还是一样的不变，
![image](https://i.111666.best/image/quDQR12QAYmVkS5GZBDvTT.png)
![image](https://i.111666.best/image/8kVmmbwGwN00ISWeX701mj.png)

:::important
需要注意的是，这里cloudflare会透传你的ip，所以你的ip会暴露在请求头中直接发送给AI服务商，因此被限制的地区的话还是不能调用api的，不能作为代理使用。
:::

## 结论

Cloudflare AI Gateway 适合简单的监控和控制，对于一些大型的项目还是需要自己搭建一套服务来实现更加详细的追踪控制。

## 参考资源

- [Cloudflare AI Gateway 官方文档](https://developers.cloudflare.com/ai-gateway/)
- [快速入门指南](https://developers.cloudflare.com/ai-gateway/get-started/)
- [产品介绍页面](https://www.cloudflare.com/zh-cn/developer-platform/products/ai-gateway/)