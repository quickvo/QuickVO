# 接入方迁移：远端播放音量（1.8.10-1）

三人同时说话时某一路几乎听不见，原因之一是 SDK 把每路远端 `RTCAudioSource.volume` 写成了 **gain 5**。WebRTC 的 0–10 是增益，`1.0` 才是原声。两路 5 倍叠进 mixer 会顶满限幅器，弱的一路被压没。

**1.8.10-1 起默认 unity（1.0），房间音量会粘在后续挂上的轨上。接入方必须改传值，不能再传 `5`。**

## 合同

| 接入方传入 | SDK 行为 |
|---|---|
| `0...1` | 用户音量。**`1` = 原声** |
| `(1, 2]` | 显式微抬 |
| `> 2`（旧的 0–10，典型是 `5`） | **丢弃，写成 1.0**，日志 `[av] playback_gain_legacy` |
| 未调用 | 远端默认 **1.0** |

- 只作用于远端播放。本地采集轨不再写 `source.volume`。
- `setRoomAudioVolume` 记在房间上：晚进房 / 重订 / 重挂都继承。
- `RTCParticipant.volume` 覆盖该人全部远端音频轨。房间级再设会清掉覆盖。

## 请怎么改

```swift
// 推荐：什么都不调，默认就是原声。
// 若有通话音量滑条，范围必须是 0...1：
room.setRoomAudioVolume(sliderValue)

participant.volume = 0.8   // 单人覆盖，同样是 0...1
```

删除：

```swift
room.setRoomAudioVolume(5)
participant.volume = 5
room.setRoomAudioVolume(100)
```

升级后 1:1 会比以前明显轻（以前默认 5 倍）。用系统音量，不要把默认增益加回去。

群通话：两人同时说，弱的一路应仍能听见。日志应有 `[av] playback_gain applied gain=1.0`。若仍有 `playback_gain_legacy value=5`，说明还在传旧值。

```swift
.package(url: "https://github.com/quickvo/QuickVO.git", from: "1.8.10-1")
```
