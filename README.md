# How-to-learn-ESP23

# ESP32 学习记录

用 ESP32-WROOM-32 38Pin 开发板，跟着实验把 ESP-IDF 学一遍。开发环境用 VS Code + ESP-IDF，代码用 C。仓库名先保留，里面学的是 ESP32。

之前接触过 STM32 HAL、Linux C/C++、串口和 ROS2，C 语法不再从头过。重点放在 ESP-IDF 的工程组织、FreeRTOS、联网和两块 MCU 的配合上。最后做一个 STM32 采集、ESP32 联网、Linux 接收的完整小项目。

## 从这里开始

| 文件 | 用来做什么 |
|---|---|
| [CURRENT.md](CURRENT.md) | 上次停在哪，下次先做什么 |
| [ROADMAP.md](ROADMAP.md) | 实验顺序、关键知识点和阶段验收 |
| [ENVIRONMENT.md](ENVIRONMENT.md) | 工具版本、接线和 build/CMake 排错 |
| [MASTERY.md](MASTERY.md) | 哪些能独立完成，哪些还要复测 |
| [TEACHING_PROTOCOL.md](TEACHING_PROTOCOL.md) | 每次学习怎么推进 |
| [daily/TEMPLATE.md](daily/TEMPLATE.md) | 实验记录模板 |
| [experiments/README.md](experiments/README.md) | 后续实验工程怎么放 |
| [projects/README.md](projects/README.md) | 综合项目拆分与验收 |

先完成阶段 0，再做 GPIO。每次只推进一个能看见结果的小实验，做完保存接线、源码和日志。计划里的勾选框都是待办，不表示已经做过。

目前只有计划和记录入口，尚未上传实验源码。2026-10-25 的截图能看到固件大小统计，烧录和实际运行仍要单独确认，见 [当前记录](daily/2026-10-25.md)。

目录参考了自己 [LINUX-STUDY](https://github.com/KiloMing/LINUX-STUDY) 里的 Linux、ROS2 学习记录，以及 [XP4-project](https://github.com/KiloMing/XP4-project) 的阶段任务写法。保留“路线、当前断点、每天记录、独立复测”这几部分，没做过的实验不提前写结论。
