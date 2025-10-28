# Error 405 no body when using Agents

> check if you are using any proxy for your LLM

> Check if th e LLM is a OpenAi compatible

In our case we used a proxy that did not implement the openAI responses API, but we had web search in the agent enabled.
Also some Poxies like LiteLLM dont support temeratires other than 1.0 and we had a follow up eror that we could fix.

> Both errors can be prevented by configuring theagent 1; with web search disabled and 2; setting temperature to 1.0
