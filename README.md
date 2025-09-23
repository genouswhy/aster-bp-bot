# ASTER-BP 对冲交易机器人

一个自动化对冲交易机器人，在 Backpack 交易所和 Aster 交易所之间执行 ASTER 代币的对冲策略，并自动管理资金费率时间。

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
5. 设置权限：**Spot Trading** (必须)
6. 复制 **API Key** 和 **Secret Key**

### 第三步：配置机器人
```bash
# 复制配置文件
cp config/hedge.example.yaml config/hedge.yaml

# 编辑配置文件
notepad config/hedge.yaml  # Windows
nano config/hedge.yaml     # Mac/Linux
```

**配置文件内容示例：**
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
  base_url: "https://sapi.asterdex.com"
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

### 第四步：运行机器人
```bash
# 运行循环对冲策略（推荐）
python scripts/hedge_bp_aster_loop.py config/hedge.yaml
```

## 📊 策略说明

### 对冲策略工作原理
1. **第一腿**：BP做空ASTER (+0.2%) → Aster买入ASTER对冲
2. **等待**：休眠20秒
3. **第二腿**：BP做多ASTER (-0.2%) → Aster卖出ASTER对冲
4. **循环**：等待60秒后重复

## 🎯 使用教程

### 教程1：首次运行测试
```bash
# 2. 观察日志输出，确认正常
# 3. 如果正常，运行循环版本
python scripts/hedge_bp_aster_loop.py config/hedge.yaml
```

### 教程2：调整交易参数
编辑 `config/hedge.yaml`：
```yaml
trade:
  quantity: "5"                     # 减少交易量到5个
  offset_percent: 0.1               # 减少价格偏移到0.1%
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
运行后你会看到类似输出：
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

## ⚙️ 配置参数详解

| 参数 | 说明 | 建议值 | 注意事项 |
|------|------|--------|----------|
| `quantity` | 每次交易数量 | 5-50 | 根据资金量调整 |
| `offset_percent` | 价格偏移百分比 | 0.1-0.5 | 太小可能不成交，太大利润少 |
| `stop_before_funding_minutes` | 资金费率前停止时间 | 3-10 | 确保能吃到资金费率 |
| `cycle_sleep` | 循环间隔秒数 | 60-300 | 避免过于频繁交易 |
| `debug` | 调试模式 | false | 生产环境建议关闭 |

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

## 📞 获取帮助

如果遇到问题：
1. 检查日志输出
2. 启用调试模式查看详细信息
3. 提交GitHub Issue

---

**⚠️ 风险提示：** 加密货币交易存在风险，请谨慎使用，建议先小额测试。
