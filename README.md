# ASTER-BP 对冲交易机器人

一个自动化对冲交易机器人，在 Backpack 交易所和 Aster 交易所之间执行 ASTER 代币的对冲策略，并自动管理资金费率时间。

## 📋 支持的功能

- **现货对冲**: Backpack 现货 vs Aster 现货
- **合约对冲**: Backpack 现货 vs Aster 合约 (推荐)
- **智能监控**: 持续监控订单状态，自动重挂未成交订单
- **资金费率管理**: 自动检测资金费率时间，避免在费率结算前后交易
- **错误处理**: 完善的错误处理和重试机制

## 🚀 快速开始

### 第一步：准备环境
```bash
# 1. 下载项目
git clone <repository-url>
cd aster-bp-bot

# 2. 创建虚拟环境
python -m venv .venv

# 3. 激活虚拟环境
# Windows:
.venv\Scripts\activate
# Mac/Linux:
source .venv/bin/activate

# 4. 安装依赖
pip install -r requirements.txt
```

### 第二步：获取API密钥

#### Backpack 交易所 API
1. 访问 [Backpack Exchange](https://backpack.exchange)
2. 注册/登录账户
3. 进入 **Settings** → **API Keys**
4. 点击 **Create New Key**
5. 设置权限：**Trading** (必须)
6. 复制 **Public Key** 和 **Secret Key** (base64格式)

#### Aster 交易所 API
1. 访问 [Aster Exchange](https://asterdex.com)
2. 注册/登录账户
3. 进入 **API Management**
4. 点击 **Create API Key**
5. 设置权限：
   - **现货交易**: Spot Trading (现货对冲)
   - **合约交易**: Futures Trading (合约对冲，推荐)
6. 复制 **API Key** 和 **Secret Key**

### 第三步：配置机器人

#### 现货对冲配置
```bash
# 复制现货配置文件
cp config/hedge.example.yaml config/hedge.yaml

# 编辑配置文件
notepad config/hedge.yaml  # Windows
nano config/hedge.yaml     # Mac/Linux
```

**现货对冲配置文件示例：**
```yaml
bp:
  api_public_key_b64: ""
  api_secret_key_b64: ""
  base_url: "https://api.backpack.exchange"
  symbol: "ASTER_USDC_PERP"
  window: 5000
  debug: false

aster:
  api_key: ""
  api_secret: ""
  base_url: "https://sapi.asterdex.com"  # 现货API
  symbol: "ASTERUSDT"
  recv_window: 5000
  debug: false

trade:
  quantity: "10"                    # 每次交易数量
  offset_percent: 0.2               # 价格偏移0.2%
  first_wait_seconds: 10            # 监控等待时间
  between_legs_sleep: 20            # 两腿间隔时间
  stop_before_funding_minutes: 5    # 资金费率前5分钟停止
  cycle_sleep: 60                   # 循环间隔60秒
```

#### 合约对冲配置 (推荐)
```bash
# 复制合约配置文件
cp config/hedge_futures.example.yaml config/hedge_futures.yaml

# 编辑配置文件
notepad config/hedge_futures.yaml  # Windows
nano config/hedge_futures.yaml     # Mac/Linux
```

**合约对冲配置文件示例：**
```yaml
bp:
  api_public_key_b64: ""
  api_secret_key_b64: ""
  base_url: "https://api.backpack.exchange"
  symbol: "ASTER_USDC_PERP"
  window: 5000
  debug: false

aster:
  api_key: ""
  api_secret: ""
  base_url: "https://fapi.asterdex.com"  # 合约API
  symbol: "ASTERUSDT"
  recv_window: 5000
  debug: false

trade:
  quantity: "10"                    # 每次交易数量
  offset_percent: 0.2               # 价格偏移0.2%
  first_wait_seconds: 10            # 监控等待时间
  between_legs_sleep: 60            # 两腿间隔时间（合约建议更长）
  stop_before_funding_minutes: 5    # 资金费率前5分钟停止
  cycle_sleep: 60                   # 循环间隔60秒
```

### 第四步：运行机器人

#### 现货对冲 (现货 vs 现货)
```bash
# 运行现货对冲策略
python scripts/hedge_bp_aster_loop.py config/hedge.yaml
```

#### 合约对冲 (现货 vs 合约) - 推荐
```bash
# 运行合约对冲策略
python scripts/hedge_bp_aster_futures_loop.py config/hedge_futures.yaml
```

#### 为什么推荐合约对冲？
- **更好的流动性**: 合约市场通常有更好的流动性
- **更低的滑点**: 合约交易滑点通常更低
- **更精确的对冲**: 合约可以更精确地控制仓位
- **资金费率收益**: 可以吃到资金费率收益

## 📊 策略说明

### 现货对冲策略工作原理
1. **第一腿**：BP做空ASTER (+0.2%) → Aster现货买入ASTER对冲
2. **等待**：休眠20秒
3. **第二腿**：BP做多ASTER (-0.2%) → Aster现货卖出ASTER对冲
4. **循环**：等待60秒后重复

### 合约对冲策略工作原理 (推荐)
1. **第一腿**：BP做空ASTER (+0.2%) → Aster合约做多ASTER对冲
2. **等待**：休眠60秒 (合约建议更长等待时间)
3. **第二腿**：BP做多ASTER (-0.2%) → Aster合约做空ASTER对冲
4. **循环**：等待60秒后重复

### 合约对冲的优势
- **资金费率收益**: 每8小时收取一次资金费率
- **更好的流动性**: 合约市场流动性通常更好
- **更精确的对冲**: 可以精确控制仓位大小
- **更低的交易成本**: 合约交易手续费通常更低

## 🎯 使用教程

### 教程1：首次运行测试

#### 现货对冲测试
```bash
# 1. 运行现货对冲测试
python scripts/hedge_bp_aster_loop.py config/hedge.yaml

# 2. 观察日志输出，确认正常
# 3. 如果正常，继续运行
```

#### 合约对冲测试 (推荐)
```bash
# 1. 运行合约对冲测试
python scripts/hedge_bp_aster_futures_loop.py config/hedge_futures.yaml

# 2. 观察日志输出，确认正常
# 3. 如果正常，继续运行
```

### 教程2：调整交易参数

#### 现货对冲参数调整
编辑 `config/hedge.yaml`：
```yaml
trade:
  quantity: "5"                     # 减少交易量到5个
  offset_percent: 0.1               # 减少价格偏移到0.1%
  between_legs_sleep: 30            # 两腿间隔30秒
  cycle_sleep: 120                  # 增加循环间隔到2分钟
```

#### 合约对冲参数调整 (推荐)
编辑 `config/hedge_futures.yaml`：
```yaml
trade:
  quantity: "5"                     # 减少交易量到5个
  offset_percent: 0.1               # 减少价格偏移到0.1%
  between_legs_sleep: 90            # 两腿间隔90秒 (合约建议更长)
  cycle_sleep: 120                  # 增加循环间隔到2分钟
```

### 教程3：启用调试模式
```yaml
bp:
  debug: true                       # 启用BP调试日志
aster:
  debug: true                       # 启用Aster调试日志
```

### 教程4：监控运行状态

#### 现货对冲运行日志
```
[Cycle 1] 开始新一轮对冲策略
[Cycle 1] 距离资金费率时间 16:00 UTC 还有 120分钟
[Leg1] BP 限价做空价: 0.1236 (基于最新价 0.1234)
[Leg1] 开始监控 BP 做空订单 12345，将持续监控直到成交...
[Leg1] 监控中... 已等待 1s，订单 12345 未成交
[Leg1] BP 做空订单 12345 已成交！总耗时 3 秒
[Leg1] BP 做空已成交，ASTER 市价买入对冲...
休眠 20 秒...
[Leg2] BP 限价做多价: 0.1232 (基于最新价 0.1234)
[Leg2] BP 做多订单 12346 已成交！总耗时 2 秒
[Leg2] BP 做多已成交，ASTER 市价卖出对冲...
[Cycle 1] 完成，等待 60 秒后开始下一轮...
```

#### 合约对冲运行日志 (推荐)
```
[Cycle 1] 开始新一轮对冲策略
[Cycle 1] 距离资金费率时间 16:00 UTC 还有 120分钟
[Leg1] BP 限价做空价: 0.1236 (基于最新价 0.1234)
[Leg1] 开始监控 BP 做空订单 12345，将持续监控直到成交...
[Leg1] 监控中... 已等待 1s，订单 12345 未成交
[Leg1] BP 做空订单 12345 已成交！总耗时 3 秒
[Leg1] BP 做空已成交，ASTER合约 市价买入对冲...
[Leg1] ASTER合约 市价买入回执: {'orderId': 12345, 'status': 'FILLED', ...}
休眠 60 秒...
[Leg2] BP 限价做多价: 0.1232 (基于最新价 0.1234)
[Leg2] BP 做多订单 12346 已成交！总耗时 2 秒
[Leg2] BP 做多已成交，ASTER合约 市价卖出对冲...
[Leg2] ASTER合约 市价卖出回执: {'orderId': 12346, 'status': 'FILLED', ...}
[Cycle 1] 完成，等待 60 秒后开始下一轮...
```

## ⚙️ 配置参数详解

### 通用参数

| 参数 | 说明 | 现货建议值 | 合约建议值 | 注意事项 |
|------|------|------------|------------|----------|
| `quantity` | 每次交易数量 | 5-50 | 5-50 | 根据资金量调整 |
| `offset_percent` | 价格偏移百分比 | 0.1-0.5 | 0.1-0.5 | 太小可能不成交，太大利润少 |
| `stop_before_funding_minutes` | 资金费率前停止时间 | 3-10 | 3-10 | 确保能吃到资金费率 |
| `cycle_sleep` | 循环间隔秒数 | 60-300 | 60-300 | 避免过于频繁交易 |
| `debug` | 调试模式 | false | false | 生产环境建议关闭 |

### 现货对冲专用参数

| 参数 | 说明 | 建议值 | 注意事项 |
|------|------|--------|----------|
| `between_legs_sleep` | 两腿间隔时间 | 20-60秒 | 现货可以较短 |
| `base_url` | Aster API地址 | `https://sapi.asterdex.com` | 现货API |

### 合约对冲专用参数 (推荐)

| 参数 | 说明 | 建议值 | 注意事项 |
|------|------|--------|----------|
| `between_legs_sleep` | 两腿间隔时间 | 60-120秒 | 合约建议更长等待时间 |
| `base_url` | Aster API地址 | `https://fapi.asterdex.com` | 合约API |
| `recv_window` | 接收窗口时间 | 5000-10000 | 合约API建议更长 |

## 🔧 常见问题解决

### 问题1：API密钥错误
```
HTTP 401: Unauthorized
```
**解决方案：**
1. 检查API密钥是否正确复制
2. 确认API权限包含交易权限
3. 检查API密钥是否过期

### 问题2：订单查询404错误
```
HTTP 404: RESOURCE_NOT_FOUND
```
**说明：** 这是正常现象，表示订单可能已成交，机器人会自动处理。

### 问题3：网络连接问题
```
requests.exceptions.ConnectionError
```
**解决方案：**
1. 检查网络连接
2. 确认防火墙设置
3. 尝试使用VPN

### 问题4：交易对不存在
```
无法获取最新价格
```
**解决方案：**
1. 检查交易对符号是否正确
2. 确认交易对在交易所存在
3. 检查API权限

## 📈 最佳实践

### 1. 资金管理
- 建议使用总资金的10-20%进行对冲
- 保持两个交易所账户有足够余额
- 定期检查账户余额

### 2. 风险控制
- 从小额开始测试
- 设置合理的价格偏移
- 监控运行状态

### 3. 参数调优
- 根据市场波动调整价格偏移
- 根据流动性调整交易量
- 根据网络状况调整等待时间

## 🛑 停止机器人

按 `Ctrl+C` 停止机器人运行。

## 🚀 合约对冲详细说明

### 为什么选择合约对冲？

合约对冲相比现货对冲有以下优势：

1. **资金费率收益**: 每8小时收取一次资金费率，这是额外的收益来源
2. **更好的流动性**: 合约市场通常有更好的流动性和更小的滑点
3. **更精确的对冲**: 可以精确控制仓位大小，不受现货余额限制
4. **更低的交易成本**: 合约交易手续费通常比现货更低
5. **杠杆支持**: 可以使用杠杆放大收益（需要谨慎）

### 合约对冲策略详解

#### 第一腿：BP做空 + Aster合约做多
```
1. 获取BP最新价格
2. 计算做空价格（最新价 + 0.2%）
3. 在BP上下限价做空订单
4. 持续监控订单状态，10秒内未成交则重新挂单
5. 订单成交后，立即在Aster合约上做多对冲
6. 检查Aster合约订单状态，确保对冲成功
```

#### 第二腿：BP做多 + Aster合约做空
```
1. 等待60秒（可配置）
2. 获取BP最新价格
3. 计算做多价格（最新价 - 0.2%）
4. 在BP上下限价做多订单
5. 持续监控订单状态，10秒内未成交则重新挂单
6. 订单成交后，立即在Aster合约上做空对冲
7. 检查Aster合约订单状态，确保对冲成功
```

### 资金费率管理

合约对冲的一个重要优势是可以吃到资金费率收益：

- **资金费率时间**: 每8小时一次（00:00, 08:00, 16:00 UTC）
- **自动停止**: 在资金费率结算前5分钟自动停止交易
- **收益计算**: 如果资金费率为正，做多方收取费用；如果为负，做空方收取费用

### 合约对冲配置示例

```yaml
# config/hedge_futures.yaml
bp:
  api_public_key_b64: "YOUR_BP_PUBLIC_KEY"
  api_secret_key_b64: "YOUR_BP_SECRET_KEY"
  base_url: "https://api.backpack.exchange"
  symbol: "ASTER_USDC_PERP"
  window: 5000
  debug: false

aster:
  api_key: "YOUR_ASTER_API_KEY"
  api_secret: "YOUR_ASTER_API_SECRET"
  base_url: "https://fapi.asterdex.com"  # 合约API
  symbol: "ASTERUSDT"
  recv_window: 5000
  debug: false

trade:
  quantity: "10"                    # 交易数量
  offset_percent: 0.2               # 价格偏移0.2%
  first_wait_seconds: 10            # 第一腿等待时间
  between_legs_sleep: 60            # 两腿间隔60秒（合约建议更长）
  stop_before_funding_minutes: 5   # 资金费率前5分钟停止
  cycle_sleep: 60                   # 循环间隔60秒
```

### 合约对冲运行示例

```bash
# 运行合约对冲
python scripts/hedge_bp_aster_futures_loop.py config/hedge_futures.yaml
```

### 合约对冲日志示例

```
[Cycle 1] 开始新一轮对冲策略
[Cycle 1] 距离资金费率时间 16:00 UTC 还有 120分钟

[Leg1] BP 限价做空价: 1.7234 (基于最新价 1.72)，symbol=ASTER_USDC_PERP
[Leg1] BP 做空下单回执: [{'orderId': '10016317016', 'status': 'New', ...}]
[Leg1] 开始监控 BP 做空订单 10016317016，将持续监控直到成交...
[Leg1] BP 做空订单 10016317016 已成交！总耗时 3 秒，状态: FILLED
[Leg1] BP 做空已成交，ASTER合约 市价买入对冲...
[Leg1] ASTER合约 市价买入回执: {'orderId': 12345, 'status': 'FILLED', ...}

休眠 60 秒...

[Leg2] BP 限价做多价: 1.7166 (基于最新价 1.72)，symbol=ASTER_USDC_PERP
[Leg2] BP 做多下单回执: [{'orderId': '10016317017', 'status': 'New', ...}]
[Leg2] 开始监控 BP 做多订单 10016317017，将持续监控直到成交...
[Leg2] BP 做多订单 10016317017 已成交！总耗时 2 秒，状态: FILLED
[Leg2] BP 做多已成交，ASTER合约 市价卖出对冲...
[Leg2] ASTER合约 市价卖出回执: {'orderId': 12346, 'status': 'FILLED', ...}

[Cycle 1] 完成，等待 60 秒后开始下一轮...
```

## 📞 获取帮助

如果遇到问题：
1. 检查日志输出
2. 启用调试模式查看详细信息
3. 提交GitHub Issue

---

**⚠️ 风险提示：** 加密货币交易存在风险，请谨慎使用，建议先小额测试。合约交易风险更高，请确保充分了解合约交易机制。
