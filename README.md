# StableDiff-Edge: Globally Distributed AI Image Gateway



A free, serverless image generation API powered by **Cloudflare Workers AI**. Generate stunning images from text prompts using Stable Diffusion models.

## ✨ Features

- 🚀 **Serverless** - No infrastructure to manage
- 🆓 **Free Tier** - Cloudflare Workers AI free tier included
- 🔐 **API Key Protection** - Secure your endpoint
- ⚡ **Fast** - Edge-deployed globally
- 🔌 **n8n Compatible** - Easy integration with automation workflows

---

## 🛠️ Cloudflare Setup

### Step 1: Create a Worker

1. Go to [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. Navigate to **Compute (AI)** → **Workers & Pages**
3. Click **Create Worker**
4. Name your worker (e.g., `free-image-api`)
5. Click **Deploy** (we'll add code later)

### Step 2: Add API Key

1. Go to your worker's **Settings** tab
2. Scroll to **Variables and Secrets**
3. Click **Add**
4. Add the following:

| Type | Name | Value |
|------|------|-------|
| Secret | `API_KEY` | Your custom API key (e.g., `mySecretKey123`) |

5. Click **Save**

### Step 3: Add AI Binding

1. In **Settings**, scroll to **Bindings**
2. Click **Add Binding**
3. Select **Workers AI**
4. Set the variable name to: `AI`
5. Click **Save**

### Step 4: Deploy the Code

1. Go to your worker and click **Edit Code**
2. Delete the existing code
3. Paste the following code:

```javascript
export default {
  async fetch(request, env) {
    const API_KEY = env.API_KEY;
    const url = new URL(request.url);
    const auth = request.headers.get("Authorization");

    // 🔐 Simple API key check
    if (auth !== `Bearer ${API_KEY}`) {
      return json({ error: "Unauthorized" }, 401);
    }

    // 🚫 Only allow POST requests to /
    if (request.method !== "POST" || url.pathname !== "/") {
      return json({ error: "Not allowed" }, 405);
    }

    try {
      const { prompt } = await request.json();

      if (!prompt) return json({ error: "Prompt is required" }, 400);

      // 🧠 Generate image from prompt
      const result = await env.AI.run(
        "@cf/stabilityai/stable-diffusion-xl-base-1.0",
        { prompt }
      );

      return new Response(result, {
        headers: { "Content-Type": "image/jpeg" },
      });
    } catch (err) {
      return json(
        { error: "Failed to generate image", details: err.message },
        500
      );
    }
  },
};

// 📦 Function to return JSON responses
function json(data, status = 200) {
  return new Response(JSON.stringify(data), {
    status,
    headers: { "Content-Type": "application/json" },
  });
}
```

4. Click **Deploy** ✅

---

## 📡 API Usage

### Endpoint

```
POST https://your-worker-name.your-subdomain.workers.dev/
```

### Headers

| Header | Value |
|--------|-------|
| `Authorization` | `Bearer YOUR_API_KEY` |
| `Content-Type` | `application/json` |

### Request Body

```json
{
  "prompt": "A monkey riding a bike"
}
```

### Response

Returns a **JPEG image** directly (binary data).

### cURL Example (Windows PowerShell)

```powershell
curl.exe -X POST "https://your-worker.workers.dev/" `
  -H "Authorization: Bearer YOUR_API_KEY" `
  -H "Content-Type: application/json" `
  -d '{"prompt": "A sunset over mountains"}' `
  --output image.jpg
```

### cURL Example (Linux/Mac)

```bash
curl -X POST "https://your-worker.workers.dev/" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "A sunset over mountains"}' \
  --output image.jpg
```

---

## 🔗 n8n Integration

Use the **HTTP Request** node to generate images in your n8n workflows.

### HTTP Request Node Configuration

| Setting | Value |
|---------|-------|
| **Method** | `POST` |
| **URL** | `https://your-worker.workers.dev/` |
| **Authentication** | Generic Credential Type → Header Auth |
| **Send Body** | ✅ ON |
| **Body Content Type** | JSON |
| **Response Format** | File |

### Authentication Setup

1. In n8n, go to **Credentials** → **Add Credential**
2. Select **Header Auth**
3. Configure:
   - **Name**: `Authorization`
   - **Value**: `Bearer YOUR_API_KEY`

### Body Configuration

**Option 1: Static Prompt**
```json
{
  "prompt": "A futuristic city at night"
}
```

**Option 2: Dynamic Prompt (from previous node)**
```json
{
  "prompt": "{{ $json.imagePrompt }}"
}
```

### Important Settings

Under **Options**:
- Set **Response Format** to `File` to receive the image as binary data
- This allows you to pipe the image to other nodes (Google Drive, S3, Email, etc.)

---

## 🎨 Available Models

You can change the model in the worker code. Available options:

| Model | Description |
|-------|-------------|
| `@cf/stabilityai/stable-diffusion-xl-base-1.0` | High quality, general purpose (default) |
| `@cf/bytedance/stable-diffusion-xl-lightning` | Faster generation |
| `@cf/lykon/dreamshaper-8-lcm` | Artistic style |
| `@cf/blackforestlabs/flux-1-schnell` | Fast, high quality |

---

## 📝 Error Responses

| Status | Error | Description |
|--------|-------|-------------|
| 401 | `Unauthorized` | Invalid or missing API key |
| 400 | `Prompt is required` | Missing prompt in request body |
| 405 | `Not allowed` | Wrong HTTP method or path |
| 500 | `Failed to generate image` | AI generation error |


---

## ☕ Support

If this project helped you, consider buying me a coffee!

[![Buy Me A Coffee](https://img.shields.io/badge/Buy%20Me%20A%20Coffee-☕-FFDD00?style=flat-square&logo=buymeacoffee&logoColor=black)](https://buymeacoffee.com/mehaksandhudev)

---
---

## 📄 License

MIT License - Feel free to use and modify!

---

Made with ❤️ using Cloudflare Workers AI
