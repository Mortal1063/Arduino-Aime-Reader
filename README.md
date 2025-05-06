# Arduino-Aime-Reader
使用 Arduino + PN532 制作的 Aime 兼容读卡器。  

- 支持卡片类型： [FeliCa](https://zh.wikipedia.org/wiki/FeliCa)（Amusement IC、Suica、八达通等）和 [MIFARE](https://zh.wikipedia.org/wiki/MIFARE)（Aime，Banapassport）
- 逻辑实现是通过对官方读卡器串口数据进行分析猜测出来的，并非逆向，不保证正确实现
- 通信数据格式参考了 [Segatools](https://github.com/djhackersdev/segatools) 和官方读卡器抓包数据，可在 [Example.txt](doc/Example.txt) 和 [nfc.txt](https://github.com/djhackersdev/segatools/blob/master/doc/nfc.txt) 查看
- 定制使用例（主要的开发测试环境）：[ESP32-CardReader](https://github.com/Sucareto/ESP32-CardReader) 


### 使用方法
1. 按照 [Aime_Reader_PN532](https://github.com/Sucareto/Aime_Reader_PN532) 的提示安装库 **（必须）**
2. 按照使用方式，在 Arduino 和 PN532 接好连接线（I2C 或 SPI 或 HSU），并调整 PN532 上的拨码开关
3. 接上 WS2812B 灯条（可选，不会影响正常读卡功能）
4. 上传 [ReaderTest](tools/ReaderTest/ReaderTest.ino) 测试硬件是否工作正常
5. 若读卡正常，可按照游戏支持列表打开设备管理器设置 COM 端口号
6. 按照游戏的波特率设置代码的`high_baudrate`选项，`115200`是`true`，`38400`是`false`
7. 如果有使用 [Segatools](https://github.com/djhackersdev/segatools)，参考 [segatools.ini 设置教程](https://github.com/djhackersdev/segatools/blob/master/doc/config/common.md#aime) 关闭 Aime 模拟功能
8. 上传 [Arduino-Aime-Reader](Arduino-Aime-Reader.ino)打开游戏测试

如果需要自定义 Aime 卡，安装 [MifareClassicTool](https://github.com/ikarus23/MifareClassicTool) 或其他同样效果的软件，修改 [Aime 卡示例](doc/aime示例.mct) 后写入空白 MIFARE UID/CUID 卡，即可刷卡使用。  
关于自定义 Aime 卡的写入和读取问题，请参考 [SAK（88->08）](https://github.com/Sucareto/Arduino-Aime-Reader/pull/17) 的讨论。

某些 Arduino 可能需要在游戏主程序连接前，给串口以正确的波特率发送 DTR/RTS，可以先打开一次 Arduino 串口监视器再启动游戏程序。  


### 支持游戏列表
| 代号 | 默认 COM 号 | 支持的卡 | 默认波特率 |
| - | - | - | - |
| SDDT/SDEZ | COM1 | FeliCa,MIFARE | 115200 |
| SDEY | COM2 | MIFARE | 38400 |
| SDHD | COM4 | FeliCa,MIFARE | cvt=38400,sp=115200 |
| SBZV/SDDF | COM10 | FeliCa,MIFARE | 38400 |
| SDBT | COM12 | FeliCa,MIFARE | 38400 |

- 如果读卡器没有正常工作，可以切换波特率试下
- 有使用 amdaemon 的，可以参考 config_common.json 内 aime > unit > port 确认端口号  
如果 `"high_baudrate" : true` 则波特率是`115200`，否则就是`38400`
- 在游戏和服务器支持的情况下，本读卡器程序可正常使用 emoney 端末认证和刷卡支付功能


### 开发板适配情况
| 开发板名 | 主控 | 备注 |
| - | - | - |
| [SparkFun Pro Micro](https://learn.sparkfun.com/tutorials/pro-micro--fio-v3-hookup-guide#hardware-overview-pro-micro) | ATmega32U4 | 需要发送 DTR/RTS |
| [NodeMCU 1.0](https://github.com/nodemcu/nodemcu-devkit-v1.0?tab=readme-ov-file#pin-map) | ESP-12E + CH340 | |
| [NodeMCU-32S](https://docs.ai-thinker.com/esp32/boards/nodemcu_32s) | ESP32-S + CH340 | 主要适配环境 |
| Arduino Uno | ATmega328P + CH340 | 未实际测试，据反馈不可用 |


### 已知问题

- 默认情况下，FeliCa 读写功能处于禁用状态，仅会在 `CMD_CARD_DETECT` 指令中读取 IDm 和 PMm，可通过 `SKIP_FeliCa_THROUGH` 定义控制此行为
- 启用 FeliCa 读写功能后，默认开启 `PN532_FeliCa_THROUGH`，所有 FeliCa 操作命令将直接转发给 PN532 处理。如需自定义读写逻辑，可禁用该定义并修改实现相关函数
- 启用 FeliCa 读写功能后，部分游戏可能不支持所有类型的 FeliCa 卡片，这与官方读卡器的行为一致
- 由于 PN532 库的限制，目前不支持多卡同时识别，仅处理最先识别到的卡片。
- 如果使用不符合 Aime、Banapassport 数据格式的 MIFARE 卡（如某些交通卡或模拟卡），可能会导致游戏状态异常
- 对于未实现的指令，默认返回 `STATUS_INVALID_COMMAND`，这可能导致某些游戏判定读卡器不可用


### 版本更新情况
- [最新](https://github.com/Sucareto/Arduino-Aime-Reader/tree/main)

通过 [QHPaeek/pull/19](https://github.com/Sucareto/Arduino-Aime-Reader/pull/19) 和 [nerimoe/pull/21](https://github.com/Sucareto/Arduino-Aime-Reader/pull/21) 实现了 FeliCa 的正确读写、固件更新处理。  
因未持有官方读卡器和框体等环境，无法进行更多测试，如没有 bug 或者新文档，将不会再更新。

- [v2.0](https://github.com/Sucareto/Arduino-Aime-Reader/commits/v2.0)

参考 [AiMeLib/AiMeNFCRW.h]() 重写命令标记和函数名，引入 status 定义表，并未在官方设备上确认效果。

- [v1.0](https://github.com/Sucareto/Arduino-Aime-Reader/commits/v1.0)

参考 [segatools](https://github.com/djhackersdev/segatools/blob/master/board/sg-nfc-cmd.h) 编写了基本实现；  
分析了官方读卡器串口数据，通过猜测数据结构完成了大部分功能。


### 参考
- 驱动 WS2812B：[FastLED](https://github.com/FastLED/FastLED)
- 驱动 PN532：[PN532](https://github.com/elechouse/PN532)
- 读取 FeliCa 参考：[PN532を使ってArduinoでFeliCa学生証を読む方法](https://qiita.com/gpioblink/items/91597a5275862f7ffb3c)
- 读取 FeliCa 数据的程序：[NFC TagInfo](https://play.google.com/store/apps/details?id=at.mroland.android.apps.nfctaginfo)，[NFC TagInfo by NXP](https://play.google.com/store/apps/details?id=com.nxp.taginfolite)
- MIFARE 读写卡，数据分析：[MifareClassicTool](https://github.com/ikarus23/MifareClassicTool)，[MifareOneTool](https://github.com/xcicode/MifareOneTool)
