---
name: ZetaChain Crosschain
description: ZetaChain 原生跨链工具：支持从 EVM 链 deposit 资产到 ZetaChain、触发 universal contract call、监控 CCTX 状态。适用于 AI agent 自动化跨链迁移、DCA、whale 跟踪、多链事件响应等场景。
version: 1.0.0
homepage: https://github.com/mango-ice-cat/zetachain-crosschain-skill
tags: ["crosschain", "zetachain", "blockchain", "web3", "ai-agent", "omnichain"]
requires: {"tools": ["fetch", "api_call", "shell"]}
---

## 核心原则
- ZetaChain 是 Universal Layer，支持原生跨链（BTC/ETH/SOL/Sui 等），无需传统桥。
- 当前重点实现 EVM 链的 evmDeposit 模式（deposit + 可选 call）。
- 优先只做查询和构建 payload，签名/广播必须用户确认或通过安全沙箱脚本。
- 参考官方文档：https://docs.zetachain.com/docs/reference/toolkit
- RPC 限频：每分钟最多 5 次调用，避免被封禁。
- 安全第一：绝不让私钥出现在 prompt、日志或公开位置。

## 触发条件（Triggers）
当用户提到以下任一情况时激活本 Skill：
- "ZetaChain"、"cross-chain"、"deposit to Zeta"、"zetachain deposit"
- 具体命令示例："deposit 100 USDC from Ethereum to ZetaChain"、"从 Polygon 跨链 50 ETH 到我的 Zeta 地址"
- "monitor CCTX"、"check cross-chain tx"、"查询 CCTX 状态 0x..."
- 场景描述："帮我自动跨链 DCA"、"whale 转移资产后跟进"、"多链事件触发 ZetaChain 操作"

## 执行步骤（Steps）
1. **解析用户输入（Parse Input）**
   - source_chain：源链（如 eth、polygon、bsc、arbitrum 等，默认 eth）
   - token：资产（native 或 ERC-20 合约地址，如 USDC）
   - amount：数量（字符串，支持小数，如 "100" 或 "0.01"）
   - receiver：目标 ZetaChain 地址（EOA 或 universal contract 地址）
   - optional calldata：如果要 deposit-and-call，附加的 calldata（hex 字符串）
   - action：deposit / call / withdraw / monitor（默认 deposit）

2. **查询 ZetaChain 必要信息（Query Info）**
   - 使用 fetch 或 api_call 工具：
     - RPC URL：https://rpc.zetachain.com （或用户自定义）
     - 获取最新区块：GET /eth_getBlockByNumber 或 POST eth_getBlockByNumber
     - 获取 Gateway / TSS 地址：参考 docs 中的 Gateway 合约地址
     - 查询费用：POST eth_call 到 Gateway 查询 fee，或从 https://api.zetachain.com/v1/cctx/fee 获取估算
   - 如果需要源链信息（如 Polygon Gateway 地址），可额外 fetch 源链 RPC

3. **构建交易 Payload（Build Payload）**
   - evmDeposit 模式：
     - to: 源链 Gateway 合约地址（从 docs 获取）
     - value: 如果 native token，则为 amount（wei 单位）
     - data: abi.encode("deposit(address receiver, address asset, uint256 amount, bytes message)")
       - message：如果有 call，则为 calldata；否则为空
   - withdraw 模式：从 ZetaChain 发起 outbound（参考 docs withdraw 部分）
   - 输出 payload 结构：{to, value, data, chainId, gas 等}

4. **签名与广播（Execute / Sign）**
   - 如果用户配置了安全 signer 脚本（如 node ~/zetachain-tool.js）：
     执行 shell：
     ```
     node {signer_script_path} --action deposit --chain {source_chain} --amount {amount} --token {token} --receiver {receiver} --confirm
     ```
   - 否则（最安全默认路径）：
     - 输出完整交易数据（to, value, data hex）
     - 指导用户："请使用 MetaMask / WalletConnect 发送到 [源链 RPC]，to: [Gateway], data: [hex], value: [wei]。完成后提供 tx hash 给我监控。"
     - 等待用户回复 "Y" 或 tx hash 才继续

5. **监控与验证（Monitor / Verify）**
   - 使用 fetch 轮询：
     - https://api.zetachain.com/v1/cctx/{inbound_tx_hash}
     - 或 ZetaChain explorer API：https://zetachain.com/explorer/cctx/{hash}
   - 每 30 秒查一次，最多 15 分钟
   - 返回关键信息：inbound status, cctx_id, outbound status, estimated time, explorer link

## 安全守则（Safety Gating）
- 绝不 在任何 prompt、日志、输出中明文出现私钥或敏感 seed
- 每笔交易前必须询问用户："确认执行？金额：X，预计费用：Y [Y/N]"
- shell 调用限制在用户指定的安全目录（如 ~/zetachain-tools/）
- 出错时 fallback：提供 ZetaChain 文档链接 + 手动操作指导
- 建议用户：使用测试网先验证、限额钱包、多签、硬件钱包

## 示例对话（Examples）
**用户**：从 Polygon deposit 100 USDC 到 ZetaChain 地址 0xMyZetaAddr 并调用合约函数 mint(1000)

**Agent 响应**：
1. 已解析：source = polygon, token = 0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174 (USDC Polygon), amount = 100 (100000000), receiver = 0xMyZetaAddr, calldata = abi.encode(mint(1000))
2. 查询中... Polygon gas ≈ 50 gwei, Zeta fee ≈ 0.1%, 总费用估算 ≈ 0.003 MATIC + Zeta 费用
3. 构建 payload：
   - to: Polygon Gateway 0x...
   - data: 0x... (deposit + call calldata)
   - value: 0 (ERC-20)
4. 请确认交易：金额 100 USDC，预计 gas 约 150,000 [Y/N]
   （如果 Y，则指导 MetaMask 发送，或执行 shell 脚本）
5. 广播后监控：inbound tx hash 0x..., CCTX id 待查询，链接：https://zetachain.com/explorer/cctx/...

**用户**：monitor CCTX 0x123abc...

**Agent 响应**：
正在查询 CCTX 0x123abc...
状态：Inbound Confirmed → Outbound Processing
预计剩余时间：约 3-8 分钟
Explorer: https://zetachain.com/explorer/cctx/0x123abc...
