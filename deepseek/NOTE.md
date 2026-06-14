# DeepSeek: No System Prompt Injection

The local DeepSeek filter proxy (`deepseek-proxy.py`) strips `role:system`
messages before forwarding to the DeepSeek API. This is intentional -- DeepSeek's
API does not support Anthropic-format system messages.

## Status

System prompt injection is **not possible** for `claude-deepseek` / `aicc ds`.

## Workaround (if needed)

Embed instructions in the first user message instead of the system prompt.
