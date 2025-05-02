# HTTP Header Propagation in LiteLLM

LiteLLM allows propagating specific HTTP headers from client requests to LLM providers. This is useful for:

- Passing custom tracking identifiers
- Maintaining the context of the original request
- Propagating application-specific metadata

## Configuration

Header propagation is configured at the model level in the `model_list`:

```yaml
model_list:
  - model_name: "gpt-4"
    litellm_params:
      model: "gpt-4"
    model_info:
      # List of HTTP headers to propagate to the LLM provider
      propagate_headers: 
        - "x-my-custom-header"
        - "x-request-id"
        - "x-session-id"
```

## Usage Example

1. Configure the headers to propagate in the `config.yaml`:

```yaml
model_list:
  - model_name: "anthropic-claude-v2"
    litellm_params:
      model: "claude-2"
    model_info:
      propagate_headers:
        - "x-conversation-id" 
        - "x-session-id"
```

2. Al hacer peticiones al proxy, incluir las cabeceras:

```bash
curl http://localhost:8000/v1/chat/completions \
  -H "x-conversation-id: conv_123" \
  -H "x-session-id: sess_456" \
  -H "authorization: Bearer sk-..." \
  -H "content-type: application/json" \
  -d '{
    "model": "anthropic-claude-v2",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

The headers `x-conversation-id` and `x-session-id` will be automatically propagated to the request LiteLLM makes to Anthropic.

## Considerations

- Headers are propagated as-is, preserving their original names and values
- If a configured header is not present in the request, it is silently ignored
- Header propagation is performed transparently, without altering the proxy's behavior

## Common Use Cases

1. **End-to-end Traceability**
   ```yaml
   propagate_headers:
     - "x-request-id"
     - "x-trace-id" 
   ```

2. **Session Context**
   ```yaml
   propagate_headers:
     - "x-session-id"
     - "x-user-id"
   ```

3. **Application Metadata**
   ```yaml
   propagate_headers:
     - "x-app-version"
     - "x-deployment-env"
   ```

## Logs

The proxy logs when headers are propagated:

```
DEBUG:litellm.proxy: Propagating header x-conversation-id to the LLM provider
```

This helps verify that propagation is working correctly.
