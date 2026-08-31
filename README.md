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
* Chat, history, and configuration modes that keep each workflow focused
* Model list loading from `/v1/models`
* Free-form model entry before explicitly loading the available model list
* Streaming chat response support via Server-Sent Events style responses
* One-click copying of the LLM response to the clipboard
* Local chat history for reviewing previously submitted user prompts and LLM responses
* Separate live display for streamed model thinking/reasoning
* Named system prompt saving and recall through `localStorage`
* Three selectable system-prompt delivery methods: system message, separate user message, or combined user prompt
* Optional `temperature` and `max_tokens` parameters
* Arbitrary provider-specific request parameters supplied as a JSON object
* API settings automatically saved in `localStorage` as you edit them
* The selected model and additional request parameters restored on the next visit
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

2. Select **Config**, then enter the API endpoint URL.

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

4. The API settings are saved automatically as you edit them. Clear both fields to
   remove the saved API settings.

5. Enter a model ID directly, or click **Load Models** to retrieve the available models
   and change the model field into a dropdown.

   The selected model is remembered in `localStorage` and selected by default the next
   time it appears in a loaded model list.

6. Optionally enter a **System Prompt Title** and **System Prompt**. The title is for
   display in the browser only and is never included in a request to the LLM. The
   current title and prompt are recalled on the next visit.

   Click **Save System Prompt** to add the prompt to the **Saved System Prompts** list.
   Saving with an existing title updates that saved prompt. To recall one, select its
   title and click **Use Selected Prompt**. Saved prompts are stored in `localStorage`.
   If the system prompt is blank, the client sends no system-prompt content.

   Choose one of the three **System Prompt Input Method** radio buttons:

   * **Send as a system message** sends the system prompt with the `system` role and
     the entered prompt with the `user` role. This is the default.
   * **Send as a separate user message** sends the system prompt with the `user` role,
     followed by the entered prompt as a second `user` message.
   * **Combine with the user prompt** sends a single user message in this format:

   ```text
   {system_prompt}

   ----

   {user_prompt}
   ```

   The selected method is also recalled on the next visit. Existing saved combined-prompt
   preferences are migrated automatically.

7. In **Config**, optionally set **Optional Parameters** and enter provider-specific settings in
   **Additional request parameters (JSON)**. For example:

   ```json
   {
     "reasoning_effort": "none"
   }
   ```

   Or, to disable thinking for providers that support chat template arguments:

   ```json
   {"chat_template_kwargs": {"enable_thinking": false}}
   ```

   This JSON text is saved as you edit it and restored on the next visit.

8. Select **Chat** and enter a user prompt. Leading and trailing whitespace in the user prompt is removed
   before it is sent. For a nonblank system prompt, whitespace is used only to decide
   whether the field is blank; the original system prompt text is sent unchanged.

9. Click **Send**.

10. Click **Copy** beside **Live Response** to copy the complete LLM response to the
    clipboard. The button becomes available as soon as response text is received.

11. Select **History** to review a previous user input and LLM output.
    Completed chats and partial responses from manually stopped chats are saved in
    `localStorage`. Up to 100 of the most recent chats are retained. You can select
    one or more specific chats and delete them together. **Select All** and **Clear
    Selection** controls make bulk selection possible without modifier keys.

The additional parameters input must contain a JSON object. Its fields are passed
through to the destination server unchanged. Support varies between OpenAI-compatible
APIs, so the destination provider may return an error for fields it does not support.

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
      "role": "system",
      "content": "You are a concise assistant."
    },
    {
      "role": "user",
      "content": "Your prompt"
    }
  ],
  "stream": true,
  "temperature": 0.7,
  "max_tokens": 1024,
  "reasoning_effort": "none"
}
```

In this example, `model`, `messages`, and `stream` are generated by the client;
`temperature` and `max_tokens` come from their dedicated inputs; and `reasoning_effort`
comes from the additional parameters JSON object. Additional parameter fields are merged
into the top level of the `/v1/chat/completions` request body without modification.
When the system prompt is empty or contains only whitespace, `messages` contains only
the user message. Otherwise, the selected input method determines whether the system
prompt is sent as a `system` message, as a separate `user` message before the entered
prompt, or combined with the entered prompt in one `user` message. The system prompt's
original whitespace is preserved, while the entered user prompt continues to have
leading and trailing whitespace removed. The combined content is the original system
prompt, two newlines, `----`, two newlines, and the trimmed user prompt.

The keys `model`, `messages`, and `stream` are reserved and cannot be overridden by
the additional JSON. The keys `temperature` and `max_tokens` are also rejected there:
their dedicated inputs are the sole source of truth, avoiding ambiguous conflicts.
The client reports malformed JSON, non-object JSON values (including arrays and
`null`), and reserved-key conflicts before sending a request.

OpenAI-compatible APIs do not all support the same optional parameters. Consult the
destination provider's documentation before adding fields; unsupported fields may
cause that provider to reject the request.

Streaming responses are parsed from `data:` lines. The client reads text from:

```js
choices[0].delta.content
```

It also includes a loose fallback for:

```js
choices[0].text
```

When a compatible provider exposes its reasoning, the client displays it separately
from the final response. It recognizes the commonly used streaming fields
`choices[0].delta.reasoning_content` and `choices[0].delta.reasoning`, plus
`choices[0].reasoning_content` as a fallback. The **Live Thinking** panel appears when
the first reasoning content is received. Providers and non-reasoning models that do
not return one of these fields leave the panel hidden.

## Security Notes

This client stores the API endpoint, API key, selected model, current and saved named
system prompts, system-prompt input method, additional request parameters, and chat
history (user inputs and LLM outputs) in the browser's `localStorage`.

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
* File upload support
* Tool/function calling support
* Authentication flows beyond bearer tokens
* Production-grade secret management

## License

Use, modify, and redistribute this file as needed for your own projects.
