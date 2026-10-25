# 环境与排错

## 先记清楚环境

| 项目 | 当前记录 |
|---|---|
| 开发板 | ESP32-WROOM-32，38Pin；具体厂家和引脚图待补 |
| 主机 | 历史截图是 Windows；系统版本待补 |
| 编辑器 | VS Code + Espressif ESP-IDF 扩展；扩展版本待补 |
| ESP-IDF | 截图显示 v6.1.0；以 `idf.py --version` 输出为准 |
| 目标 | esp32，不能选成 esp32s3、esp32c3 |
| 工程 | 截图为 `E:\ESP_project\first_project`；源码尚未入库 |
| 串口、USB 芯片 | 待设备管理器和板上丝印确认 |
| Python / CMake / Ninja | 在 ESP-IDF 终端记录实际版本 |

先用当前能正常构建的 IDF 版本，不为跟教程随意切换。例程、驱动 API 和文档都跟这个版本走。截图里的版本信息还不是完整的环境验收。

在扩展提供的 ESP-IDF 终端运行，普通终端不一定加载了相同环境：

```text
idf.py --version
python --version
cmake --version
ninja --version
```

Windows 可用 `where.exe python`、`where.exe cmake`、`where.exe idf.py` 查路径。在 PowerShell 用 `$env:IDF_PATH` 查看 IDF 根目录。路径冲突时先重新选择扩展使用的 IDF 环境，不手工往系统 PATH 里堆多个工具链。

## 工程应该从哪里构建

打开含顶层 `CMakeLists.txt` 的工程文件夹，而不是只打开 `main/` 或 `build/`。后续每个实验独立建工程，仓库根目录只是文档入口。

```text
00_hello/
├── CMakeLists.txt
├── main/
│   ├── CMakeLists.txt
│   └── main.c
├── sdkconfig
└── build/                 # 生成目录，不放自己写的源码
```

检查两层 CMake 的分工：顶层按本机同版本官方例程引入 `tools/cmake/project.cmake` 并声明工程；main 组件通过 `idf_component_register` 注册实际存在的源文件、头文件目录和依赖。不要把普通桌面 CMake 项目的写法直接搬过来，也不要用 Code Runner 单独编译 main.c。

入口是 `app_main(void)`。先读懂它与 STM32 裸机 `main + while(1)` 的区别：IDF 已经启动了系统和调度器，app_main 在任务中执行。

## 当前 build/CMake 提示怎么处理

截图中的关键提示是 build 目录不像 CMake 构建目录，工具拒绝自动清理。这说明清理步骤没有识别到有效构建目录；仅凭这一行，还不能判断最早失败在工具链、配置还是目录内容。

1. 先保存完整输出，向上找第一次 configure/build 失败的位置。记录执行了哪个命令，不能只留最后一句 `ninja failed`。
2. 检查当前目录、两份 CMakeLists.txt、main.c 文件名和 IDF 环境。先不要改业务代码或重装所有工具。
3. 查看 build 内是否混入手写文件，保存需要的内容。停止正在进行的构建，把旧 build 改名为 `build_backup_日期`，保留现场。
4. 在工程根目录新建构建结果。首次建工程或目标错误时用 `idf.py set-target esp32`；它会清理构建并重新生成配置，已有工程先保存 sdkconfig。目标正确就不必每次运行它。
5. 运行 `idf.py reconfigure`，成功后运行 `idf.py build`。新的 build 成功后，再决定是否清理旧备份。
6. 若仍失败，记录第一条具体错误和对应文件。`idf.py fullclean` 适用于工具能识别的构建目录，遇到同一拒绝提示时不要反复点 clean。

截图还出现 Windows UTF-8 提示。它和 build 目录提示分开处理：只有日志确实有编码异常时，才尝试在当前 PowerShell 会话设置 `$env:PYTHONUTF8="1"`，或在相应控制台运行 `chcp 65001` 后重试。编码提示本身不能证明 CMake 失败的原因，也不先修改系统全局语言设置。

`stdio.h` 红线也要分开看：如果真实 build 成功，优先检查扩展的配置和 `build/compile_commands.json` 是否生成、编辑器是否指向本工程；如果 build 也失败，先解决编译器输出的错误，不靠关闭红线当作修好。

## 分三步确认成功

在实际工程目录运行；下面 COMx 必须换成设备管理器看到的端口。

```text
idf.py build
idf.py -p COMx flash
idf.py -p COMx monitor
```

| 步骤 | 要保存的证据 | 不能据此推断什么 |
|---|---|---|
| 编译 | 命令成功退出、产物生成 | 还不能说明板子收到固件 |
| 烧录 | 对应端口的写入与校验完成日志 | 还不能说明业务功能正常 |
| 运行 | 复位后出现本次版本标记和持续计数；断电重启仍能复现 | 还不能说明其他外设通过 |

ESP-IDF Size 只是大小统计。空 app_main 没有业务输出很正常；自己补一条启动标记和周期计数来验证，不把旧固件的日志误认成这次运行结果。监视器通常用 Ctrl+] 退出。

烧录连接失败时，依次检查数据线、设备端口、驱动、串口是否被其他工具占用，再按板子说明使用 BOOT/EN。WROOM-32 通常经板载 USB 转串口下载，不能因为有 USB 插座就当作芯片原生 USB。此时不改 CMake。

## 接线先约定

38Pin 是排针数量，不是可任意使用的 GPIO 数量。以下只是各个独立实验的候选 GPIO 编号，接线前核对实物丝印和原理图，不按排针第几脚猜。

| 用途 | 候选引脚 | 注意 |
|---|---|---|
| 外接 LED / PWM | GPIO25，串限流电阻 | 不假定板载 LED 在 GPIO2 |
| 外接按键 | GPIO27，按键接 GND | 配上拉，先轮询去抖 |
| 外设 UART | TX GPIO17、RX GPIO16 | TX/RX 交叉，共地，3.3V TTL |
| ADC | GPIO34，ADC1 | 仅输入；输入范围还受衰减配置限制 |
| I2C | SDA GPIO21、SCL GPIO22 | 确认外部上拉到 3.3V |
| SPI | SCLK18、MOSI23、MISO19、CS26 | 按实际器件模式配置，接线尽量短 |

GPIO6～11 通常连模块 Flash，不作普通实验引脚；GPIO34～39 只能输入且没有内部上下拉；GPIO0/2/5/12/15 涉及启动配置，入门接线先避开；GPIO1/3 通常留给下载和日志。ADC 优先选 ADC1，避免经典 ESP32 上 ADC2 与 Wi-Fi 的使用冲突。

GPIO 不接 5V 信号。LED 串限流电阻；电机、继电器等负载用驱动电路，不直接接 GPIO。两板通信先共地并核对电平，不把两路电源输出随意相连。

## 查资料的入口

先在页面选择与本机一致的 IDF 版本和 esp32 目标；stable 链接以后可能变化。

- [idf.py 与构建命令](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-guides/tools/idf-py.html)
- [GPIO 限制](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/peripherals/gpio.html)
- [ESP32 数据手册](https://www.espressif.com/sites/default/files/documentation/esp32_datasheet_en.pdf)
- [ESP32-WROOM-32 模组手册](https://www.espressif.com/sites/default/files/documentation/esp32-wroom-32_datasheet_en.pdf)
- [I2C 驱动](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/peripherals/i2c.html)
- [IDF FreeRTOS](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/system/freertos_idf.html)
- [ESP-MQTT](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/protocols/mqtt.html)

模组手册不能代替具体开发板的原理图。IDF 6.x 的 MQTT 已拆为外部组件，做到该阶段再在实验工程中添加 `espressif/mqtt`，保存依赖声明和锁定文件。不要直接复制旧版“内置 mqtt 组件”的配置。
