#  AI Tab

In the AI tab, you can connect Typemill to any OpenAI-compatible AI provider or to Anthropic (Claude), choose a default model, and configure the API settings for [Kixote](/admin-guide/kixote).

![Screenshot of the AI tab of Typemill](media/live/typemill-ai-tab.webp){.center loading="lazy" width="820" height="512"}

| Feature | Description | 
|:---|:---|
| AI adapter | Select **OpenAI-compatible** for OpenAI, Ollama, LM Studio, OpenRouter, and most other providers, or **Anthropic** for Claude. | 
| API base URL | Enter the base URL of your AI provider, for example `https://api.openai.com/v1`, `http://localhost:11434/v1`, or `https://api.anthropic.com/v1`. Do not add a trailing slash. | 
| Model | Set the default model name as expected by your provider, such as `gpt-4.1`, `llama3`, `mistral`, or `claude-sonnet-4`. | 
| API key | Add your API key if your provider requires authentication. Leave this empty for local providers like Ollama or LM Studio. | 
| Provider name | Set a human-readable name for your provider, such as “OpenAI” or “My Ollama Server”. This is shown to users on the agreement screen. | 
| Terms & Conditions URL | Optional link to the provider’s terms and conditions. | 
| Temperature | Set the default temperature from `0.1` (strict) to `1.0` (creative). Default: `0.7`. | 
| Maximum output tokens | Define the maximum output length. Default: `4000`. | 
| AI request timeout | Select a timeout for long articles. Default: 120 | 
| Reasoning effort | For thinking/reasoning models only (e.g. Ollama, OpenAI o1/o3). 'Default' lets the provider decide. 'Off' disables reasoning. Other levels control reasoning depth. |

## Bring Your Own AI

Connect Typemill to any OpenAI-compatible AI provider (OpenAI, Ollama, LM Studio, OpenRouter, ...) or to Anthropic (Claude). For local providers like Ollama, no API key is needed.

If you use a cloud provider, user inputs will be sent to that external service. Make sure this complies with your local data privacy regulations. Each user will be asked for confirmation before using AI features. Read more in the documentation.

Within the Kixote interface, users can also select another model from the provider and overwrite the default model.

