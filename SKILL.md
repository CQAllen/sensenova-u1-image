---
name: sensenova-u1-image
description: Generate images via SenseNova U1 model using curl to call the text-to-image API. Trigger when the user mentions SenseNova, U1, text-to-image generation, or wants to create images from text descriptions using this specific model.
---

# SenseNova U1 Image Generation

Call SenseNova's U1 model API via curl to generate images from text prompts.

## First-Time Setup

1. **Check existing token**: Look in MEMORY.md for `sensenova-u1-image token`. If found, use it directly.
2. **If no token**: Ask the user to provide their SenseNova API token (`Authorization: Bearer <token>`).
3. **Store it**: Write the token to MEMORY.md :
   ```
   ### sensenova-u1-image
   - token: <the-token-value>
   ```
4. Confirm to the user that the token has been saved.

## Usage Workflow

1. Understand the user's image description and desired parameters (prompt, size, image count).
2. Construct the curl command using the stored token.
3. Execute the command and capture the response.
4. Parse the response to extract image URL(s).
5. Download the generated image(s) to a local path using `browser_use` or `curl`.
6. Show the image(s) to the user via `view_image` or `send_file_to_user`.

## API Reference

### Endpoint

```
POST https://token.sensenova.cn/v1/images/generations
```

### Headers

```
Authorization: Bearer {token}
Content-Type: application/json
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `model` | string | yes | `sensenova-u1-fast` |
| `prompt` | string | yes | Text prompt, max-token=4096 |
| `size` | string | no | Image size with aspect ratio. Options:<br>`1664x2496` ｜ 2:3<br>`2496x1664` ｜ 3:2<br>`1760x2368` ｜ 3:4<br>`2368x1760` ｜ 4:3<br>`1824x2272` ｜ 4:5<br>`2272x1824` ｜ 5:4<br>`2048x2048` ｜ 1:1<br>`2752x1536` ｜ 16:9<br>`1536x2752` ｜ 9:16<br>`3072x1376` ｜ 21:9<br>`1344x3136` ｜ 9:21 |
| `n` | integer | no | Number of images to generate (default 1) |

### Example curl Command

```bash
curl -s https://token.sensenova.cn/v1/images/generations \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "sensenova-u1-fast",
    "prompt": "A cute cat wearing a hat, digital art style",
    "size": "2752x1536",
    "n": 1
  }'
```

### Response Format

The API returns JSON. Extract the image URL(s) from the response (typically under `data[].url` or similar path — parse dynamically).

### Download Image

```bash
curl -s -o output.png "<image_url>"
```

## Notes

- The token is reused once saved — the user only provides it once.
- If the API returns an error, show the full error message to the user for debugging.
- Respect the user's prompt language — use their language for the `prompt` field.
