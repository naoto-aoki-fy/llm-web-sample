# OpenAI-Compatible API Chat Client

A minimal, single-file browser client for testing OpenAI-compatible chat completion APIs.

This client connects directly from the browser to endpoints compatible with:

- `GET /v1/models`
- `POST /v1/chat/completions`

It is intended for quick local testing, lightweight demos, and checking whether an OpenAI-compatible API server supports model listing and streaming chat completions.

## Features

- Single HTML file
- No build step
- No external JavaScript dependencies
- Model list loading from `/v1/models`
- Streaming chat response support via Server-Sent Events style responses
- Optional `temperature` and `max_tokens` parameters
- API endpoint persistence
- API key encryption using the browser Web Crypto API
- Light and dark color schemes using `prefers-color-scheme`
- Solid background styling without gradients

## Requirements

You need a browser that supports:

- `fetch`
- `ReadableStream`
- `TextDecoder`
- `localStorage`
- Web Crypto API, including `crypto.subtle`

The target API server must support browser access, including appropriate CORS headers.

## Usage

1. Open the HTML file in a modern browser.
2. Enter the API endpoint URL.

   Example:

   ```text
   https://api.example.com
   ```

   or:

   ```text
   https://api.example.com/v1
   ```

   The client automatically normalizes the base URL to use `/v1`.

3. Enter an API key if required by your API server.
4. Click **Save Settings**.
5. Click **Load Models** to retrieve available models.
6. Select a model.
7. Enter a prompt.
8. Click **Send**.

## Saved Settings

The endpoint URL is saved in `localStorage`.

The API key is not saved directly in `localStorage`. Instead, the client uses the following approach:

1. A random 256-bit local secret is generated if one does not already exist.
2. The local secret is saved in `localStorage`.
3. The API key is encrypted with AES-GCM using the local secret.
4. The encrypted API key is stored in the URL fragment as:

   ```text
   index.html#?key={escaped_encrypted_key}
   ```

5. When the page is opened with this URL format, the client decrypts the key and pre-fills the API key form field.

## API Key Encryption: Security Trade-Off

The API key encryption used by this client is a compromise.

It avoids storing the API key itself as plaintext in `localStorage`, but it is not equivalent to production-grade secret management. The encryption key is still stored in the same browser environment, so any script or attacker that can access the page's origin and `localStorage` may be able to decrypt the API key.

This design is useful for reducing accidental exposure from simple `localStorage` inspection, but it does not protect against:

- Cross-site scripting on the same origin
- Malicious browser extensions
- Compromised browsers
- Users sharing the full URL, including the fragment
- Screenshots or logs that include the URL
- Other users with access to the same browser profile

The URL fragment is not sent to the server as part of normal HTTP requests, but it can still be visible in the browser address bar and may be copied or shared by the user.

For production use, do not expose API keys directly to frontend code. Use a backend proxy, server-side session, OAuth flow, or another server-side authentication mechanism.

## API Compatibility

The client expects a model list response similar to:

```json
{
  "data": [
    {
      "id": "model-name"
    }
  ]
}
```

For chat completions, the client sends a request similar to:

```json
{
  "model": "selected-model",
  "messages": [
    {
      "role": "user",
      "content": "Your prompt"
    }
  ],
  "stream": true
}
```

Streaming responses are parsed from `data:` lines. The client reads text from:

```js
choices[0].delta.content
```

It also includes a loose fallback for:

```js
choices[0].text
```

## CORS Notes

Because this client calls the API directly from the browser, the API server must allow cross-origin requests.

If requests fail before reaching the API server, check whether the server returns appropriate CORS headers, such as:

```http
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: Authorization, Content-Type
Access-Control-Allow-Methods: GET, POST, OPTIONS
```

The exact configuration depends on your deployment and security requirements.

## Customization

The UI is implemented with CSS custom properties. Light and dark themes are controlled through `prefers-color-scheme`.

You can customize colors by editing the variables in the `:root` block and the `@media (prefers-color-scheme: dark)` block.

## Limitations

This is a minimal testing client. It does not provide:

- Multi-turn conversation history
- System prompt editing
- File upload support
- Tool/function calling support
- Authentication flows beyond bearer tokens
- Production-grade secret management

## License

Use, modify, and redistribute this file as needed for your own projects.
