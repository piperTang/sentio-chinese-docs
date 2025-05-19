# 触发器
触发器定义了与处理器绑定的特定类型的区块链活动，比如：onEventTransfer就是一个触发器

### 触发器类型
- 特定的event：比如：onEventXxx
- onEvent：捕获合约的任何事件
- 时间或者区块驱动的处理器，比如指定区块或者指定时间触发
- onTransaction（每一笔交易都会执行，会使用比较多的资源，分析gas price的时候可以使用）
- onCallxxx，函数调用的时候触发，适合没有event的合约
- onObjectChange，监听特定状态变量，资源类型（Aptos/Sui）或涉及合约的资产转移。

### ctx上下文对象
- ctx.meter ：访问 Counter 和 Gauge 以发射指标数据
- ctx.eventLogger ：用于发出结构化事件日志。
- ctx.store ：持久化实体存储的接口（用于读写实体）。
- ctx.exporter ：通过 Webhooks 发送数据的接口。
- ctx.blockNumber （或 Aptos/Sui 中的 ctx.version ）：正在处理的区块/版本号。
- ctx.timestamp ：当前区块的时间戳。
- ctx.chainId ：链标识符（例如 1 、 56 ）。
- ctx.transaction : 触发事件的相关交易信息（如适用，具体可用性取决于处理器类型）。
- ctx.contract : 用于进行视图调用的合约实例引用（详见下文）。

### 过滤器
通常与触发器一起使用
```typescript
// ERC20 Transfer event filter example
// Only trigger handler if 'from' is the zero address (mint event)
const mintFilter = ERC20Processor.filters.Transfer(
  '0x0000000000000000000000000000000000000000', // from address filter
  null, // to address filter (null means match any)
  // value filter (null means match any)
);

ERC20Processor.bind({ address: '0x...', network: 1 })
  .onEventTransfer((event, ctx) => {
    // This code only runs for transfers FROM the zero address
    ctx.meter.Counter('token_mints').add(event.args.value.scaleDown(18));
  }, mintFilter); // Pass the filter as the third argument
```