# MoonBit RINEX 观测文件接收检查

**本项目尚未公开；只有本地源码。公开地址由对接团队创建后填写，旧 moonbit-ntp 地址不能替代新项目。**

模块 `xuting22/rinex`，0.1.1，MIT。拟替换旧 NTP 申报，换题尚未获赛事确认。

面向接收 GNSS 观测文件的 MoonBit 应用：在数据进入后续解算/归档前，分行检查观测声明、载荷、时间窗和缺失记录，并输出可定位的失败。核心读取流、时间计算和检查策略均为 MoonBit；Node 只提供有界文件 I/O。支持范围见 [SCOPE](docs/SCOPE.md)。

## 可直接复现

```sh
moon build --target js --release
node tools/check-file.mjs examples/minimal.rnx
node tools/check-file.mjs examples/brux-first-two-epochs.rnx 2026 9 23 -1
moon run examples/consumer --target wasm-gc
```

第一条文件检查返回0。第二条输入是公开BRUX日文件转换后未改动的前两个完整epoch，**预期退出2**：语法可完整检查，但显然不覆盖全天，头部首末/卫星总数也不匹配。它不能被当作合格整日记录。完整41MB日文件的核验见 [PUBLIC-DATA](PUBLIC-DATA.md)。

库调用 `Checker::new(window=Window::epn_daily(...))`，逐行 `feed_line`，最后 `finish`。不需要Node即可从MoonBit应用消费。核心不保存整文件，工作集随声明的系统/信号/卫星和当下epoch增长，不随已读历史长度增长。

## 三种结果

- 退出0：`complete=true, acceptable=true`，通过**本库已声明检查项**。
- 退出2：`complete=true, acceptable=false`，完整读完但存在数据/所选窗口问题，具体代码和行号在findings中。
- 退出1：不支持、语法/容量错误或I/O失败，`complete=false`，不会提供“部分合格”的结果。核心失败后不可继续，finish后也不可复用。

最多保留128条具体诊断，全部发生次数另计；不能因为例子被截断便认为后续没有问题。通过不代表完整RINEX合规、EPN认可、定位质量或生产部署。

## 公开数据已发现的实际差异

BRUX00BEL 2026-09-23数据由Royal Observatory of Belgium提供，CC-BY-4.0；原CRINEX经GSI RNXCMP4.2.0转换，未由本库解压。2,880个GPS时刻、136,993条卫星记录全部完成。头声明131颗卫星、体中只有127颗，返回 `header-satellite-total`；并披露PRN细项缺失。本项目不会为了演示通过而修掉原始头。

独立Python参考核对时刻/每系统记录、所有信号的非空/空字段和LLI位计数：共2815818个槽，非空2373196、空442622、数值零0。JS/Wasm-GC核心各31项，公开节选15组变异及纯MoonBit消费者通过。`nonempty_fields` 是文本槽非空计数，`zero_values` 单列数值零，**不把非空比例解释为物理有效观测率**。LLI位只按原值统计，不自行诊断周跳。

## 复用与边界

[GeoRust rinex](https://docs.rs/rinex/0.22.0/rinex/) 与 [gnss-js](https://github.com/MiguelPuntoEs/gnss-js) 已有成熟解析/QC，EPN也有质量检查。本库不是新GNSS算法，新增交付是可在MoonBit JS/Wasm应用中直接依赖的有界接收契约、精确100ns时间检查和显式daily/hourly策略。有限Mooncakes/GitHub检索未定位同范围MoonBit包，不等于生态空白或资格保证，见 [DUPLICATION](DUPLICATION.md)。

仅列明的RINEX3.03/3.04/3.05/4.00/4.01 OBS、GPS时系、flag0/1。不支持NAV、RINEX2、压缩、事件2–6、动态头更新、非GPS/闰秒时间。未解释的头标签会列出，尤其不应用缩放/接收机改正，也不校验PRN细项数值。没有卫星可见性模型、定位、精度结论或科研/测绘替代承诺。

编译器固定 `0.10.14+7d59c7ec9`，Node24；本地检查见 [TESTING](TESTING.md)。公开/换题动作、报名表和外部复审由团队完成。

2026-09-27追加：用固定GeoRinex1.16.2核对另一份公开CEDA3.03样本的时刻、信号计数和L1/L2 LLI，并独立复核814条间隔诊断。BRUX4.01未获该参考支持，原始版本头保持不变。详见 [GeoRinex参考](GEORINEX-REFERENCE.md)。

当前0.1.1又以固定Rust rinex0.22.0实际解析原BRUX4.01，并与重新运行的MoonBit结果核对90组星座/信号码及22组相位LLI统计。两种参考支持的版本与空槽边界分别说明，见 [原BRUX4.01参考](GEORUST-REFERENCE.md)。

## 本地验收与公开交付（2026-09-28）

核心实现使用 MoonBit；[固定编译器](.moonbit-version)为 `moonc 0.10.14+7d59c7ec9`。先按本文安装宿主依赖、运行 `moon update`，再从仓库根目录执行以下与 [CI](.github/workflows/ci.yml) 对齐的检查；可运行任务和适用边界见本文前面的示例与说明。

```sh
moon check --target all --deny-warn
moon test --target js --deny-warn
moon test --target wasm-gc --deny-warn
moon build --target js --release --deny-warn
moon package
```

跨平台复核（2026-09-28，本地 Ubuntu-D 26.04 WSL2）：从当时的源码归档全新解包，固定 `moonc 0.10.14+7d59c7ec9` 下通过 `moon update`、`moon fmt --check`、`moon info`、严格检查、JS/Wasm-GC 测试及 JS release 构建；Node 24.21.0 跑通本仓一条宿主入口。本次补记仅修改文档，代码与 CI 未变；复核日志在本地交接包中，公开提交后的 GitHub Actions 仍须单独核对。

本地核验：JS/Wasm-GC 各 31 项测试及 15 项 CLI/公开契约检查通过；严格检查无警告。`moon package` 已完成离线打包预检，它不等于已发布到 Mooncakes。

公开交付（2026-09-28 核对）：尚无本项目正式公开仓库 URL 或 Mooncakes 版本；当前模块名为 `xuting22/rinex`；换题资格、仓库、公开 CI 和首次发布均待团队办理，不能沿用旧题仓库链接。相关远端 CI 与赛事结果仍需以实际记录核对。项目许可见 [LICENSE](LICENSE)；如使用第三方材料，其来源和许可见仓内相应说明。
