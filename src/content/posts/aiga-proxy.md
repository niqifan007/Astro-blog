---
title: 利用deno代理cloudflare ai gateway
published: 2025-01-11
description: '使用deno反代ai gateway，解决地区限制'
image: 'https://i.111666.best/image/EM4wnNEAjRJcwBCoKdmbEV.jpg'
tags: [LLM, Deno, AI]
category: 'Guides'
draft: false 
lang: ''
---
## 利用deno代理cloudflare ai gateway
上一篇文章介绍了如何使用cloudflare ai gateway，但是由于cloudflare ai gateway会透传用户的ip，所以有些地区原本无法访问的ai api还是会无法访问，这时候我们可以使用deno来代理cloudflare ai gateway，这样就可以解决地区限制的问题。

### 代码
```typescript
const TARGET_HOST = "gateway.ai.cloudflare.com";
const BASE_PATH = "/v1/你自己的标识符/aigate/google-ai-studio";

Deno.serve(async (request) => {
  const url = new URL(request.url);
  const originalPath = url.pathname;
  
  url.host = TARGET_HOST;
  url.pathname = BASE_PATH + originalPath;
  url.protocol = "https:";

  const newRequest = new Request(url.toString(), {
    headers: request.headers,
    method: request.method,
    body: request.body,
    redirect: "follow",
  });

  const response = await fetch(newRequest);
  
  // 添加 CORS 头
  const headers = new Headers(response.headers);
  headers.set("Access-Control-Allow-Origin", "*");
  headers.set("Access-Control-Allow-Methods", "*");
  headers.set("Access-Control-Allow-Headers", "*");

  return new Response(response.body, {
    status: response.status,
    headers: headers,
  });
});
```
### 说明
这个是一个例子，代理了gateway的google-ai-studio，你可以根据自己的需求修改`BASE_PATH`，这样就可以代理其他的ai gateway了。使用方法和gateway一样，只是地址换成了你的deno服务的地址。