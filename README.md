

Based on the provided code map and SKILL.md content, I can see this is a project for generating images using the SenseNova U1 model via API calls. Here's a comprehensive README:

# sensenova-u1-image

SenseNova U1 文生图模型 API 调用工具

## 项目简介

本项目提供通过 curl 调用商汤科技 SenseNova U1 文生图 API 的方法，支持将文本描述转换为图像。

## 首次配置

在使用本项目之前，您需要：

1. 获取 SenseNova API 密钥
2. 配置 API 端点凭证

## 使用流程

1. 准备文本提示词
2. 调用 API 生成图像
3. 获取返回的图像 URL
4. 下载并保存图像

## API 参考

### 接口地址

`https://api.sensenova.cn/v1/text-to-image`

### 请求头

- `Authorization`: Bearer 您的API密钥
- `Content-Type`: application/json

### 请求体

```json
{
  "prompt": "your text description",
  "parameters": {
    // 可选参数
  }
}
```

### curl 示例

```bash
curl -X POST "https://api.sensenova.cn/v1/text-to-image" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "your text description"}'
```

### 响应格式

API 返回图像 URL，可通过该 URL 下载生成的图像。

## 注意事项

- 请确保您的 API 密钥安全，不要公开泄露
- 遵守商汤科技 API 使用条款
- 图像生成时间可能因服务器负载而异
