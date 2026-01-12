---
title: LiteLLM Proxy Integration
seoTitle: Open Source Observability for LiteLLM Proxy
sidebarTitle: LiteLLM Proxy
logo: /images/integrations/litellm_icon.png
---

# LiteLLM Proxy Integration

In this guide, we will show you how to use the LiteLLM Proxy to capture LLM calls and log them to Langfuse.

> **What is LiteLLM?** [LiteLLM](https://github.com/BerriAI/litellm) is an open-source proxy and SDK that provides a single unified API to call and manage hundreds of different LLM providers and models with OpenAI-compatible endpoints.

> **What is Langfuse?** [Langfuse](https://langfuse.com) is an open-source LLM observability platform that helps you trace, monitor, and debug your LLM applications.

---

There are three ways to integrate LiteLLM with Langfuse:

1. Sending logs via the LiteLLM Proxy to capture all LLM calls going through the proxy.
2. Using the LiteLLM SDK to capture LLM calls directly.
3. Using any compatible framework (such as the OpenAI or LangChain SDK) to capture LLM calls.

<Callout type="info">
  This integration is for the LiteLLM Proxy. If you are looking for the LiteLLM
  SDK integration, see the [LiteLLM SDK
  Integration](/integrations/frameworks/litellm-sdk) page.
</Callout>
---

## LiteLLM Proxy

<Tabs items={["Local", "LiteLLM UI"]}>
<Tab>

Add the integration to your proxy configuration:

**1. Add the credentials to your environment variables**

```bash
export LANGFUSE_PUBLIC_KEY="pk-lf-..."
export LANGFUSE_SECRET_KEY="sk-lf-..."
export LANGFUSE_OTEL_HOST="https://us.cloud.langfuse.com"  # Default US region
# export LANGFUSE_OTEL_HOST="https://otel.my-langfuse.company.com"  # custom OTEL endpoint
```

**2. Setup litellm_config.yaml**

```yaml litellm_config.yaml
litellm_settings:
  callbacks: ["langfuse_otel"]
```

**3. Start the proxy**

```bash
litellm --config /path/to/litellm_config.yaml
```

**4. Use the proxy to log traces to Langfuse**

```bash
curl -X POST "http://0.0.0.0:4000/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-xxxx" \
  -d '{
    "model": "gpt-3.5-turbo",
    "messages": [
      {"role": "system", "content": "You are a very accurate calculator. You output only the result of the calculation."},
      {"role": "user", "content": "1 + 1 = "}
    ]
  }'
```

**5. See the LiteLLM generations in Langfuse**

<Frame className="rounded-lg overflow-hidden">
  ![LiteLLM Proxy
  Trace](/images/cookbook/integration_litellm/litellm-proxy-trace.png)
</Frame>

[Example trace in Langfuse](https://cloud.langfuse.com/project/cloramnkj0002jz088vzn1ja4/traces/9a9de57946027786cdda14480860ebf3?timestamp=2025-09-30T16%3A36%3A58.741Z&observation=7df5f09bb6a6bea4)

You can find detailed information on how to use the LiteLLM Proxy in the [LiteLLM docs](https://docs.litellm.ai/docs/observability/langfuse_otel_integration).

</Tab>
<Tab>

By setting the callback to Langfuse in the LiteLLM UI you can instantly log your responses across all providers. For more information on how to setup the Proxy UI, see the [LiteLLM docs](https://docs.litellm.ai/docs/proxy/ui).

<Callout type="info">

You can add additional Langfuse attributes to the requests in order to group requests into traces, add userIds, tags, sessionIds, and more. These attributes are shared across LiteLLM Proxy and SDK, please refer to both documentation pages to learn about all potential options:

- [LiteLLM Proxy docs](https://docs.litellm.ai/docs/proxy/logging#langfuse)
- [LiteLLM Python SDK docs](https://litellm.vercel.app/docs/observability/langfuse_integration#advanced)

</Callout>

<Frame>
  ![Set Langfuse as callback in Proxy
  UI](https://langfuse.com/images/docs/litellm-ui.png)
</Frame>

</Tab>
</Tabs>



## Learn more about LiteLLM

### What is LiteLLM?

[LiteLLM](https://litellm.ai) is an open source proxy server to manage auth, loadbalancing, and spend tracking across more than 100 LLMs. LiteLLM has grown to be a popular utility for developers working with LLMs and is universally thought to be a useful abstraction.

### Is LiteLLM an Open Source project?

Yes, LiteLLM is open source. The majority of its code is permissively MIT-licensed. You can find the open source LiteLLM repository on [GitHub](https://github.com/BerriAI/litellm).

### Can I use LiteLLM with Ollama and local models?

Yes, you can use LiteLLM with Ollama and other local models. LiteLLM supports all models from Ollama, and it provides a Docker image for an OpenAI API-compatible server for local LLMs like llama2, mistral, and codellama.

### How does LiteLLM simplify API calls across multiple LLM providers?

LiteLLM provides a unified interface for calling models such as OpenAI, Anthropic, Cohere, Ollama and others. This means you can call any supported model using a consistent method, such as `completion(model, messages)`, and expect a uniform response format. The library does away with the need for if/else statements or provider-specific code, making it easier to manage and debug LLM interactions in your application.



## GitHub Discussions

import { GhDiscussionsPreview } from "@/components/gh-discussions/GhDiscussionsPreview";

<GhDiscussionsPreview labels={["integration-litellm"]} />

