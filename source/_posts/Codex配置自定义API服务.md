---
title: Codex配置自定义API服务
date: 2026-03-25
tags:
  - 工具
---
### **1、安装Codex**

- 命令行使用：

```
npm install -g @openai/codex
```

- 使用图形化界面：[Codex.dmg](https://chatgpt.com/codex?utm_source=google&utm_medium=paid_search&utm_campaign=GOOG_X_SEM_GBR_Codex_CDX_BAU_ACQ_PER_DMA_ALL_NAMER_US_EN_031826&c_id=23655882150&c_agid=197896593561&c_crid=800673030479&c_kwid=kwd-111182835&c_ims=&c_pms=1013802&c_nw=g&c_dvc=c&gad_source=1&gad_campaignid=23655882150)
- VSCode中插件使用：
![Pasted image 20260325222309](/blog-img/Pasted%20image%2020260325222309.png)

### **2、配置文件**

编辑文件 `~/.codex/config.toml`

```toml
model_provider = "mycpa"
model = "gpt-5.4"
model_reasoning_effort = "high"
disable_response_storage = true

# mycpa
[model_providers.mycpa]
name = "mycpa"
base_url = "http://192.168.1.239:8317/v1"  # 这里需要更改
wire_api = "responses"
requires_openai_auth = true
```

编辑文件 `~/.codex/auth.json`

```json
{
  "OPENAI_API_KEY": "替换为API Key"
}
```

使用：

![Pasted image 20260325222646](/blog-img/Pasted%20image%2020260325222646.png)