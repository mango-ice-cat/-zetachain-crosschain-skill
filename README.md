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
   git clone https://github.com/你的用户名/zetachain-openclaw-skill.git
   cd zetachain-openclaw-skill

2. 复制 SKILL.md 到 OpenClaw skills 目录bash
   mkdir -p ~/.openclaw/skills/zetachain-crosschain
   cp SKILL.md ~/.openclaw/skills/zetachain-crosschain/SKILL.md
3. 重启 OpenClaw 或重新加载 skills
4. 测试示例对话
   用 zetachain-crosschain 从 Ethereum deposit 0.01 ETH 到我的 ZetaChain 地址 0xYourZetaAddr 测试一下
   或
   查询 ZetaChain CCTX 状态 txHash 0xabc...
   
   安全声明私钥永不进入 prompt 或日志  
   所有交易必须用户手动 Y/N 确认  
   推荐使用限额测试钱包、多签或硬件钱包  
   shell 调用脚本时，限制在安全目录

## 下一步计划v0.3：集成 @zetachain
/toolkit 的 Node.js wrapper，实现可靠签名与广播  
支持 ZetaChain 2.0 private memory（让 agent 记住你的跨链偏好）  
发布到 ClawHub（clawhub.ai）  
欢迎 PR / issue / fork！


相关资源ZetaChain 官方文档：https://docs.zetachain.com  
ZetaChain Toolkit：https://github.com/zeta-chain/toolkit  
OpenClaw 项目：https://openclaw.ai  
ClawHub 技能市场：https://clawhub.ai

MIT Licensed | 为 ZetaChain × OpenClaw 社区而作
@0xTop_Gun
 – ZetaChain 大使Star & Fork 支持一下～





