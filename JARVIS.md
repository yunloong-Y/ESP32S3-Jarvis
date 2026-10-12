# ESP32S3-Jarvis：立创实战派开发路线

本仓库是 [78/xiaozhi-esp32](https://github.com/78/xiaozhi-esp32) 的公开 fork，保留上游完整源码和许可证。当前自有功能是 `docs/` 下的 Web Serial USB 烧录工具；尚未发布自行编译的“贾维斯”固件。不会改动小智后台的智能体、设备或 OTA 配置。

## 板型与基线

根据板子照片，目标是立创实战派小智 AI 智能体：ESP32-S3-WROOM-1 **N16R8**（16 MB Flash、8 MB PSRAM），2.0 英寸 240×320 电容触屏、GC0308 摄像头、ES8311 扬声器音频、ES7210 麦克风 ADC，USB 串口含 CH343。芯片 `revision v0.2` 只能说明芯片修订，不能决定板型固件。

上游板级目录：[`main/boards/lckfb/szpi-esp32s3`](main/boards/lckfb/szpi-esp32s3)。编译配置：`IDF_TARGET=esp32s3`，板型 `lckfb-lichuang-dev`，启用板上回声消除。以 [小智 v2.5.2 发布页](https://github.com/78/xiaozhi-esp32/releases/tag/v2.5.2) 的 `v2.5.2_lckfb-lichuang-dev.zip` 为起点；官方包内 `merged-binary.bin` 的 SHA-256 为 `bbc8a4f222eddec8322649852f9aad929c62f28420483e30a637e6d978660886`。

## 可直接使用的 USB 烧录工具

公开 HTTPS 页面：[贾维斯 USB 烧录台](https://yunloong-y.github.io/ESP32S3-Jarvis/)（GitHub Pages 发布成功后可用）。在电脑 Chrome/Edge 中打开，连接板子，选择上述 ZIP 解压的 `merged-binary.bin`。工具先确认 ESP32-S3 和 16 MB Flash，对比分区表，要求下载前 64 KB 备份，并再次确认板型。分区表完全一致时跳过 NVS 的 `0x9000–0xCFFF`；不同则要求明确同意全量烧录。网页仅通过本机 Web Serial 和串口通信，不调用小智后台接口。

**边界：** 备份包含本地 Wi-Fi 等敏感信息，不要上传或公开。烧录器没有实体板接入测试；第一次刷写请连接稳定电源，并准备好官方恢复镜像。保留 NVS 也不等于保证所有本地数据都不变。烧录后若设备不能正常启动，用官方烧录工具和原版镜像恢复。不要把整块 Flash 的备份提交到公开仓库。

## “贾维斯”产品方向

| 方向 | 利用这块板的什么能力 | 优先级 |
| --- | --- | --- |
| 桌面语音中枢 | 双麦/回声消除、扬声器、触屏 | 第一阶段：状态屏、唤醒、音量与静音 |
| 可见的助手 | GC0308 摄像头、屏幕 | 第二阶段：按键拍照识物，显示引用与置信提示 |
| 家居场景控制 | 小智已有 MCP/物联网能力 | 第三阶段：设备查询、明确确认后执行控制 |
| 桌面工作流 | 触屏卡片、MCP 工具 | 第三阶段：计时、待办、天气等可审计动作 |
| 外接机器人 | GH1.25 扩展与 I²C | 后续：舵机或底座，加入运动限位与急停 |

优先把“贾维斯”做成**这块板的本地交互层**：开机视觉、状态卡片、按键/触控、声音反馈和明确的执行确认。云端人格、模型、知识库、OTA 等仍由用户在现有小智后台自行管理。新增设备控制应复用小智现有协议和权限，不在固件中写死后台密钥。

可参考的小智二创与硬件形态：上游列出的 [M5Stack CoreS3](https://github.com/78/xiaozhi-esp32)、[Magic Button](https://github.com/78/xiaozhi-esp32)、[LILYGO T-Circle-S3](https://github.com/78/xiaozhi-esp32)、[SenseCAP Watcher](https://github.com/78/xiaozhi-esp32) 等，分别提示桌面触屏、随身按钮、圆形表盘、视觉感知的交互方式。它们是灵感，不能直接使用其固件烧录到本板。

## 源码开发顺序

1. 用官方固件验证摄像头、触屏、麦克风、扬声器和原有后台连接；保存当前设备信息和备份。
2. 在 `main/boards/lckfb/szpi-esp32s3` 中保留引脚、电源、显示和音频配置，先添加独立的 Jarvis UI 与按键交互。
3. 按 [上游自定义板型说明](https://github.com/78/xiaozhi-esp32/blob/main/docs/custom-board.md) 设置独立的板型标识，避免后续 OTA 把定制固件误替换。
4. 在 ESP-IDF 6.1.0 环境编译并检查 16 MB 分区与产物；先用串口日志和功能清单验收，再考虑发布新的自编译镜像。

目前仓库只包含上游源码和 Web 烧录器，**没有可声明为已验证的定制固件**。
