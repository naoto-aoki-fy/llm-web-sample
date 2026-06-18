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
- Streaming chat response support
- Optional `temperature` and `max_tokens` parameters
- Light and dark color schemes using `prefers-color-scheme`
- Solid background styling without gradients
- Encrypted API key storage in `localStorage`
- Secret key stored in the URL fragment as `#?key=...`

## Requirements

You need a browser that supports:

- `fetch`
- `ReadableStream`
- `TextDecoder`
- `localStorage`
- Web Crypto API

The target API server must support browser access, including appropriate CORS headers.

## Usage

1. Open `index.html` in a modern browser.
2. If the URL does not contain a secret key, the client automatically generates one and redirects to a URL like:

   ```text
   index.html#?key=GENERATED_SECRET_KEY
   ```

3. Enter the API endpoint URL.

   Example:

   ```text
   https://api.example.com
   ```

   or:

   ```text
   https://api.example.com/v1
   ```

   The client automatically normalizes the base URL to use `/v1`.

4. Enter an API key if required by your API server.
5. Click **Save Settings**.
6. Click **Load Models** to retrieve available models.
7. Select a model.
8. Enter a prompt.
9. Click **Send**.

## URL Secret Key

The client uses the URL fragment format below:

```text
index.html#?key={escaped_key}
```

When this format is present, the `key` value is used as the local encryption secret.

If no key is present, the client generates a random 32-byte secret key and calls `location.replace()` to update the current URL to the `#?key=...` format.

The URL fragment is not sent to the server as part of normal HTTP requests. However, it can still remain in browser history, bookmarks, screenshots, copied URLs, and local browser state.

Treat the full URL as sensitive.

## API Key Storage

The API key is not stored as plaintext in `localStorage`.

Instead, the client stores an encrypted payload using:

- PBKDF2 with SHA-256
- AES-GCM with a 256-bit key
- A random salt
- A random IV

The encryption key is derived from the secret key in the URL fragment.

## Security Model and Limitations

This encryption is a practical compromise, not a complete security solution.

It protects against casual inspection of `localStorage`, for example when someone opens browser developer tools and looks directly at stored values. It is better than storing the API key as plaintext.

However, it does not provide strong protection against an attacker who can access or execute JavaScript in the same browser context.

In particular, this design does not protect against:

- Malicious or compromised JavaScript served with the page
- Browser extensions with sufficient permissions
- A user or attacker who has the full `index.html#?key=...` URL
- XSS vulnerabilities introduced by future modifications
- Compromised devices
- Shared or public computers
- Browser profiles synchronized to untrusted environments

For production use, do not expose long-lived API keys directly in frontend code. Use a backend proxy, short-lived tokens, server-side authentication, or another controlled credential flow instead.

## CORS Notes

Because this client calls the API directly from the browser, the API server must allow cross-origin requests.

If requests fail before reaching the API server, check whether the server returns appropriate CORS headers, such as:

```http
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: Authorization, Content-Type
Access-Control-Allow-Methods: GET, POST, OPTIONS
```

The exact configuration depends on your deployment and security requirements.

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

## Customization

The UI is implemented with CSS custom properties. Light and dark themes are controlled through `prefers-color-scheme`.

You can customize colors by editing the variables in the `:root` block and the `@media (prefers-color-scheme: dark)` block.

## Limitations

This is a minimal testing client. It does not provide:

- Multi-turn conversation history
- System prompt editing
- File upload support
- Tool/function calling support
- Production-grade secret management
- Protection against malicious scripts running in the same page context

## License

Use, modify, and redistribute this file as needed for your own projects.
