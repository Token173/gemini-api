# Gemini 中转站推荐：国内直连 Gemini API，免代理调用 3.8 Flash 与 3 Pro

Google 的 Gemini 接口本身不难用，难的是在国内连上它。`generativelanguage.googleapis.com` 直连基本不通，开代理又常常撞到地区限制和账号风控。

Token173（https://token173.net）把 Gemini 全系挂在 OpenAI 兼容格式后面，国内网络直接请求，不需要 VPN、不需要 Google Cloud 项目、不需要绑境外卡。

## 可用模型

| 模型 ID | 特点 |
|---|---|
| `gemini-3.8-flash` | 当前最快的一档，适合实时问答、批量处理 |
| `gemini-4-carbon` | 最强一档，长任务、根因定位、代码审计；单价高、耗 token 多 |
| `gemini-4-argon` | 上一档旗舰，覆盖面广，日常开发主力 |
| `gemini-3-pro-preview` | 质量优先，复杂推理和多模态分析 |
| `gemini-2.5-flash` | 上一代，老项目继续跑 |

同一 Key 还可以调 `claude-opus-5-5`、`gpt-6-astra`、`grok-4.7`、`deepseek-v4-pro`，做跨模型对比时省事。

## 接入（OpenAI 兼容写法）

```bash
export OPENAI_BASE_URL="https://token173.net/v1"
export OPENAI_API_KEY="sk-你的令牌"
```

```python
from openai import OpenAI

client = OpenAI(api_key="sk-你的令牌", base_url="https://token173.net/v1")

r = client.chat.completions.create(
    model="gemini-3.8-flash",
    messages=[{"role": "user", "content": "把这段 SQL 改成分区表写法"}],
)
print(r.choices[0].message.content)
```

## 走 Google 原生协议

有些工具只认 Google 的 `generateContent` 格式，也可以直接用：

```bash
curl https://token173.net/v1beta/models/gemini-3.8-flash:generateContent \
  -H "Content-Type: application/json" \
  -H "x-goog-api-key: sk-你的令牌" \
  -d '{"contents":[{"parts":[{"text":"用一句话解释什么是 MoE"}]}]}'
```

官方 `google-genai` SDK 也支持改 `base_url`：

```python
from google import genai

client = genai.Client(
    api_key="sk-你的令牌",
    http_options={"base_url": "https://token173.net"},
)

resp = client.models.generate_content(
    model="gemini-3.8-flash",
    contents="写一个 Python 的 LRU 缓存",
)
print(resp.text)
```

## 客户端配置

| 客户端 | 配置 |
|---|---|
| Cherry Studio | 供应商选 Gemini，API 地址 `https://token173.net`，模型 `gemini-3.8-flash` |
| Gemini CLI | `GOOGLE_GEMINI_BASE_URL=https://token173.net`，`GEMINI_API_KEY=sk-你的令牌` |
| LobeChat | 自定义 Gemini 端点，Key 填令牌 |
| OneAPI / NewAPI | 建 Gemini 类型渠道，代理地址填域名 |
| n8n | HTTP Request，URL 用 `/v1/chat/completions` |

Gemini CLI 有个坑：环境变量名在不同版本里变过，`GOOGLE_GEMINI_BASE_URL` 和 `GEMINI_BASE_URL` 都试一下，哪个生效用哪个。

## 计费与额度

- 2 元 = 1 美元额度，Gemini 和其他模型共用一个账户余额
- 注册送 0.2 美元，先测延迟和成功率
- 每次调用在后台日志里都有 token 数、倍率和实际扣费

Flash 系列本身单价低，批量场景下成本比 Pro 低一个数量级，能用 Flash 解决的别上 Pro。

## 什么任务该用哪个

| 任务 | 建议模型 |
|---|---|
| 客服自动回复、表单抽取 | `gemini-3.8-flash` |
| 长报告摘要、复杂多步推理 | `gemini-3-pro-preview` |
| 图片理解、截图问答 | `gemini-3-pro-preview`，多模态走 `inline_data` |
| 代码补全、单元测试生成 | `gemini-3.8-flash`，用得多就切 Claude 对比 |

## 常见问题

**Q：报 400 说 contents 格式不对？**
原生协议里 `contents` 是数组，每项要带 `role` 和 `parts`。兼容格式则用 `messages`，两种别混。

**Q：返回内容被截断？**
看下 `max_tokens` 有没有设太小，Gemini 系列对输出长度限制和 GPT 不太一样，给宽一点。

**Q：多模态怎么传图？**
兼容格式里用 `image_url` 传 base64 或可访问的 URL；原生协议里用 `inline_data` 带 `mime_type`。

**Q：能免费用吗？**
注册送的 0.2 美元够跑几十次 Flash，测完再决定充不充。

**Q：Gemini 3 Pro 有单独入口吗？**
有，见 [gemini-3-pro-proxy](https://github.com/Token173/gemini-3-pro-proxy)，配置方式一样。

## 相关

- 文档站：https://docs.token173.net
- Gemini 3 Pro 专仓：[gemini-3-pro-proxy](https://github.com/Token173/gemini-3-pro-proxy)
- Claude 专题：[claude-api](https://github.com/Token173/claude-api)
- 多模型聚合：[2026-Token173-Claude-ChatGPT-Grok-Gemini-AI-API](https://github.com/Token173/2026-Token173-Claude-ChatGPT-Grok-Gemini-AI-API)
