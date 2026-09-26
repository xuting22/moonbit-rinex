# 本地复查

固定工具链见 .moonbit-version；Moon CLI0.1.20260904、编译器0.10.12+1634b282e；Node24.11，Python3.14标准库。无运行时第三方依赖。

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
