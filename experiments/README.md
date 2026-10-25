# 实验工程

做一个建一个，不提前生成完整答案。命名按路线里的 `00_hello` 到 `11_stm32_link`；每个目录是独立 ESP-IDF 工程，VS Code 打开具体工程目录，不能在本仓库根目录直接执行 idf.py build。

每个工程至少保留 main、两层 CMakeLists.txt、必要的组件和配置说明。README 写清板子、IDF 版本、接线、构建运行步骤、预期现象和已知问题。增加外部组件时提交 `idf_component.yml` 和 `dependencies.lock`。

构建目录不提交。sdkconfig 可能含 Wi-Fi 密码，默认忽略；把复现所需且不含凭据的设置整理进 `sdkconfig.defaults`，注明本地怎样填写凭据。不要提交整个 IDF 安装目录或本机绝对路径。

实验输出放在当日 daily 的附件目录，用记录链接回对应代码提交。不要让下一次实验覆盖上一次跑通的工程。
