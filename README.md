# 🚀 AxonHub 快速配置工具 (AxonHub Config Generator)

![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat-square&logo=html5&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black)
![TailwindCSS](https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=flat-square&logo=tailwind-css&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square)

这是一个基于纯前端（HTML + 原生 JavaScript + Tailwind CSS）实现的单页应用程序。该工具旨在将杂乱的 API 渠道（Channels）配置、复杂的 LLM 模型（Models）匹配规则以及 API 访问凭证（API Keys），自动化地转换为 **AxonHub** 标准的 JSON 配置文件（备份/恢复文件）。

无需部署任何后端服务，**直接双击 HTML 文件即可在浏览器中运行**，数据完全在本地处理，安全可靠。

---

## 📑 目录

- [✨ 核心特性](#-核心特性)
- [⚠️ 常见问题与注意事项](#-常见问题与注意事项)
- [📖 使用指南](#-使用指南)
  - [1. 运行方式](#1-运行方式)
  - [2. Channels 配置格式](#2-channels-配置格式)
  - [3. Models 配置格式](#3-models-配置格式)
  - [4. API Keys 配置格式](#4-api-keys-配置格式)
- [🛡️ 关于 CORS 跨域代理](#️-关于-cors-跨域代理)
- [📄 输出 JSON 结构概览](#-输出-json-结构概览)
- [🙏 鸣谢 (Acknowledgments)](#-鸣谢--acknowledgments)
- [📜 开源证书 (License)](#-开源证书--license)

---

## ✨ 核心特性

### 📡 渠道配置转换 (Channels)
*   **多格式支持**：支持通过 **YAML** 或 **CSV** 格式批量导入 API Key 和 URL，支持显式指定 `type`。
*   **智能 URL 补全**：内置 AxonHub 官方的 `CHANNEL_CONFIGS` 映射表，根据类型（如 `deepseek_anthropic`, `zhipu` 等）自动为 Base URL 拼接正确的后缀（如 `/v1`, `/api/paas/v4` 等）。
*   **并发与重连机制**：并发请求所有 API 的 `/v1/models` 接口获取可用模型；内置 `10s` -> `20s` -> `30s` 阶梯式超时重连机制。
*   **智能跨域回退 (CORS)**：检测到浏览器跨域拦截时，自动无缝回退至配置的 CORS 代理地址重新发起请求。
*   **类型与模型智能推断**：
    *   智能识别 `gpt-5` 及以上版本与 `codex` 模型，自动归类为 `openai_responses`。
    *   根据模型名称自动补全标签（如 `glm`, `kimi`, `deepseek`, `claude` 等）。

### 🧠 模型元数据生成 (Models)
*   **云端元数据同步**：自动从 Cloudflare Pages 获取最新的 LLM 元数据（失败时自动回退至 GitHub Pages）。
*   **高级正则匹配**：支持通过 YAML 配置 `includes` 和 `excludes` 正则表达式批量筛选模型。
*   **智能版本解析与全排列**：自动解析模型版本号（如将 `4-5` 转换为 `4.5`，单数字版本自动补齐 `-`），并自动生成模型名称各组件的**全排列正则匹配规则**（Associations）。
*   **Model Card 补全**：自动填充模型的上下文长度、定价（Cost）、多模态能力（Vision/Audio）、工具调用（Tool Call）及推理能力（Reasoning）等元数据。

### 🔑 访问凭证生成 (API Keys)
*   **批量生成**：支持通过 YAML 或 CSV 批量创建 AxonHub 访问令牌。
*   **智能校验与安全生成**：自动补齐 `ah-` 前缀；如果用户提供的密钥不符合 64 位十六进制标准，工具将使用高强度的 `crypto.getRandomValues` 自动生成安全的随机密钥。

### 💻 可视化与交互体验
*   **多列独立输入**：Channels、Models 和 API Keys 配置区域相互独立，互不干扰，逻辑清晰。
*   **终端级实时日志**：内置深色日志面板，实时打印网络请求、跨域回退、重连次数及模型匹配细节。
*   **数据统计面板**：实时展示成功数、失败数、生成的渠道数、生成的模型数以及 API Keys 数量。
*   **一键复制**：一键将生成的标准 JSON 复制到剪贴板，直接导入 AxonHub。

---

## ⚠️ 常见问题与注意事项

**❓ 为什么直接将 channels 中 `gpt-5` 及以上版本（含 `codex`）的 `type` 都归类为 `openai_responses`？**
> 因为目前大部分中转站点都已经支持该格式，而且**有些站点只支持使用 `openai_responses` 格式**来进行访问。为了最大程度适配这些强制要求的站点，工具默认进行了此转换。

**💡 兼容性提示：**
> 虽然大部分站点支持，但仍有**少部分中转站（API 代理商）尚未完全兼容 `openai_responses` 格式**。如果您在后续使用生成的配置时，遇到请求报错或访问失败的问题，请自行在生成的 JSON 中，将对应渠道的 `type` 字段手动修改回 `openai`。或者在 AxonHub 渠道管理页面中复制新的渠道，将 API 格式改为 `OpenAI (Chat Completions)`。

---

## 📖 使用指南

### 1. 运行方式
*   **在线使用**：直接访问 [Github Pages](https://osozos.github.io/fast_axonhub_config/) 即可使用。
*   **本地使用**：下载本仓库中的 `index.html`，直接使用 Chrome、Edge 或 Firefox 等现代浏览器打开即可。

### 2. Channels 配置格式
支持 YAML 和 CSV 两种格式混合输入。

**YAML 格式示例：**
```yaml
- url: https://api.example.com
  keys: 
    - sk-xxxxxx1
    - sk-xxxxxx2
  name: myapi
  type: claudecode  # 可选：显式指定类型，跳过自动推断，并自动匹配对应后缀
  tags:
    - 公益站
```

**CSV 格式示例：**
```csv
# 格式: URL,apiKey[,name[,type[,tags]]] (tags用|分隔)
192.168.1.128:3000,sk-yyyyyy,localhost,zhipu_anthropic,公益站|claude
```

### 3. Models 配置格式
仅支持 YAML 格式。支持指定 `providers` 遍历，以及复杂的正则匹配规则。

**YAML 格式示例：**
```yaml
# 默认提供商（可选）
# 当包含规则没有指定提供商时，遍历使用这些提供商进行匹配查询
providers:
  # 国产模型厂商
  - deepseek
  - alibaba
  - zai
  - doubao
  - moonshotai
  - minimax
  - xiaomi
  # 知名模型厂商
  - openai
  - anthropic
  - google
  - xai
  # 第三方厂商
  - vercel
  - openrouter
  - 302ai

# 包含规则：匹配这些模式的模型会被转换
# 格式: "pattern1|pattern2,provider" 或 "pattern1|pattern2"
# 当 include 没有指定厂商时，默认搭配 provider 遍历查询
includes:
  # Anthropic Claude 4.5+
  - sonnet-4-[5-9]|sonnet-4\\.[5-9]|4-[5-9]-sonnet|4\\.[5-9]-sonnet,anthropic
  - opus-4-[5-9]|opus-4\\.[5-9]|4-[5-9]-opus|4\\.[5-9]-opus,anthropic
  - haiku-4-[5-9]|haiku-4\\.[5-9]|4-[5-9]-haiku|4\\.[5-9]-haiku,anthropic

  # OpenAI GPT 5+
  - gpt-5|gpt.*-5(\\.\\d)?,openai
  - gpt-oss,openai

  # Google Gemini 3+
  - gemini.*3(.\\d+)?|gemini-3(.\\d+)?,google

  # xAI Grok 4+
  - grok.*4(.\\d+)?|grok-4(.\\d+)?,xai
  - grok.*code|grok-code,xai

  # DeepSeek 3.2+
  - deepseek.*3\\.[2-9]|deepseek.*3\\.\\d{2,}|deepseek.*[4-9](\\.\\d)?,deepseek

  # Moonshot Kimi 2.5+
  - kimi.*2\\.[5-9]|kimi.*2\\.\\d{2,}|kimi.*[3-9](\\.\\d)?,moonshotai

  # MiniMax 2.1+
  - minimax.*2\\.[1-9]|minimax.*2\\.\\d{2,}|minimax.*[3-9](\\.\\d)?,minimax

  # Zhipu GLM 4.7+
  - glm.*4\\.[7-9]|glm.*4\\.\\d{2,}|glm.*[5-9](\\.\\d)?,zai

# 排除规则：匹配这些模式的模型会被过滤掉（可选）
excludes:
  - think
  - thinking
  - image
  - search
```

### 4. API Keys 配置格式
支持 YAML 和 CSV 格式。如果只提供名称，系统会自动生成符合规范的高强度密钥。

**YAML 格式示例：**
```yaml
- name: opencode
  key: ah-b09cb09f55be265b275e8e1b4832c1eee36c6c358ecc9f6110c88d56f0ec24c7
- name: claudecode
```

**CSV 格式示例：**
```csv
# 格式: name[,key]
gemini
codex,ah-1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef
```

---

## 🛡️ 关于 CORS 跨域代理

由于浏览器的同源安全策略，纯前端直接请求第三方 API 时常会遇到 `Origin 'null' has been blocked by CORS policy` 错误。

工具内置了代理回退机制，**强烈建议使用自建的 Cloudflare Worker 作为代理**，以保证您的 API Key 绝对安全，不经过任何第三方公共服务器。

### 如何部署免费专属代理 (Cloudflare Worker):
1. 登录 [Cloudflare](https://dash.cloudflare.com/)，进入 **Workers & Pages** -> **创建 Worker**。
2. 将以下代码粘贴到编辑器中并部署：
```javascript
export default {
  async fetch(request, env, ctx) {
    if (request.method === "OPTIONS") {
      return new Response(null, {
        headers: {
          "Access-Control-Allow-Origin": "*",
          "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE, OPTIONS",
          "Access-Control-Allow-Headers": "Content-Type, Authorization",
        },
      });
    }
    const url = new URL(request.url);
    const targetUrlStr = url.searchParams.get("url");
    if (!targetUrlStr) return new Response("Missing ?url=", { status: 400 });

    try {
      const targetUrl = new URL(targetUrlStr);
      const newRequest = new Request(targetUrl, {
        method: request.method,
        headers: request.headers,
        body: request.body,
      });
      const response = await fetch(newRequest);
      const newResponse = new Response(response.body, response);
      newResponse.headers.set("Access-Control-Allow-Origin", "*");
      return newResponse;
    } catch (e) {
      return new Response(e.message, { status: 500 });
    }
  },
};
```
3. 在转换器界面的“代理地址”输入框中填入：`https://您的worker域名.workers.dev/?url=` 即可。

---

## 📄 输出 JSON 结构概览

转换完成后，工具会生成包含 `version`、`timestamp`、`channels`、`models` 和 `api_keys` 的标准 JSON：

```json
{
    "version": "1.1",
    "timestamp": "2026-02-24T12:00:00.000000Z",
    "channels": [
        {
            "id": 1,
            "type": "anthropic",
            "base_url": "https://api.example.com",
            "name": "myapi (claude)",
            "supported_models": ["claude-3-opus", "claude-3-sonnet"],
            "credentials": {
                "apiKey": "sk-xxxxxx"
            }
        }
    ],
    "models": [
        {
            "id": 1,
            "developer": "openai_responses",
            "model_id": "gpt-5",
            "name": "GPT-5",
            "model_card": {
                "reasoning": { "supported": true, "default": true },
                "cost": { "input": 1.25, "output": 10 }
            },
            "settings": {
                "associations": [
                    { "regex": { "pattern": ".*gpt.*-5" } }
                ]
            }
        }
    ],
    "api_keys": [
        {
            "id": 1,
            "user_id": 1,
            "project_id": 1,
            "key": "ah-b09cb09f55be265b275e8e1b4832c1eee36c6c358ecc9f6110c88d56f0ec24c7",
            "name": "opencode",
            "type": "user",
            "status": "enabled",
            "scopes": ["read_channels", "write_requests"],
            "project_name": "Default"
        }
    ]
}
```

---

## 🙏 鸣谢 | Acknowledgments

本项目的诞生离不开以下优秀的开源项目、数据源与 AI 助手的支持，在此表示诚挚的感谢：

*   **[AxonHub](https://github.com/looplj/axonhub)** - 强大的 AI 网关，让你无需改动一行代码即可无缝切换模型供应商。
*   **[LLM 元数据 (LLM Metadata)](https://github.com/basellm/llm-metadata)** - 提供模型基础数据的轻量级静态 API。
*   **Gemini** - 大语言模型（AI）辅助编写与迭代完成。

---

## 📜 开源证书 | License

本项目采用 [MIT License](https://opensource.org/licenses/MIT) 开源协议。

这意味着您可以完全免费地将本工具用于个人或商业项目。您可以自由地使用、复制、修改、合并、发布分发、再授权及售卖本软件的副本，只需在软件和软件的所有副本中包含上述版权声明和本许可声明即可。
