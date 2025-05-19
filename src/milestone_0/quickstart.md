# 快速开始
下面我们将运行一个最简单的sentio处理器来展示其基本的工作流程，我们将会监控一个erc20合约，并统计转账次数，以及总转账的金额

### 先决条件
- Node.js(版本22)
- Sentio账户：需要访问app.sentio.xyz注册
### step1
安装Sentio CLI，并登陆

```
npx @sentio/cli@latest login
```

### step2
创建一个项目
```
npx @sentio/cli@latest create erc20-counter
cd erc20-counter
```

### step3
获取weth 合约的abi,sentio会自动生成合约对应的模块代码
```
yarn sentio add --chain 1 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2
```

### step4
编写processor代码
src/processor.ts
```typescript
import { Counter, Gauge } from '@sentio/sdk'
import { ERC20Processor, ERC20Context, TransferEvent } from '@sentio/sdk/eth/builtin/erc20'
import { EthChainId } from '@sentio/sdk/eth'

// Define the contract address and network
const WETH_ADDRESS = '0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2'
const NETWORK = EthChainId.ETHEREUM // Or use '1'

// Define metrics
// 'transfer_cnt' will count the number of transfer events
const transferCount = Counter.register('transfer_cnt', {
  description: 'Counts the number of Transfer events',
})

// 'transfer_volume' will sum the value of tokens transferred
const transferVolume = Counter.register('transfer_volume', {
  description: 'Total volume of tokens transferred',
  unit: 'weth' // Optional unit
})

// Create and bind the ERC20 Processor
ERC20Processor.bind({ 
  address: WETH_ADDRESS, 
  network: NETWORK,
  // Optional: Specify a start block if you don't need full history
  // startBlock: 18000000 
})
.onEventTransfer(async (event: TransferEvent, ctx: ERC20Context) => {
  // This code runs every time a Transfer event occurs for WETH
  
  // Increment the transfer count metric by 1
  transferCount.add(ctx, 1)
  
  // Get the transfer value (it's a BigInt)
  const value = event.args.value
  
  // Add the transfer value to the volume counter
  // We use scaleDown(18) because WETH has 18 decimals
  transferVolume.add(ctx, value.scaleDown(18))
  
  // Optionally, log the event details (useful for debugging)
  ctx.eventLogger.emit('TransferProcessed', {
    distinctId: event.args.to, // Associate log with recipient
    from: event.args.from,
    to: event.args.to,
    value: value.scaleDown(18).toString(), // Log the scaled value
    message: `Processed WETH transfer from ${event.args.from} to ${event.args.to}`
  })
})

console.log('ERC20 Counter Processor started for WETH')

```

# step5
构建，检查语法错误
```
yarn sentio build
```
# step6
上传
```
yarn sentio upload
```
# step7
到dashboard中查看运行结果

# final
恭喜，你完成了一个基础的sentio项目的创建
