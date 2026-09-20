# sensenova-u1-image

调用商汤 SenseNova **U1.5** 图片模型的 Claude Code Skill —— 文生图与基于参考图的图片编辑。

## 项目简介

本仓库是一个 Claude Code Skill（见 [SKILL.md](./SKILL.md)），封装了通过 `curl` 调用商汤官方图片 API 的完整流程：请求构造、参数校验、响应解析、多图循环、错误重试与友好提示。

- 模型：`sensenova-u1.5-fast`（加速版，默认）、`sensenova-u1.5-lite`（高质量版 / 参考图创作）
- 能力：文生图 `/v1/images/generations`、图片编辑 `/v1/images/edits`
- 官方文档：<https://platform.sensenova.cn/docs>

## 首次配置

1. 前往 [SenseNova 平台](https://platform.sensenova.cn) 注册账号。
2. 在控制台（`/console/keys`）创建 API Key。
3. 首次调用 skill 时把 Key 提供给 Claude，它会记录到 MEMORY.md，后续调用无需重复提供。

## 使用流程

1. 描述想要的画面（或直接给一张参考图）。
2. Skill 按用途自动选择模型与尺寸，构造并发送请求。
3. 解析响应中的 `data[0].url`（24 小时有效）并下载图片。
4. 把图片展示给用户。

## 快速体验

```bash
curl -s --max-time 180 \
  https://token.sensenova.cn/v1/images/generations \
  -H "Authorization: Bearer $SENSENOVA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "sensenova-u1.5-fast",
    "prompt": "A fluffy white baby seal floating on a calm sea, in a photorealistic style",
    "size": "2048x2048",
    "n": 1,
    "watermark": false,
    "response_format": "url",
    "output_format": "png"
  }'
```

> 注意 `response_format` 官方默认是 `b64_json`，想要 URL 必须显式传 `"url"`；`watermark` 默认为 `true`，想要无水印需显式传 `false`。完整参数说明见 [SKILL.md](./SKILL.md)。

## 安装到 Claude Code

```bash
cp -r . ~/.claude/skills/sensenova-u1-image
```

重启 Claude Code 后，提到「商汤」「U1.5」「文生图」「改图」等关键词即可触发。

## 目录结构

```
sensenova-u1-image/
├── README.md    # 本文件，项目说明
└── SKILL.md     # Skill 定义与完整 API 参考
```

## 注意事项

- 妥善保管 API Key，不要公开提交到版本库。
- 遵守商汤科技 API 使用条款。
- 图片链接 24 小时后失效，请及时下载保存。
- 图片生成耗时随分辨率、模型（Lite 慢于 Fast）与服务负载变化，建议预留充足超时时间。
