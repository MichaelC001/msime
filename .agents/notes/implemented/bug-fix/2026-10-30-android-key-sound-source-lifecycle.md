# Agent Note: Android 按键音效缓存绑定来源目录

Status: implemented

## Problem

`ImeKeyFeedback` 原先只按 `plugins.key_sound.pack` 的 ID 判断缓存是否可复用。输入框切换会更换 `preferencesDirectory`，同一音效包 ID 可能由新目录提供不同样本，但旧 `SoundPool` 仍会被保留；加载期间播放路径也只比较 ID，因此会继续播放旧会话的音效。

## Decision

按键音效缓存身份同时包含包 ID 和解析请求使用的状态目录。`PackSounds` 保存来源目录，偏好刷新时计算当前来源；只有包 ID 与来源都相同才复用或播放。加载中的请求也绑定这两个字段，旧目录的回调只释放结果，不替换当前缓存。

## Alternatives considered

- 只在输入会话切换时清空 `SoundPool`：可以避免复用旧样本，但会让同一目录的会话无谓地重新加载，而且仍需处理切换期间迟到的回调。
- 只比较音效包 ID 的文件修改时间：状态目录不同或两个包同时使用相同时间戳时仍可能误认，不能表达请求实际解析的来源。

## Consequences

切换到不同偏好目录时，加载完成前按键音回落到系统音效；新包加载后再使用 `SoundPool`。同一目录和包 ID 的会话继续复用现有缓存。新增纯 JVM 策略冒烟覆盖同包同目录、换目录、换包、缺失状态和缺失来源。
