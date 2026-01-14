---
title: karabiner-elements配置Caps Lock键
date: 2025-10-30
tags:
  - 代码片段
---

## 功能说明

这个 Karabiner-Elements 配置将 Caps Lock 键改造成一个多功能键：

- **单独按下**：发送 F18 键（可用于触发其他快捷键或应用）
- **按住不放**：变成 Hyper 键（同时按下 Right Option + Right Command + Right Control）

## 使用场景

1. **F18 作为触发键**：由于 F18 是一个很少被使用的功能键，可以将其绑定到 Alfred、Raycast 等启动器，或者用于切换输入法
2. **Hyper 键组合**：Hyper 键是一个几乎不会与任何现有快捷键冲突的修饰键组合，非常适合自定义全局快捷键

## 配置详解

| 参数 | 说明 |
|------|------|
| `lazy: true` | 延迟发送修饰键，只有在按下其他键时才真正触发 |
| `hold_down_milliseconds: 100` | F18 按键保持 100 毫秒，确保被正确识别 |
| `modifiers.optional: ["any"]` | 允许与其他修饰键组合使用 |

## 代码

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