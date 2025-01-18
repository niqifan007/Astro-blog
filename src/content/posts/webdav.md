---
title: 使用docker在Debian12搭建Webdav服务端
published: 2025-01-19
description: 'WebDAV 服务器搭建教程：使用 Docker 快速部署文件共享服务'
image: 'https://i.111666.best/image/8D09vpTzVLpPGxOwgqgGCW.jpg'
tags: [linux]
category: 'Guides'
draft: false 
lang: ''
---
# WebDAV 服务器搭建教程：使用 Docker 快速部署文件共享服务

## 前言
WebDAV (Web Distributed Authoring and Versioning) 是 HTTP 协议的扩展，允许客户端对 Web 服务器上的内容进行在线编辑和管理。本教程将介绍如何使用 Docker 快速搭建一个安全的 WebDAV 服务器。

## 项目地址
::github{repo="hacdias/webdav"}

## 环境要求
- Docker 环境
- 一台 Linux/Windows/MacOS 服务器或个人电脑

## 步骤一：创建配置文件
首先创建一个工作目录并进入：
```bash
mkdir webdav
cd webdav
```

创建配置文件 `config.yml`：
```yaml
# 服务器地址和端口配置
address: 0.0.0.0
port: 6065

# 基础配置
tls: false
prefix: /
debug: false
noSniff: false
behindProxy: false
directory: .

# 权限配置（完整读写权限）
permissions: CRUD

# CORS配置（允许网页访问）
cors:
  enabled: true
  credentials: true
  allowed_headers:
    - Depth
    - Authorization
    - Content-Type
    - Content-Length
  allowed_hosts:
    - "*"
  allowed_methods:
    - GET
    - PUT
    - POST
    - DELETE
    - PROPFIND
    - MKCOL
    - COPY
    - MOVE
  exposed_headers:
    - Content-Length
    - Content-Range

# 用户认证配置
users:
  - username: waka
    password: wsadwsad
```

## 步骤二：创建数据目录
```bash
mkdir data
```

## 步骤三：启动 Docker 容器
```bash
docker run \
  -p 6065:6065 \
  -v $(pwd)/config.yml:/config.yml:ro \
  -v $(pwd)/data:/data \
  ghcr.io/hacdias/webdav -c /config.yml
```

## 配置说明

### 1. 端口配置
- `address: 0.0.0.0` 允许所有网络接口访问
- `port: 6065` 服务监听端口

### 2. 权限设置
权限类型包括：
- C：Create（创建）
- R：Read（读取）
- U：Update（更新）
- D：Delete（删除）

### 3. CORS 配置
- `enabled: true` 启用跨域访问
- `allowed_hosts: "*"` 允许所有域名访问
- `allowed_methods` 配置允许的 HTTP 方法

### 4. 安全性考虑
- 建议修改默认用户名和密码
- 如果不需要网页访问，可以禁用 CORS
- 生产环境建议启用 TLS 加密
  
## 进阶配置

### 反向代理配置
如果你使用 Nginx 作为反向代理，需要正确转发头信息以避免 502 错误：

```nginx
location / {
  proxy_pass http://127.0.0.1:6065;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header REMOTE-HOST $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header Host $host;
  proxy_redirect off;
}
```

## 访问方式

### 1. 网页访问
```
http://服务器IP:6065
```

### 2. WebDAV 客户端
- MacOS: Cyberduck

## 常见问题

### 1. 403 权限错误
检查：
- permissions 是否配置为 CRUD
- 用户权限是否正确
- 目录权限是否正确

### 2. CORS 错误
检查：
- allowed_hosts 配置
- allowed_methods 是否包含需要的方法
- allowed_headers 是否完整

## 总结
通过 Docker 部署 WebDAV 服务器具有以下优势：
1. 部署简单快速
2. 配置灵活
3. 易于维护
4. 支持多平台

## 附录：官方配置文件示例
```yaml
# 服务器地址和端口配置
address: 0.0.0.0
port: 6065

# TLS 相关设置（如果你想直接启用 TLS）
tls: false
cert: cert.pem
key: key.pem

# WebDAV 路径的前缀。默认为 '/'
prefix: /

# 启用或禁用调试日志。默认为 'false'
debug: false

# 禁用文件内容类型嗅探。默认为 'false'
noSniff: false

# 服务器是否运行在受信任的代理后面。当设置为 true 时，
# 将使用 X-Forwarded-For 头部来记录访问尝试的远程地址（如果可用）
behindProxy: false

# 用户连接时可以访问的目录。
# 除非用户有自己的 'directory' 定义，否则将使用此目录。
# 默认为 '.'（当前目录）
directory: .

# 用户的默认权限。这是一个不区分大小写的选项。可能的权限包括：
# C（创建）、R（读取）、U（更新）、D（删除）。
# 你可以组合多个权限。例如，要允许读取和创建，设置为 "RC"。默认为 "R"
permissions: R

# 用户的默认权限规则。默认为空。规则从后向前应用，
# 即从末尾开始匹配请求的第一条规则将被应用
rules: []

# 重新定义用户规则的行为。可以是：
# - overwrite: 当用户有规则定义时，这些规则将覆盖任何已定义的全局规则。
#   即全局规则不适用于该用户。
# - append: 当用户有规则定义时，这些规则将附加到已定义的全局规则之后。
#   即对于该用户，首先检查其特定规则，然后是全局规则。
# 默认为 'overwrite'
rulesBehavior: overwrite

# 日志配置
log:
  # 日志格式（'console' 或 'json'）。默认为 'console'
  format: console
  # 启用或禁用颜色。默认为 'true'。仅在 format 为 'console' 时适用
  colors: true
  # 日志输出。可以有多个输出。默认仅为 'stderr'
  outputs:
    - stderr

# CORS 配置
cors:
  # 是否应用 CORS 配置。默认为 'false'
  enabled: true
  credentials: true
  allowed_headers:
    - Depth
  allowed_hosts:
    - http://localhost:8080
  allowed_methods:
    - GET
  exposed_headers:
    - Content-Length
    - Content-Range

# 用户列表。如果列表为空，则不会有身份验证。
# 否则，将自动配置基本身份验证。
#
# 如果你将身份验证委托给其他服务，你可以使用基本身份验证代理用户名，
# 然后使用以下选项禁用 webdav 的密码检查：
#
# noPassword: true
users:
  # 示例 'admin' 用户，使用明文密码
  - username: admin
    password: admin
  # 示例 'john' 用户，使用 bcrypt 加密密码，具有自定义目录。
  # 你可以使用 'webdav bcrypt' 命令行工具生成 bcrypt 加密密码
  - username: john
    password: "{bcrypt}$2y$10$zEP6oofmXFeHaeMfBNLnP.DO8m.H.Mwhd24/TOX2MWLxAExXi4qgi"
    directory: /another/path
  # 示例用户，其详细信息将从环境变量中获取
  - username: "{env}ENV_USERNAME"
    password: "{env}ENV_PASSWORD"
  - username: basic
    password: basic
    # 覆盖默认权限
    permissions: CRUD
    rules:
      # 通过此规则，用户不能访问 /some/files
      - path: /some/file
        permissions: none
      # 通过此规则，用户可以在 /public/access 中创建、读取、更新和删除
      - path: /public/access/
        permissions: CRUD
      # 通过此规则，用户可以读取和更新所有以 .js 结尾的文件。使用正则表达式
      - regex: "^.+.js$"
        permissions: RU
```