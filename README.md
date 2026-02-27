# ZetaChain Crosschain Skill for OpenClaw

🌉 **OpenClaw** AI Agent 的 ZetaChain 原生跨链技能  
让你的 AI agent 能直接理解并执行 ZetaChain 的跨链操作：  
- 从 EVM 链（如 ETH、Polygon、BSC）deposit 资产到 ZetaChain  
- 触发 universal smart contract call  
- 查询和监控 CCTX（Cross-Chain Transaction）状态  
- 未来扩展：withdraw、BTC/SOL 支持、多链事件自动触发等

适用于 AI 自动化场景：跨链 DCA、跟随 whale 资产迁移、监控多链 LP 后一键转移等。

## 当前版本：v0.2.0
- 纯 prompt 指导 + fetch / api_call 查询费用和状态  
- 支持手动签名或通过 shell 调用本地脚本（安全沙箱）  
- 强制用户确认，绝不暴露私钥

## 快速本地使用

1. 克隆本仓库
   ```bash
   git clone https://github.com/你的用户名/zetachain-openclaw-skill.git
   cd zetachain-openclaw-skill
