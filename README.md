# OpenAI-Compatible API Chat Client

A minimal, single-file browser client for testing OpenAI-compatible chat completion APIs.

This client connects directly from the browser to endpoints compatible with:

* `GET /v1/models`
* `POST /v1/chat/completions`

It is intended for quick local testing, lightweight demos, and checking whether an OpenAI-compatible API server supports model listing and streaming chat completions.

## Features

* Single HTML file
* No build step
* No external JavaScript dependencies
* Model list loading from `/v1/models`
* Streaming chat response support via Server-Sent Events style responses
* Optional `temperature` and `max_tokens` parameters
* API settings saved in `localStorage`
* Light and dark color schemes using `prefers-color-scheme`
* Solid background styling without gradients

## Requirements

You need a browser that supports:

* `fetch`
* `ReadableStream`
* `TextDecoder`
* `localStorage`

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

## Security Notes

This client stores the API endpoint and API key in the browser's `localStorage`.

Do not use it on:

* Shared computers
* Public terminals
* Untrusted browsers
* Hosted pages that other users can access

For production use, avoid exposing API keys directly in frontend code. Use a backend proxy or another server-side authentication flow instead.

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

* Multi-turn conversation history
* System prompt editing
* File upload support
* Tool/function calling support
* Authentication flows beyond bearer tokens
* Production-grade secret management

## License

Use, modify, and redistribute this file as needed for your own projects.
