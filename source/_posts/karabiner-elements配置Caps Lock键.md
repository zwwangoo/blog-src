---
title: karabiner-elements配置Caps Lock键
date: 2025-10-30
tags:
  - 代码片段
---


```
{
    "description": "caps_lock -> f18(alone) | Hyper(held)",
    "manipulators": [
        {
            "from": {
                "key_code": "caps_lock",
                "modifiers": { "optional": ["any"] }
            },
            "to": [
                {
                    "key_code": "right_option",
                    "lazy": true,
                    "modifiers": ["right_command", "right_control"]
                }
            ],
            "to_if_alone": [
                {
                    "hold_down_milliseconds": 100,
                    "key_code": "f18"
                }
            ],
            "type": "basic"
        }
    ]
}
```