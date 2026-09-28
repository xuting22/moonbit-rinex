# 工具链固定版本更新（2026-09-28）

已将 .moonbit-version 更新为 0.10.14+7d59c7ec9。moon update、格式检查、全目标 check、JS/Wasm-GC 测试（各31项）、release JS 构建、moon info 和生成接口差异检查均通过。公开契约运行器通过15项检查，consumer示例在 JS 和 Wasm-GC 上均运行通过。check报告14条 implicit_impl_as_method 弃用警告，CI未将警告设为失败。本次未重跑 2026-09-27 回执所记载的 Rust 独立对照。

## 先前追加验证（2026-09-27）

运行时代码/API未变。本轮重新构建JS release，并完整处理原41MB BRUX4.01（预期退出2，继续报告头/体不符），用固定Rust rinex0.22.0实际解析输出直接比对：2880历元、136993卫星记录、127颗卫星、90组观测计数和22组相位LLI位计数全部一致。参考工具Rust/Cargo1.85.1；便携比较器及依赖锁文件随源码保存，详见 [GEORUST-REFERENCE](GEORUST-REFERENCE.md)。

本轮没有重复执行未改MoonBit核心的JS/Wasm全测试，原测试为下面的基线。新证据在evidence/georust-20260927/LOCAL-CHECKS.json；未运行远程CI，没有定位或完整RINEX合规证明。

## 本地复查

固定工具链见 .moonbit-version；Moon CLI0.1.20260920、编译器0.10.14+7d59c7ec9；Node24.11，Python3.14标准库。无运行时第三方依赖。

```sh
moon fmt
moon info
moon check --target all
moon test --target js
moon test --target wasm-gc
moon build --target js --release
node tools/test-public-contract.mjs
moon run examples/consumer --target js
moon run examples/consumer --target wasm-gc
```

核心检查针对定宽/精确时间/整数容量、续行与错误状态；容量边界直接设置内部计数器验证预检/封闭，不生成512MB无意义夹具。公开节选15组变异验证截断、重复、非法数值、100ns格点偏移、未支持事件、CRLF及严格字节/路径/参数边界。独立参考按 PUBLIC-DATA.md 获取完整源。

实际回执在 evidence/public-20260927；每次改动源应重新运行受影响检查并更新源码指纹。CI配置不是已在GitHub runner运行的证据，所有当前验证均本地。无测速、地学正确性、生产或组织采纳结论。
