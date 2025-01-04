---
title: 使用Deno构建多种大模型AI API代理(2025.1.4更新)
published: 2025-01-04
description: '使用deno将多种AI API的代理整合到一个服务中(openai, gemini等)'
image: 'https://i.111666.best/image/g7lNeIW4q2FnEnvKbY0XRL.jpg'
tags: [LLM, Deno, AI]
category: 'Guides'
draft: false 
lang: ''
---
## 使用 Deno 搭建大模型 API 反向代理服务

### 背景介绍
随着各大 AI 公司相继推出自己的语言模型 API 服务,开发者在使用这些服务时经常会遇到网络访问的问题。本文将介绍如何使用 Deno 搭建一个反向代理服务器,用于代理访问各种 AI 模型的 API 接口。

### 为什么使用 Deno
众所周知cloudflare workers可以作为一个非常好的反向代理服务,但是它的免费版有很多限制,比如每天的请求次数有限制,还有会暴露你的ip地址等。
Deno是一个安全的运行时环境,它的安全性和性能都非常好,而且它的部署也非常简单。

### 代码实现
```typescript
import { serve } from "https://deno.land/std/http/server.ts";
import { Buffer } from "node:buffer";

const apiMapping = {
  '/discord': 'https://discord.com/api',
  '/telegram': 'https://api.telegram.org',
  '/openai': 'https://api.openai.com',
  '/claude': 'https://api.anthropic.com',
  '/meta': 'https://www.meta.ai/api',
  '/groq': 'https://api.groq.com/openai',
  '/xai': 'https://api.x.ai',
  '/cohere': 'https://api.cohere.ai',
  '/huggingface': 'https://api-inference.huggingface.co',
  '/together': 'https://api.together.xyz',
  '/novita': 'https://api.novita.ai',
  '/portkey': 'https://api.portkey.ai',
  '/fireworks': 'https://api.fireworks.ai',
  '/openrouter': 'https://openrouter.ai/api',
  '/nvidia': 'https://integrate.api.nvidia.com',
  '/cerebras': 'https://api.cerebras.ai',
  '/sambanova': 'https://api.sambanova.ai',
  '/gemini': 'https://generativelanguage.googleapis.com'
};

class HttpError extends Error {
  constructor(message, status) {
    super(message);
    this.name = this.constructor.name;
    this.status = status;
  }
}

const fixCors = ({ headers, status, statusText }) => {
  headers = new Headers(headers);
  headers.set("Access-Control-Allow-Origin", "*");
  return { headers, status, statusText };
};

const handleOPTIONS = async () => {
  return new Response(null, {
    headers: {
      "Access-Control-Allow-Origin": "*",
      "Access-Control-Allow-Methods": "*",
      "Access-Control-Allow-Headers": "*",
    }
  });
};

const BASE_URL = "https://generativelanguage.googleapis.com";
const API_VERSION = "v1beta";
const API_CLIENT = "genai-js/0.21.0";

const makeHeaders = (apiKey, more) => ({
  "x-goog-api-client": API_CLIENT,
  ...(apiKey && { "x-goog-api-key": apiKey }),
  ...more
});

async function handleModels(apiKey) {
  const response = await fetch(`${BASE_URL}/${API_VERSION}/models`, {
    headers: makeHeaders(apiKey),
  });
  let { body } = response;
  if (response.ok) {
    const { models } = JSON.parse(await response.text());
    body = JSON.stringify({
      object: "list",
      data: models.map(({ name }) => ({
        id: name.replace("models/", ""),
        object: "model",
        created: 0,
        owned_by: "",
      })),
    }, null, "  ");
  }
  return new Response(body, fixCors(response));
}

const DEFAULT_EMBEDDINGS_MODEL = "text-embedding-004";
async function handleEmbeddings(req, apiKey) {
  if (typeof req.model !== "string") {
    throw new HttpError("model is not specified", 400);
  }
  if (!Array.isArray(req.input)) {
    req.input = [req.input];
  }
  let model;
  if (req.model.startsWith("models/")) {
    model = req.model;
  } else {
    req.model = DEFAULT_EMBEDDINGS_MODEL;
    model = "models/" + req.model;
  }
  const response = await fetch(`${BASE_URL}/${API_VERSION}/${model}:batchEmbedContents`, {
    method: "POST",
    headers: makeHeaders(apiKey, { "Content-Type": "application/json" }),
    body: JSON.stringify({
      "requests": req.input.map(text => ({
        model,
        content: { parts: { text } },
        outputDimensionality: req.dimensions,
      }))
    })
  });
  let { body } = response;
  if (response.ok) {
    const { embeddings } = JSON.parse(await response.text());
    body = JSON.stringify({
      object: "list",
      data: embeddings.map(({ values }, index) => ({
        object: "embedding",
        index,
        embedding: values,
      })),
      model: req.model,
    }, null, "  ");
  }
  return new Response(body, fixCors(response));
}

const DEFAULT_MODEL = "gemini-1.5-pro-latest";
async function handleCompletions(req, apiKey) {
  let model = DEFAULT_MODEL;
  switch (true) {
    case typeof req.model !== "string":
      break;
    case req.model.startsWith("models/"):
      model = req.model.substring(7);
      break;
    case req.model.startsWith("gemini-"):
    case req.model.startsWith("learnlm-"):
      model = req.model;
  }
  const TASK = req.stream ? "streamGenerateContent" : "generateContent";
  let url = `${BASE_URL}/${API_VERSION}/models/${model}:${TASK}`;
  if (req.stream) { url += "?alt=sse"; }
  const response = await fetch(url, {
    method: "POST",
    headers: makeHeaders(apiKey, { "Content-Type": "application/json" }),
    body: JSON.stringify(await transformRequest(req)),
  });

  let body = response.body;
  if (response.ok) {
    let id = generateChatcmplId();
    if (req.stream) {
      body = response.body
        .pipeThrough(new TextDecoderStream())
        .pipeThrough(new TransformStream({
          transform: parseStream,
          flush: parseStreamFlush,
          buffer: "",
        }))
        .pipeThrough(new TransformStream({
          transform: toOpenAiStream,
          flush: toOpenAiStreamFlush,
          streamIncludeUsage: req.stream_options?.include_usage,
          model, id, last: [],
        }))
        .pipeThrough(new TextEncoderStream());
    } else {
      body = await response.text();
      body = processCompletionsResponse(JSON.parse(body), model, id);
    }
  }
  return new Response(body, fixCors(response));
}

const harmCategory = [
  "HARM_CATEGORY_HATE_SPEECH",
  "HARM_CATEGORY_SEXUALLY_EXPLICIT",
  "HARM_CATEGORY_DANGEROUS_CONTENT",
  "HARM_CATEGORY_HARASSMENT",
  "HARM_CATEGORY_CIVIC_INTEGRITY",
];

const safetySettings = harmCategory.map(category => ({
  category,
  threshold: "BLOCK_NONE",
}));

const fieldsMap = {
  stop: "stopSequences",
  n: "candidateCount",
  max_tokens: "maxOutputTokens",
  max_completion_tokens: "maxOutputTokens",
  temperature: "temperature",
  top_p: "topP",
  top_k: "topK",
  frequency_penalty: "frequencyPenalty",
  presence_penalty: "presencePenalty",
};

const transformConfig = (req) => {
  let cfg = {};
  for (let key in req) {
    const matchedKey = fieldsMap[key];
    if (matchedKey) {
      cfg[matchedKey] = req[key];
    }
  }
  if (req.response_format) {
    switch (req.response_format.type) {
      case "json_schema":
        cfg.responseSchema = req.response_format.json_schema?.schema;
        if (cfg.responseSchema && "enum" in cfg.responseSchema) {
          cfg.responseMimeType = "text/x.enum";
          break;
        }
      case "json_object":
        cfg.responseMimeType = "application/json";
        break;
      case "text":
        cfg.responseMimeType = "text/plain";
        break;
      default:
        throw new HttpError("Unsupported response_format.type", 400);
    }
  }
  return cfg;
};

const parseImg = async (url) => {
  let mimeType, data;
  if (url.startsWith("http://") || url.startsWith("https://")) {
    try {
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error(`${response.status} ${response.statusText} (${url})`);
      }
      mimeType = response.headers.get("content-type");
      data = Buffer.from(await response.arrayBuffer()).toString("base64");
    } catch (err) {
      throw new Error("Error fetching image: " + err.toString());
    }
  } else {
    const match = url.match(/^data:(?<mimeType>.*?)(;base64)?,(?<data>.*)$/);
    if (!match) {
      throw new Error("Invalid image data: " + url);
    }
    ({ mimeType, data } = match.groups);
  }
  return {
    inlineData: {
      mimeType,
      data,
    },
  };
};

const transformMsg = async ({ role, content }) => {
  const parts = [];
  if (!Array.isArray(content)) {
    parts.push({ text: content });
    return { role, parts };
  }
  for (const item of content) {
    switch (item.type) {
      case "text":
        parts.push({ text: item.text });
        break;
      case "image_url":
        parts.push(await parseImg(item.image_url.url));
        break;
      case "input_audio":
        parts.push({
          inlineData: {
            mimeType: "audio/" + item.input_audio.format,
            data: item.input_audio.data,
          }
        });
        break;
      default:
        throw new TypeError(`Unknown "content" item type: "${item.type}"`);
    }
  }
  return { role, parts };
};

const transformMessages = async (messages) => {
  if (!messages) { return; }
  const contents = [];
  let system_instruction;
  for (const item of messages) {
    if (item.role === "system") {
      delete item.role;
      system_instruction = await transformMsg(item);
    } else {
      item.role = item.role === "assistant" ? "model" : "user";
      contents.push(await transformMsg(item));
    }
  }
  if (system_instruction && contents.length === 0) {
    contents.push({ role: "model", parts: { text: " " } });
  }
  return { system_instruction, contents };
};

const transformRequest = async (req) => ({
  ...await transformMessages(req.messages),
  safetySettings,
  generationConfig: transformConfig(req),
});

const generateChatcmplId = () => {
  const characters = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
  const randomChar = () => characters[Math.floor(Math.random() * characters.length)];
  return "chatcmpl-" + Array.from({ length: 29 }, randomChar).join("");
};

const reasonsMap = {
  "STOP": "stop",
  "MAX_TOKENS": "length",
  "SAFETY": "content_filter",
  "RECITATION": "content_filter",
};
const SEP = "\n\n|>";
const transformCandidates = (key, cand) => ({
  index: cand.index || 0,
  [key]: {
  role: "assistant",
  content: cand.content?.parts.map(p => p.text).join(SEP) },
  logprobs: null,
  finish_reason: reasonsMap[cand.finishReason] || cand.finishReason,
});

const transformCandidatesMessage = transformCandidates.bind(null, "message");
const transformCandidatesDelta = transformCandidates.bind(null, "delta");

const transformUsage = (data) => ({
  completion_tokens: data.candidatesTokenCount,
  prompt_tokens: data.promptTokenCount,
  total_tokens: data.totalTokenCount
});

const processCompletionsResponse = (data, model, id) => {
  return JSON.stringify({
    id,
    choices: data.candidates.map(transformCandidatesMessage),
    created: Math.floor(Date.now() / 1000),
    model,
    object: "chat.completion",
    usage: transformUsage(data.usageMetadata),
  });
};

const responseLineRE = /^data: (.*)(?:\n\n|\r\r|\r\n\r\n)/;
async function parseStream(chunk, controller) {
  chunk = await chunk;
  if (!chunk) { return; }
  this.buffer += chunk;
  do {
    const match = this.buffer.match(responseLineRE);
    if (!match) { break; }
    controller.enqueue(match[1]);
    this.buffer = this.buffer.substring(match[0].length);
  } while (true);
}

async function parseStreamFlush(controller) {
  if (this.buffer) {
    console.error("Invalid data:", this.buffer);
    controller.enqueue(this.buffer);
  }
}

function transformResponseStream(data, stop, first) {
  const item = transformCandidatesDelta(data.candidates[0]);
  if (stop) { item.delta = {}; } else { item.finish_reason = null; }
  if (first) { item.delta.content = ""; } else { delete item.delta.role; }
  const output = {
    id: this.id,
    choices: [item],
    created: Math.floor(Date.now() / 1000),
    model: this.model,
    object: "chat.completion.chunk",
  };
  if (data.usageMetadata && this.streamIncludeUsage) {
    output.usage = stop ? transformUsage(data.usageMetadata) : null;
  }
  return "data: " + JSON.stringify(output) + delimiter;
}

const delimiter = "\n\n";

async function toOpenAiStream(chunk, controller) {
  const transform = transformResponseStream.bind(this);
  const line = await chunk;
  if (!line) { return; }
  let data;
  try {
    data = JSON.parse(line);
  } catch (err) {
    console.error(line);
    console.error(err);
    const length = this.last.length || 1;
    const candidates = Array.from({ length }, (_, index) => ({
      finishReason: "error",
      content: { parts: [{ text: err }] },
      index,
    }));
    data = { candidates };
  }
  const cand = data.candidates[0];
  console.assert(data.candidates.length === 1, "Unexpected candidates count: %d", data.candidates.length);
  cand.index = cand.index || 0;
  if (!this.last[cand.index]) {
    controller.enqueue(transform(data, false, "first"));
  }
  this.last[cand.index] = data;
  if (cand.content) {
    controller.enqueue(transform(data));
  }
}

async function toOpenAiStreamFlush(controller) {
  const transform = transformResponseStream.bind(this);
  if (this.last.length > 0) {
    for (const data of this.last) {
      controller.enqueue(transform(data, "stop"));
    }
    controller.enqueue("data: [DONE]" + delimiter);
  }
}

const errHandler = (err) => {
  console.error(err);
  return new Response(err.message, fixCors({ status: err.status ?? 500 }));
};

function extractPrefixAndRest(pathname, prefixes) {
  for (const prefix of prefixes) {
    if (pathname.startsWith(prefix)) {
      return [prefix, pathname.slice(prefix.length)];
    }
  }
  return [null, null];
}

serve(async (request) => {
  const url = new URL(request.url);
  const pathname = url.pathname;

  if (pathname === '/' || pathname === '/index.html') {
    return new Response('Service is running!', {
      status: 200,
      headers: { 'Content-Type': 'text/html' }
    });
  }

  if (pathname === '/robots.txt') {
    return new Response('User-agent: *\nDisallow: /', {
      status: 200,
      headers: { 'Content-Type': 'text/plain' }
    });
  }

  // 特殊处理 /gemini 路径
  if (pathname.startsWith('/gemini')) {
    if (request.method === "OPTIONS") {
      return handleOPTIONS();
    }

    try {
      const auth = request.headers.get("Authorization");
      const apiKey = auth?.split(" ")[1];

      if (pathname.endsWith("/chat/completions")) {
        return handleCompletions(await request.json(), apiKey)
          .catch(errHandler);
      } else if (pathname.endsWith("/embeddings")) {
        return handleEmbeddings(await request.json(), apiKey)
          .catch(errHandler);
      } else if (pathname.endsWith("/models")) {
        return handleModels(apiKey)
          .catch(errHandler);
      } else {
        throw new HttpError("404 Not Found", 404);
      }
    } catch (err) {
      return errHandler(err);
    }
  }

  // 处理其他 API 路径
  const [prefix, rest] = extractPrefixAndRest(pathname, Object.keys(apiMapping));
  if (!prefix) {
    return new Response('Not Found', { status: 404 });
  }

  const targetUrl = `${apiMapping[prefix]}${rest}`;

  try {
    const headers = new Headers();
    const allowedHeaders = ['accept', 'content-type', 'authorization'];
    for (const [key, value] of request.headers.entries()) {
      if (allowedHeaders.includes(key.toLowerCase())) {
        headers.set(key, value);
      }
    }

    const response = await fetch(targetUrl, {
      method: request.method,
      headers: headers,
      body: request.body
    });

    const responseHeaders = new Headers(response.headers);
    responseHeaders.set('X-Content-Type-Options', 'nosniff');
    responseHeaders.set('X-Frame-Options', 'DENY');
    responseHeaders.set('Referrer-Policy', 'no-referrer');

    return new Response(response.body, {
      status: response.status,
      headers: responseHeaders
    });

  } catch (error) {
    console.error('Failed to fetch:', error);
    return new Response('Internal Server Error', { status: 500 });
  }
});
```
### 食用方法
1. 复制上面的代码。
2. 打开 [Deno Playground](https://dash.deno.com)。
3. 用github账号登录。
   ![image](https://i.111666.best/image/ev3jS05Ohuu4pyGiv7O270.png)
4. 点击蓝色的 `New Playground` 按钮。
   ![image](https://i.111666.best/image/0rSkYIFxyH04upWw1EhDWj.png)
5. 粘贴代码到左边的编辑器中。
6. 点击上方的 `Save & Deploy` 按钮。
7. 当右边的终端显示 `Service is running!` 时,你的服务就已经部署成功了。
8. 在`Settings`中可以设置你的域名。
   ![image](https://i.111666.best/image/1Mge6Q5OFH4xuyDamPfPTt.png)
9. 可以参考下面的表格,将BASE_URL改成对应的代理地址。
    例如: `BASE_URL = "https://api.openai.com";` 改成 `BASE_URL = "https://你的地址/openai";`
    请求就会变成 `https://你的地址/openai/v1/completions`。

:::important
Gemini已经转成了Openai格式，所以Gemini的请求地址也要改成Openai格式的。不能直接使用Gemini的请求格式。
如果只想单纯的代理Gemini的请求，可以部署下面的代码。
:::

```typescript
import { serve } from "https://deno.land/std/http/server.ts";

const apiMapping = {
  '/discord': 'https://discord.com/api',
  '/telegram': 'https://api.telegram.org',
  '/openai': 'https://api.openai.com',
  '/claude': 'https://api.anthropic.com',
  '/meta': 'https://www.meta.ai/api',
  '/groq': 'https://api.groq.com/openai',
  '/xai': 'https://api.x.ai',
  '/cohere': 'https://api.cohere.ai',
  '/huggingface': 'https://api-inference.huggingface.co',
  '/together': 'https://api.together.xyz',
  '/novita': 'https://api.novita.ai',
  '/portkey': 'https://api.portkey.ai',
  '/fireworks': 'https://api.fireworks.ai',
  '/openrouter': 'https://openrouter.ai/api',
  '/nvidia': 'https://integrate.api.nvidia.com',
  '/cerebras': 'https://api.cerebras.ai',
  '/sambanova': 'https://api.sambanova.ai',
  '/gemini': 'https://generativelanguage.googleapis.com'
};

const CORS_HEADERS = {
  "access-control-allow-origin": "*",
  "access-control-allow-methods": "*",
  "access-control-allow-headers": "*",
};

serve(async (request) => {
  // 处理 CORS 预检请求
  if (request.method === "OPTIONS") {
    return new Response(null, {
      headers: CORS_HEADERS,
    });
  }

  const url = new URL(request.url);
  const pathname = url.pathname;

  if (pathname === '/' || pathname === '/index.html') {
    return new Response('Service is running!', {
      status: 200,
      headers: { 
        'Content-Type': 'text/html',
        ...CORS_HEADERS 
      }
    });
  } 
  
  if (pathname === '/robots.txt') {
    return new Response('User-agent: *\nDisallow: /', {
      status: 200,
      headers: { 
        'Content-Type': 'text/plain',
        ...CORS_HEADERS 
      }
    });
  }

  const [prefix, rest] = extractPrefixAndRest(pathname, Object.keys(apiMapping));
  if (!prefix) {
    return new Response('Not Found', { status: 404 });
  }

  try {
    // 特殊处理 Gemini API 请求
    if (prefix === '/gemini') {
      const targetUrl = new URL(`${apiMapping[prefix]}${rest}`);
      
      // 转发所有查询参数
      url.searchParams.forEach((value, key) => {
        targetUrl.searchParams.append(key, value);
      });

      const headers = new Headers();
      const geminiHeaders = [
        'accept',
        'content-type',
        'authorization',
        'x-goog-api-client',
        'x-goog-api-key'
      ];

      for (const [key, value] of request.headers.entries()) {
        if (geminiHeaders.includes(key.toLowerCase())) {
          headers.set(key, value);
        }
      }

      const response = await fetch(targetUrl, {
        method: request.method,
        headers: headers,
        body: request.body
      });

      const responseHeaders = new Headers({
        ...CORS_HEADERS,
        ...Object.fromEntries(response.headers),
        'X-Content-Type-Options': 'nosniff',
        'X-Frame-Options': 'DENY',
        'Referrer-Policy': 'no-referrer'
      });

      return new Response(response.body, {
        status: response.status,
        headers: responseHeaders
      });
    }

    // 处理其他 API 请求
    const targetUrl = `${apiMapping[prefix]}${rest}`;
    const headers = new Headers();
    const allowedHeaders = ['accept', 'content-type', 'authorization'];
    
    for (const [key, value] of request.headers.entries()) {
      if (allowedHeaders.includes(key.toLowerCase())) {
        headers.set(key, value);
      }
    }

    const response = await fetch(targetUrl, {
      method: request.method,
      headers: headers,
      body: request.body
    });

    const responseHeaders = new Headers(response.headers);
    responseHeaders.set('X-Content-Type-Options', 'nosniff');
    responseHeaders.set('X-Frame-Options', 'DENY');
    responseHeaders.set('Referrer-Policy', 'no-referrer');
    
    // 添加 CORS 头
    Object.entries(CORS_HEADERS).forEach(([key, value]) => {
      responseHeaders.set(key, value);
    });

    return new Response(response.body, {
      status: response.status,
      headers: responseHeaders
    });

  } catch (error) {
    console.error('Failed to fetch:', error);
    return new Response('Internal Server Error', { status: 500 });
  }
});

function extractPrefixAndRest(pathname: string, prefixes: string[]): [string | null, string | null] {
  for (const prefix of prefixes) {
    if (pathname.startsWith(prefix)) {
      return [prefix, pathname.slice(prefix.length)];
    }
  }
  return [null, null];
}
```

### 代理地址

| 代理地址 | 源地址 |
|----------|---------|
| `https://你的地址/anthropic` | `https://api.anthropic.com` |
| `https://你的地址/cerebras` | `https://api.cerebras.ai` |
| `https://你的地址/cohere` | `https://api.cohere.ai` |
| `https://你的地址/discord` | `https://discord.com/api` |
| `https://你的地址/fireworks` | `https://api.fireworks.ai` |
| `https://你的地址/gemini` | `https://generativelanguage.googleapis.com` |
| `https://你的地址/groq` | `https://api.groq.com/openai` |
| `https://你的地址/huggingface` | `https://api-inference.huggingface.co` |
| `https://你的地址/meta` | `https://www.meta.ai/api` |
| `https://你的地址/novita` | `https://api.novita.ai` |
| `https://你的地址/nvidia` | `https://integrate.api.nvidia.com` |
| `https://你的地址/openai` | `https://api.openai.com` |
| `https://你的地址/openrouter` | `https://openrouter.ai/api` |
| `https://你的地址/portkey` | `https://api.portkey.ai` |
| `https://你的地址/telegram` | `https://api.telegram.org` |
| `https://你的地址/together` | `https://api.together.xyz` |
| `https://你的地址/xai` | `https://api.x.ai` |
| `https://你的地址/sambanova` | `https://api.sambanova.ai` |

### 参考项目
- [openai-gemini](https://github.com/PublicAffairs/openai-gemini)
