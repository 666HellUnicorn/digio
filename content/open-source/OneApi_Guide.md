+++
date = '2026-05-09T08:43:10+08:00'
draft = false
title = 'One API 使用教程'
tags = ['open-source']
listIcon = "/contribute.png"
teaser = "在 CCSwitch 中配置和使用 One API 中转服务。"
+++
# CCSwitch 使用中转 API 教程

本文档用于指导你在 CCSwitch 中使用统一中转 API。

请注意：你只需要使用中转 API 地址和令牌，不需要也不应该填写任何上游平台的原始 API Key。

## 一、准备信息

请先向管理员获取以下信息：

```text
Base URL：https://api.hellcious.lol/v1
API Key：由自己分配的 One API 令牌，通常以 sk- 开头
模型名称：例如 qwen3.6-plus、deepseek-v3.2、glm-5、mimo-v2.5-pro
```

示例：

```text
Base URL：https://api.hellcious.lol/v1
API Key：sk-xxxxxxxxxxxxxxxx
Model：qwen3.6-plus
```

## 二、在 CCSwitch 中添加配置

打开 CCSwitch，进入供应商或 API 配置页面。

选择：

```text
添加供应商 / Add Provider
自定义 / Custom
OpenAI Compatible / OpenAI 兼容
```

然后填写：

```text
名称：Hellcious One API
Base URL：https://api.hellcious.lol/v1
API Key：填写你的 sk- 令牌
模型：填写管理员提供的模型名称（可通过获取模型列表来确认）
```



```text
推荐在opencode中使用（经过测试）
```



## 三、获取模型列表是什么意思

CCSwitch 里的“获取模型列表”会请求：

```text
https://api.hellcious.lol/v1/models
```

它的作用是自动读取当前令牌可用的模型，避免手动输入模型名。

如果获取失败，也可以手动填写模型名称。模型名称必须和管理员提供的一致，大小写也要一致。

## 四、手动配置模型 JSON

如果 CCSwitch 需要手动填写模型配置，可以参考：

```json
[
  {
    "id": "qwen3.6-plus",
    "name": "Qwen 3.6 Plus"
  },
  {
    "id": "deepseek-v3.2",
    "name": "DeepSeek V3.2"
  },
  {
    "id": "glm-5",
    "name": "GLM 5"
  },
  {
    "id": "mimo-v2.5-pro",
    "name": "Mimo V2.5 Pro"
  }
]
```

说明：

```text
id：真实请求使用的模型名，不能写错
name：界面显示名称，可以自定义
```

## 五、测试是否可用

保存配置后，点击 CCSwitch 的测试按钮。

如果测试通过，说明：

```text
Base URL 可以连接
API Key 可用
模型基本可用
```

之后在支持 OpenAI 兼容 API 的工具中选择该配置即可。

## 六、常见错误

### 1. 401 Unauthorized

通常是 API Key 错误。

请检查：

```text
是否复制完整
是否包含多余空格
是否使用了管理员分配的 One API 令牌
```

不要填写 Mimo、阿里百炼等上游平台的原始 Key。

### 2. 404 Not Found

通常是 Base URL 写错。

正确写法：

```text
https://api.hellcious.lol/v1
```

不要写成：

```text
https://api.hellcious.lol
```

### 3. 400 Not supported model

通常是模型名写错，或者当前令牌没有权限使用该模型。

请确认模型名完全一致，例如：

```text
mimo-v2.5-pro
```

不要写成：

```text
MiMo-V2.5-Pro
```

模型名大小写可能敏感。



确认可用后，再切换到其他模型。

## 七、安全注意事项

请妥善保管你的 API Key：

```text
不要发到公开群聊
不要提交到 GitHub
不要写进前端网页代码
不要分享给其他人
```

如果怀疑 Key 泄露，请自行重新生成。

