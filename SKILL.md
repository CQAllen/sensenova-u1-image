---
name: sensenova-u1-image
description: Generate and edit images via SenseNova U1.5 models (商汤日日新) using curl against the official image API. Covers sensenova-u1.5-fast and sensenova-u1.5-lite, text-to-image and image editing with reference images. Trigger when the user mentions 商汤, SenseNova, U1, U1.5, 文生图, 图生图, 图片编辑, 改图, or wants to create/modify images from text or reference images using this model family.
---

# SenseNova U1.5 图片生成与编辑

通过商汤 SenseNova U1.5 模型 API，将文字描述生成图片，或基于参考图编辑图片。

> 模型家族：`sensenova-u1.5-fast`（加速版，默认）与 `sensenova-u1.5-lite`（高质量版）。
> Base URL：`https://token.sensenova.cn/v1`
> 官方文档：https://platform.sensenova.cn/docs

**U1 系列只能用于同步图片生成 / 同步图片编辑，不能作为对话模型调用**（不要走 `/v1/chat/completions`）。

---

## 模型选型

| 模型 | model_id | 何时使用 |
|------|----------|----------|
| **U1.5 Fast** | `sensenova-u1.5-fast` | 默认选择。生成与编辑效率更高，兼顾质量与响应速度，适合快速创作迭代 |
| **U1.5 Lite** | `sensenova-u1.5-lite` | 需要最高画面质量、参考图创作、复杂指令与多重约束时 |

两者接口、参数完全一致，只需替换 `model` 字段。

> **向后兼容**：旧模型 ID `sensenova-u1-fast` 实测**仍然可用**（返回 200 正常出图），并不会返回 404。因此既有的 U1 集成不会立即失效，但建议迁移到 `sensenova-u1.5-*`。

---

## 快速开始

### 1. 配置 Token

首次使用需要 API Token：

1. **检查已有 Token**：在 MEMORY.md 的「工具设置」部分查找 `sensenova-u1-image token`，若已存在则直接使用。
2. **获取 Token**：若未配置，请用户提供其商汤 API Key（在 [SenseNova 控制台](https://platform.sensenova.cn/console/keys) 创建）。
3. **保存 Token**：写入 MEMORY.md 的「工具设置」部分：
   ```
   ### sensenova-u1-image
   - token: <the-token-value>
   ```
4. 确认已保存，后续调用无需重复提供。

### 2. 文生图

```bash
curl -s --max-time 180 --retry 3 --retry-delay 5 \
  https://token.sensenova.cn/v1/images/generations \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "sensenova-u1.5-fast",
    "prompt": "你的图片描述",
    "size": "2048x2048",
    "n": 1,
    "watermark": false,
    "response_format": "url",
    "output_format": "png"
  }'
```

两个参数必须显式传入，原因见下方「默认值陷阱」。

### 3. 下载并展示

响应里取 `data[0].url`，立即下载（URL 24 小时后失效）：

```bash
curl -s -o output.png "<image_url>"
```

将图片展示给用户或发送给用户。

---

## API 参考

### 端点

| 用途 | 端点 |
|------|------|
| 文生图（仅文本 prompt） | `POST https://token.sensenova.cn/v1/images/generations` |
| 图片编辑（参考图 + 编辑指令） | `POST https://token.sensenova.cn/v1/images/edits` |

### 请求头

```
Authorization: Bearer {token}
Content-Type: application/json
```

### 文生图请求参数

| 字段 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `model` | string | 是 | — | `sensenova-u1.5-fast` 或 `sensenova-u1.5-lite` |
| `prompt` | string | 是 | — | 图像生成描述 |
| `size` | string | 否 | `auto` | 图像尺寸，格式 `WIDTHxHEIGHT`（如 `1024x1024`），规则见下 |
| `n` | integer | 否 | `1` | 生成数量，**仅支持 `1`** |
| `watermark` | boolean | 否 | `true` | `true` 添加日日新官方 Logo 水印；`false` 无水印纯图 |
| `response_format` | string | 否 | `b64_json` | `b64_json` 返回 Base64；`url` 返回 24 小时有效的临时下载链接 |
| `output_format` | string | 否 | `png` | 图片文件格式：`png`、`jpeg`、`webp`（不影响 Base64/URL 的返回方式） |
| `prompt_extend` | boolean | 否 | `true` | 提示词自动润色扩写；扩写失败时自动回退到原始 prompt |

### 尺寸规则

- 格式 `WIDTHxHEIGHT`，小写 `x`、无空格
- 宽和高三者均需满足：**是 32 的倍数**、**最小 512**、**最大 4096**、**长宽比不超过 3:1**（或 1:3）
- 不传或传 `auto` 时由模型自动选择

**官方推荐分辨率**

| 用途 | 尺寸 | 比例 | 分辨率 |
|------|------|------|--------|
| 人像 / 竖版海报 | `1664x2496` | 2:3 | 2K |
| 风景 / 横版展示 | `2496x1664` | 3:2 | 2K |
| 手机壁纸 / 竖版插画 | `1536x2720` | 9:16 | 2K |
| 头像 / 图标 / 通用 | `2048x2048` | 1:1 | 2K |
| 桌面壁纸 / 宽屏展示 | `2720x1536` | 16:9 | 2K |
| 最高画质 | `4096x4096` | 1:1 | 4K |

**其他常用比例**（均已验证为 32 的倍数且满足约束）

| 用途 | 尺寸 | 比例 |
|------|------|------|
| 横版插画 / 课件配图 | `1920x1440` | 4:3 |
| 竖版插画 | `1440x1920` | 3:4 |
| 社交媒体封面（竖） | `1664x2048` | 4:5 |
| 社交媒体封面（横） | `2048x1664` | 5:4 |
| 超宽横幅 / 电影感 | `2688x1280` | 21:9 |
| 超长竖幅 / 条幅 | `1056x2432` | 9:21 |

> 用户未指定尺寸时，按用途从上表推荐；无法判断时默认 `2048x2048`。
> 自定义尺寸务必同时校验「32 的倍数 / 512–4096 / 比例 ≤ 3:1」三条，否则会被 400 拒绝。

### 图片编辑请求参数（`/v1/images/edits`）

| 字段 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `model` | string | 是 | — | `sensenova-u1.5-fast` 或 `sensenova-u1.5-lite` |
| `images` | array | 是 | — | 图片对象数组，每项含 `image_url`；**第 1 张为主编辑图**；**至多 5 张**参考图 |
| `images[].image_url` | string | 是 | — | 公网 URL（http/https）或 Base64 Data URL |
| `prompt` | string | 是 | — | 编辑指令，描述期望最终画面；去除首尾空格后不可为空；尽量保留未指定修改的主体元素 |
| `n` | integer | 否 | `1` | 生成数量，**仅支持 `1`** |
| `size` | string | 否 | `auto` | 同上规则；`auto` 时自动适配主图 |
| `response_format` | string | 否 | `b64_json` | `b64_json` 或 `url`（24 小时有效） |
| `output_format` | string | 否 | `png` | `png`、`jpg`、`jpeg`、`webp` |
| `watermark` | boolean | 否 | `true` | 是否添加官方 Logo 水印 |
| `prompt_extend` | boolean | 否 | `true` | 提示词自动润色扩写 |

参考图输入两种写法：

```json
// 公网 URL
"images": [{ "image_url": "https://example.com/photo.png" }]

// 本地文件 -> Base64 Data URL（必须带完整前缀）
"images": [{ "image_url": "data:image/png;base64,iVBORw0KGgo..." }]
```

> 必须包含完整的 `data:image/{format};base64,` 前缀，**不支持直接传纯 Base64 字符串**。
> 图片链接不可访问、内容不是有效图片、或 Base64 解码失败，请求会被直接拒绝。

---

## 响应格式

### 文生图成功响应

```json
{
  "created": 1788849614,
  "data": [
    { "url": "https://cdn.sensenova.dev/gen/..." }
  ],
  "output_format": "png",
  "size": "1024x1024",
  "usage": {
    "input_tokens": 1540,
    "input_tokens_details": { "image_tokens": 0, "text_tokens": 1540 },
    "output_tokens": 4096,
    "total_tokens": 5636,
    "images_count": 1
  }
}
```

- `response_format=url` → `data[].url`；`response_format=b64_json` → `data[].b64_json`
- 两者互斥，不会同时返回；同一次请求中的所有图片返回格式一致
- 需要 Base64 落地时：`jq -r '.data[0].b64_json' resp.json | base64 -d > output.png`

### 图片编辑成功响应

结构相同，`data[0]` 依 `response_format` 返回 `url` 或 `b64_json`，`usage.input_tokens_details.image_tokens` 记录参考图占用的 token。

---

## 默认值陷阱（务必显式传参）

| 参数 | 官方默认 | 建议传值 | 原因 |
|------|----------|----------|------|
| `response_format` | `b64_json` | `url` | 默认返回 Base64 而非 URL，直接取 `data[0].url` 会拿到空值 |
| `watermark` | `true` | `false` | 默认加 Logo 水印；无水印目前公测免费，但官方提示后续可能转付费，显式传参可避免未来默认值变更影响使用 |
| `n` | `1` | `1` | 只支持 1，传其他值会被拒绝 |

---

## 多张图片

`n` 只支持 `1`，需要 N 张时循环调用 N 次：

```bash
for i in 1 2 3; do
  curl -s https://token.sensenova.cn/v1/images/generations \
    -H "Authorization: Bearer YOUR_TOKEN" \
    -H "Content-Type: application/json" \
    -d '{
      "model": "sensenova-u1.5-fast",
      "prompt": "精美的花卉特写，高清摄影",
      "size": "2048x2048",
      "n": 1,
      "watermark": false,
      "response_format": "url"
    }' | jq -r ".data[0].url" > "url_$i.txt"
done
```

---

## 错误处理

### 重试策略

- 网络超时、429、500 时自动重试，最多 3 次；429 采用指数退避。
- curl 已包含 `--retry 3 --retry-delay 5`；`--max-time 180`（4K 大图与 Lite 模型耗时更长）。

### 错误响应结构

文档定义的错误信封（`code` 为字符串、带 `type`）：

```json
{
  "error": {
    "type": "invalid_request_error",
    "code": "3",
    "message": "invalid temperature, should in [0,2]."
  }
}
```

**实测鉴权类错误是另一种形状**：`code` 为数字、**没有 `type` 字段**（2026-09-20 实测）：

```json
// 未带 Authorization 头 -> HTTP 401
{ "error": { "code": 16, "message": "Authorization Not Found" } }

// Token 无效 -> HTTP 401
{ "error": { "code": 16, "message": "Forbidden" } }
```

> 解析错误时以 `error.message` 为准，`type` 可能缺失，不要依赖 `error.code` 是字符串。

### 错误码对照

| HTTP | `error.type` | 含义 | 处理方式 |
|------|--------------|------|----------|
| `400` | `invalid_request_error` | 请求参数错误（缺失 / 越界 / 格式错） | 检查 `prompt` 非空、`size` 符合尺寸规则、`n` 为 1、枚举值合法 |
| `400` | `failed_precondition_error` | 前置条件失败（编码失败、引擎不可用等） | 检查 Base64 Data URL 是否完整；稍后重试 |
| `401` | —（`code: 16`） | Token 缺失（`Authorization Not Found`）或无效（`Forbidden`） | 提示用户检查 / 重新提供 API Key |
| `403` | `permission_denied_error` | 当前语言权限不足或被策略拦截 | 检查账号是否有该模型权限、内容是否违规 |
| `404` | `not_found_error` | 模型 ID 未知或已下线（message: `model is not found`，`code: "5"`） | 确认 model 是 `sensenova-u1.5-fast` / `sensenova-u1.5-lite` |

**实测错误信息原文**（可直接用于快速定位问题）：

```text
n 不合法     -> {"error":{"message":"invalid n, should be in [1, 1]",
              "type":"invalid_request_error","code":"3"}}

size 不合法  -> {"error":{"message":"invalid size, should be auto or WIDTHxHEIGHT.
              width and height should be multiples of 32, width and height
              should be in [512,4096], and aspect ratio should be within 3:1",
              "type":"invalid_request_error","code":"3"}}

模型不存在   -> {"error":{"message":"model is not found",
              "type":"not_found_error","code":"5"}}
```
| `408` | `canceled_error` | 客户端取消了请求 | 检查本地超时设置是否过短 |
| `429` | `quota_exceeded_error` | 额度或频率超限 | 指数退避后重试，或提醒用户稍后再试 |
| `500` | `internal_server_error` | 服务端内部错误 | 等待后重试；持续出现则提示商汤服务可能暂不可用 |
| 网络超时 / 无响应 | — | 网络问题 | 检查网络；重试仍失败则提示用户检查网络环境 |

### 错误处理流程

1. 执行 curl 后先检查 HTTP 状态码。
2. 非 `200` 时解析响应体的 `error.message`（`error.type` 可能缺失），按上表处理。
3. 对可重试错误（429、500、网络超时）自动等待后重试，最多 3 次。
4. 重试仍失败时，给出**友好的中文错误提示**，包含：错误原因（通俗描述）、建议解决方式、原始 `error.message`（供排查）。

---

## 使用场景示例

### 示例 1：生成头像（默认 Fast）

> 用户说：「帮我生成一个赛博朋克风格的头像」

- `model`: `sensenova-u1.5-fast`，`size`: `2048x2048`（1:1）
- `prompt`: `赛博朋克风格的头像，霓虹灯光，未来感，数字艺术`

### 示例 2：桌面壁纸

> 用户说：「画一张山间日出的风景壁纸」

- `size`: `2720x1536`（16:9，2K）
- `prompt`: `山间日出的壮丽风景，金色阳光穿透云层，远山层叠，高清摄影风格`

### 示例 3：手机壁纸

> 用户说：「做一个可爱的猫咪手机壁纸」

- `size`: `1536x2720`（9:16，2K）
- `prompt`: `可爱的橘猫趴在窗台上，阳光洒落，温馨治愈风格，插画`

### 示例 4：高质量海报（Lite + 4K）

> 用户说：「做一张电影质感的竖版海报」

- `model`: `sensenova-u1.5-lite`，`size`: `4096x4096`（4K）
- `prompt`: `电影质感竖版海报，戏剧性光影，胶片颗粒，宽幅构图`

### 示例 5：改图（基于参考图编辑）

> 用户说：「把这张图换成雪景背景」（用户提供了一张图）

```bash
curl -s https://token.sensenova.cn/v1/images/edits \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "sensenova-u1.5-lite",
    "images": [{ "image_url": "https://.../原图.png" }],
    "prompt": "将背景替换为广阔冰川，保持主体人物不变",
    "size": "auto",
    "watermark": false,
    "response_format": "url"
  }'
```

本地文件需先转 Base64 Data URL：`data:image/png;base64,$(base64 -w0 原图.png)`

### 示例 6：批量生成

> 用户说：「给我生成 3 张不同风格的花卉图」

- `n` 只支持 1，按「多张图片」小节的循环方式调用 3 次，每次换 prompt 变体以获得不同风格

---

## 常见问题（FAQ）

**Q：Token 从哪里获取？**
A：前往 [SenseNova 平台](https://platform.sensenova.cn) 注册账号，在控制台（`/console/keys`）创建 API Key。

**Q：图片链接有效期多久？**
A：`response_format=url` 返回的是**临时链接，固定有效期 24 小时**，超时后直接失效、无法再次访问。生成后应立即下载保存；需要长期留存用 `b64_json`。

**Q：会加水印吗？**
A：默认 `watermark=true` 会加日日新官方 Logo 水印。无水印（`watermark=false`）当前公测免费，官方提示后续可能转为付费功能，因此建议每次请求都显式传 `watermark`。

**Q：一次最多生成几张？**
A：`n` **只支持 `1`**，多于 1 会被拒绝。需要多张请循环调用。

**Q：能一次传多张参考图吗？**
A：图片编辑接口最多支持 **5 张**参考图，第 1 张作为主编辑图。

**Q：`prompt_extend` 要开还是关？**
A：默认 `true` 会自动润色扩写 prompt，通常效果更好；扩写失败会回退原始 prompt。需要严格保留原句表述（如精确排版文字）时设为 `false`。

**Q：prompt 有什么技巧？**
A：描述越具体越好。推荐结构：`主体 + 场景/背景 + 风格 + 细节修饰`。例如「一只橘猫坐在书桌上，窗外的阳光，水彩画风格，温暖色调」远好于「猫」。

---

## 实测校验（2026-09-20，真实调用验证）

| 验证项 | 结果 |
|--------|------|
| `sensenova-u1.5-fast` 2048x2048 出图 | ✅ HTTP 200，15.1s，落地为真 2048×2048 PNG（4.6MB） |
| `sensenova-u1.5-lite` 出图 | ✅ HTTP 200，14.1s |
| `watermark:false` | ✅ 四角干净，无水印 |
| 省略 `watermark`（默认 true） | ✅ 右下角出现「日日新 SenseNova」Logo |
| 省略 `response_format`（默认 b64_json） | ✅ `data[0]` 只有 `b64_json`，**完全没有 `url` 键** |
| `response_format:url` | ✅ 返回签名 URL，带 `X-Amz-Expires=86400`（正好 24 小时） |
| `output_format` 生效 | ✅ 传 `webp` 则回显 `"output_format":"webp"` 且返回 webp |
| `/v1/images/edits` 参考图编辑 | ✅ HTTP 200，19.8s，`image_tokens:4096` |
| 编辑接口 `size:auto` | ✅ 自动适配主图，回显主图尺寸 |
| `n=2` | ✅ HTTP 400 `invalid n, should be in [1, 1]` |
| `size` 违反 3:1 / 非 32 倍数 | ✅ HTTP 400，错误信息逐字复述尺寸规则 |
| 未知模型 ID | ✅ HTTP 404 `model is not found` |
| 旧 ID `sensenova-u1-fast` | ⚠️ **仍返回 200 正常出图**，未下线 |
| 响应结构 | ✅ 与文档完全一致（`created` / `data` / `output_format` / `size` / `usage`） |

**耗时参考**（单图，官方同步接口）

| 场景 | 耗时 |
|------|------|
| Fast 512x512 | ~7s |
| Fast 2048x2048 | ~15s |
| Fast 图片编辑 2048x2048 | ~20s |
| Lite 512x512 | ~14s |

> 4K 分辨率与 Lite 模型更慢，`--max-time 180` 留有余量。Lite 在小尺寸下并不比 Fast 快，Lite 的优势在画质而非速度。

---

## 注意事项

- Token 保存后自动复用，用户只需提供一次。
- 尊重用户的语言偏好，用用户使用的语言撰写 `prompt`。
- 每次请求显式传 `watermark` 与 `response_format`，避免官方默认值变更带来意外。
- 用户未指定尺寸时按用途从推荐表选择；自定义尺寸须校验 32 倍数 / 512–4096 / 比例 ≤ 3:1。
- 图片 URL 24 小时失效，生成后立即下载保存。
- 需要最高画质或参考图创作时用 `sensenova-u1.5-lite`，日常快速出图用 `sensenova-u1.5-fast`。
- 遇到错误时优先自动重试，重试失败后给出友好的中文提示与解决建议。
