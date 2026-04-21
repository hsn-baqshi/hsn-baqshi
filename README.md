- 👋 Hi, I’m @hsn-baqshi. I am an engineer, artist and developer
- 👀 I’m interested in Game Development, Data Science, video games, cars, and sciences in general
- 🌱 I’m currently working on an indie game 
- 📫 You can reach me @ albaqshiha@gmail.com

<!---
hsn-baqshi/hsn-baqshi is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

litellm.ContextWindowExceededError: litellm.BadRequestError: ContextWindowExceededError: Hosted_vllmException - {"error":{"message":"This model's maximum context length is 131072 tokens. However, your request has 199948 input tokens. Please reduce the length of the input messages. (parameter=input_tokens, value=199948)","type":"BadRequestError","param":"input_tokens","code":400}}
model=glm-4.7. context_window_fallbacks=None. fallbacks=[{'glm-4.7': ['gpt-oss-120b']}].

Set 'context_window_fallback' - https://docs.litellm.ai/docs/routing#fallbacks. Received Model Group=glm-4.7
Available Model Group Fallbacks=['gpt-oss-120b']
Error doing the fallback: litellm.BadRequestError: Hosted_vllmException - {"error":{"message":"max_tokens must be at least 1, got -161706. (parameter=max_tokens, value=-161706)","type":"BadRequestError","param":"max_tokens","code":400}}No fallback model group found for original model_group=gpt-oss-120b. Fallbacks=[{'glm-4.7': ['gpt-oss-120b']}]. Received Model Group=gpt-oss-120b
Available Model Group Fallbacks=None
Error doing the fallback: litellm.BadRequestError: Hosted_vllmException - {"error":{"message":"max_tokens must be at least 1, got -161706. (parameter=max_tokens, value=-161706)","type":"BadRequestError","param":"max_tokens","code":400}}No fallback model group found for original model_group=gpt-oss-120b. Fallbacks=[{'glm-4.7': ['gpt-oss-120b']}]

how can i tackle this issue with openwebui
