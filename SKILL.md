---
name: sensenova-u1-image
description: Generate images via SenseNova U1 model (商汤日日新) using curl to call the text-to-image API. Trigger when the user mentions 商汤, SenseNova, U1, 文生图, text-to-image generation, or wants to create images from text descriptions using this specific model.
---

# SenseNova U1 文生图

通过商汤 SenseNova U1 模型 API，将文字描述生成图片。

---

## 快速开始

### 1. 配置 Token

首次使用需要 API Token：

1. **检查已有 Token**：在 MEMORY.md 的「工具设置」部分查找 `sensenova-u1-image token`，若已存在则直接使用。
2. **获取 Token**：若未配置，请用户提供其商汤 API Token（格式：`Authorization: Bearer <token>`）。
3. **保存 Token**：将 Token 写入 MEMORY.md 的「工具设置」部分：
   ```
   ### sensenova-u1-image
   - token: <the-token-value>
   ```
4. 确认已保存，后续调用无需重复提供。

### 2. 生成图片

根据用户描述构建请求并执行：

```bash
curl -s --max-time 60 --retry 3 --retry-delay 5 https://token.sensenova.cn/v1/images/generations \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "sensenova-u1-fast",
    "prompt": "你的图片描述",
    "size": "2048x2048",
    "n": 1
  }'
```

### 3. 下载并展示

```bash
curl -s -o output.png "<image_url>"
```

将图片展示给用户或发送给用户。

---

## API 参考

### 端点

```
POST https://token.sensenova.cn/v1/images/generations
```

### 请求头

```
Authorization: Bearer {token}
Content-Type: application/json
```

### 请求参数

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `model` | string | 是 | 固定值 `sensenova-u1-fast` |
| `prompt` | string | 是 | 图片描述文字，最大 4096 token |
| `size` | string | 否 | 图片尺寸，默认 `2048x2048`（见下方尺寸表） |
| `n` | integer | 否 | 生成图片数量，默认 1 |

### 尺寸与比例对照表

| 用途建议 | 尺寸 | 比例 |
|----------|------|------|
| 人像 / 竖版海报 | `1664x2496` | 2:3 |
| 风景 / 横版展示 | `2496x1664` | 3:2 |
| 手机壁纸 / 竖版插画 | `1760x2368` | 3:4 |
| 横版插画 / 课件配图 | `2368x1760` | 4:3 |
| 社交媒体封面 | `1824x2272` | 4:5 |
| 社交媒体封面（横） | `2272x1824` | 5:4 |
| **头像 / 图标 / 通用** | **`2048x2048`** | **1:1** |
| **桌面壁纸 / 宽屏展示** | **`2752x1536`** | **16:9** |
| 手机全屏壁纸 | `1536x2752` | 9:16 |
| 超宽横幅 / 电影感 | `3072x1376` | 21:9 |
| 超长竖幅 / 条幅 | `1344x3136` | 9:21 |

> 用户未指定尺寸时，默认使用 `2048x2048`（1:1 方图）。

---

## 响应格式

### 成功响应

```json
{
  "created": 1700000000,
  "data": [
    {
      "url": "https://..."
    }
  ]
}
```

提取图片 URL：`response.data[0].url`

### 生成多张图片（n > 1）

```json
{
  "data": [
    { "url": "https://...image1" },
    { "url": "https://...image2" }
  ]
}
```

遍历 `response.data` 获取所有图片 URL。

---

## 错误处理

### 重试策略

- 网络超时或连接失败时，**自动重试最多 3 次**，每次间隔 5 秒。
- curl 命令中已包含 `--retry 3 --retry-delay 5 --max-time 60` 参数。

### 常见错误码

| 错误码 | 含义 | 处理方式 |
|--------|------|----------|
| `401` / `Unauthorized` | Token 无效或过期 | 提醒用户检查 Token 是否正确，必要时重新配置 |
| `429` / `RateLimitExceeded` | 请求频率超限 | 等待 10 秒后重试，或提醒用户稍后再试 |
| `400` / `BadRequest` | 请求参数错误 | 检查 prompt 是否为空、size 是否在支持列表中、n 是否为正整数 |
| `500` / `InternalError` | 服务端错误 | 等待后重试；若持续出现，提醒用户商汤服务可能暂时不可用 |
| 网络超时 / 无响应 | 网络问题 | 检查网络连接，重试；若持续失败，提醒用户检查网络环境 |

### 错误处理流程

1. 执行 curl 命令后，首先检查 HTTP 状态码。
2. 若状态码非 `200`，解析响应体中的错误信息，按上表处理。
3. 对于可重试错误（429、500、网络超时），自动等待后重试，最多 3 次。
4. 重试仍失败时，向用户展示**友好的中文错误提示**，包含：
   - 错误原因（用通俗语言描述）
   - 建议的解决方式
   - 原始错误信息（供排查参考）

---

## 使用场景示例

### 示例 1：生成头像

> 用户说：「帮我生成一个赛博朋克风格的头像」

- 尺寸选择：`2048x2048`（1:1，适合头像）
- prompt：`赛博朋克风格的头像，霓虹灯光，未来感，数字艺术`

### 示例 2：生成桌面壁纸

> 用户说：「画一张山间日出的风景壁纸」

- 尺寸选择：`2752x1536`（16:9，适合桌面壁纸）
- prompt：`山间日出的壮丽风景，金色阳光穿透云层，远山层叠，高清摄影风格`

### 示例 3：生成手机壁纸

> 用户说：「做一个可爱的猫咪手机壁纸」

- 尺寸选择：`1536x2752`（9:16，适合手机竖屏）
- prompt：`可爱的橘猫趴在窗台上，阳光洒落，温馨治愈风格，插画`

### 示例 4：生成社交媒体配图

> 用户说：「帮我做一张公众号文章封面图，主题是 AI 教育」

- 尺寸选择：`2272x1824`（5:4，适合公众号封面）
- prompt：`AI 教育主题封面，机器人与孩子一起学习，科技感与温暖并存，扁平化插画风格`

### 示例 5：批量生成

> 用户说：「给我生成 3 张不同风格的花卉图」

- 尺寸选择：`2048x2048`
- n：3
- prompt：`精美的花卉特写，不同品种和风格，高清摄影`

---

## 常见问题（FAQ）

**Q：Token 从哪里获取？**
A：前往商汤 SenseNova 官网（https://platform.sensenova.cn）注册账号，在控制台创建 API Key 即可获取。

**Q：生成的图片链接有效期多久？**
A：API 返回的图片 URL 通常有时效限制，建议生成后**立即下载保存**到本地，不要依赖 URL 长期访问。

**Q：支持哪些语言的 prompt？**
A：支持中文和英文，建议使用**英文 prompt** 获得最佳效果，中文 prompt 同样可用但细节表现可能略有差异。

**Q：一次最多生成几张图片？**
A：通过 `n` 参数控制，建议单次不超过 4 张，数量越多等待时间越长。

**Q：生成一张图大概需要多久？**
A：通常 5-15 秒，复杂 prompt 或多张图片可能需要更长时间。curl 已设置 60 秒超时。

**Q：prompt 有什么技巧？**
A：描述越具体越好。推荐结构：`主体 + 场景/背景 + 风格 + 细节修饰`。例如「一只橘猫坐在书桌上，窗外的阳光，水彩画风格，温暖色调」比「猫」效果好得多。

---

## 注意事项

- Token 保存后自动复用，用户只需提供一次。
- 尊重用户的语言偏好，用用户使用的语言进行 `prompt` 描述。
- 用户未指定尺寸时，根据用途智能推荐（见尺寸对照表），或默认 `2048x2048`。
- 图片 URL 有时效性，生成后应立即下载保存。
- 遇到错误时，优先自动重试，重试失败后给出友好的中文提示和解决建议。
