# ESP32 Traffic Forwarder

ESP32 流量转发工具，烧录向导适用于 Windows 64 位。 [下载 Windows 烧录包](../../releases)

**需自备一块 ESP32-S3 N16R8 最小系统板。** 目标模组为乐鑫 **ESP32-S3-WROOM-1-N16R8**，配有 16 MB Quad SPI Flash 和 8 MB Octal SPI PSRAM。购买或烧录前可核对模组金属屏蔽罩上的型号。 [乐鑫模组规格书](https://documentation.espressif.com/esp32-s3-wroom-1_wroom-1u_datasheet_en.pdf)

解压整个 ZIP，用可传数据的 USB 线连接板子，双击 **ESP32TrafficForwarder.exe**。烧录和日常使用都无需操作板载屏幕。

本软件许可限非商业使用。使用前请阅读 [非商业使用协议](NON-COMMERCIAL-TERMS.md) 和 [免责声明](DISCLAIMER.md)。

## 1. 手机看 → 手机/电脑玩

1. **观看方手机**开启移动数据和 **2.4 GHz 个人热点**。在烧录向导选「手机看 → 手机/电脑玩」，再选「另一部手机」或「电脑」。按向导填写观看方手机已有的热点名称和密码。烧录后，ESP32 会连接这部手机的热点上网。
2. **玩游戏的另一部手机或电脑**连接 ESP32 发出的 Wi-Fi `ESP32-Forwarder`，使用向导里给 ESP32 新设的密码。选了电脑就同时按下方步骤禁用以太网、断开其他 Wi-Fi。
3. **观看方手机**保持移动数据和热点开启，在 App 里选择「ESP32 网关」，填入向导报告的地址和 `app-pairing.txt` 里的配对码。

## 2. 电脑玩 → 手机看

1. **观看方手机**开启移动数据和 **2.4 GHz 个人热点**。在烧录向导直接选「电脑玩 → 手机看」，填写观看方手机已有的热点名称和密码。烧录后，ESP32 会连接这部手机的热点上网。
2. **玩游戏的电脑**连接 ESP32 发出的 Wi-Fi `ESP32-Forwarder`，使用向导里给 ESP32 新设的密码。按下方步骤禁用以太网、断开其他 Wi-Fi，再打开游戏。
3. **观看方手机**保持移动数据和热点开启，在 App 里选择「ESP32 网关」，填入向导报告的地址和 `app-pairing.txt` 里的配对码。

## Windows 电脑只通过 ESP32 上网

按 **Win + R**，输入 `ncpa.cpl` 并回车，右键正在使用的「以太网」适配器，选择「禁用」。网线可以保持插着。断开电脑的其他 Wi-Fi 连接，只保留 `ESP32-Forwarder`。用完后，可以在同一窗口右键选择「启用」，恢复以太网连接。

## 串口与配对

- 串口由 Windows 分配，换电脑或 USB 口时可能会变。向导只识别到一个乐鑫 USB 串口时会自动选择。出现串口列表时，选开发板对应的端口。烧录会替换板子上的应用程序。
- 同一台电脑用本工具重刷同一块板时，会沿用这块板的配对码。沿用同一个观看方热点时，切换电脑或手机玩，只需让选好的游戏设备连接 `ESP32-Forwarder`。
- App 里填写的是 ESP32 从观看方手机热点取得的地址。观看方手机保持自己的热点开启，在 App 内连接 ESP32。
- 向导如果暂未读到地址，确认观看方手机的移动数据和热点均已开启。用 115200 波特率查看串口输出的 `PAIRING_READY uplink=…`，其中的地址用于 App 配对。

## 命令行部署

下列命令可以由脚本或 AI 在解压目录执行。`--self-check` 校验包内固件，`--list-ports` 列出串口。

```powershell
.\ESP32TrafficForwarder.exe --self-check
.\ESP32TrafficForwarder.exe --list-ports
```

向使用者取得本机热点设置，将下例的三个占位值替换后，保存为 UTF-8 的 `wifi.local.json`。`mode` 的值 `pc` 表示电脑玩，`phone` 表示另一部手机玩。两种模式的观看端都是开热点的手机。两组密码分别用于观看手机热点和 ESP32 热点，允许 8–63 个可打印 ASCII 字符。配置文件和配对码均为私有数据。

```json
{
  "mode": "pc",
  "ssid": "<观看手机的热点名>",
  "hotspot_password": "<观看手机的热点密码>",
  "esp32_password": "<设给 ESP32-Forwarder 的密码>"
}
```

把 `COM4` 换成实际串口。`--yes` 确认替换该板应用固件。若工具只能识别到一个乐鑫 USB 串口，可省略 `--port`。

```powershell
.\ESP32TrafficForwarder.exe --port COM4 --settings .\wifi.local.json --yes --report .\deploy-result.json
```

命令退出状态为 `0` 且生成以下报告时，固件已写入设备。

- `status: "ready"` 表示串口已报告网关就绪。`address` 是手机 App 要填写的地址，`pairing_file` 指向配对码文件。
- `status: "flashed_waiting_for_hotspot"` 表示固件已写入，向导等待期间尚未收到热点地址。检查热点后从串口读 `PAIRING_READY uplink=…`。
- `status: "started"` 只表示已尝试烧录。命令非零退出时查看错误输出。遇到串口占用，先关闭串口监视器。板子未进入下载模式时，按住 BOOT 并短按 RESET，再松开 BOOT 后重试。

部署工具使用独立的乐鑫 esptool v5.6.0。第三方许可证及来源见 `THIRD-PARTY-LICENSES.txt`。 [乐鑫 esptool v5.6.0](https://github.com/espressif/esptool/releases/tag/v5.6.0)
