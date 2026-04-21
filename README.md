# ParkingHub Skill

江南之星小区车位共享平台 AI 助手 Skill。

让 AI（Hermes Agent、Claude Code 等）帮用户搜索车位、预订、生成管家报备文案。

## 功能

| 工具 | 说明 |
|------|------|
| `parking_search` | 搜索可用车位 |
| `parking_book` | 预订车位 |
| `parking_confirm_payment` | 确认付款 |
| `parking_report_text` | 生成管家报备文案 |
| `parking_my_spots` | 查看我发布的车位 |
| `parking_my_bookings` | 查看我的预订记录 |
| `parking_share` | 发布空闲车位 |

## 对话示例

```
用户：今晚江南之星有车位吗？
AI：有 3 个可用车位：
    1. A-053 地面层 — ¥4/时，封顶¥20（张哥）
    2. B-112 地下一层 — ¥4/时，封顶¥20（李姐）
    3. C-088 地面层 — ¥4/时，封顶¥20（王叔）

用户：第二个
AI：请回复你的车牌号。
用户：桂A88888
AI：已预订 B-112，时间 18:00-08:00，费用 ¥20。
    请转账至支付宝：138xxxx（李姐）

用户：已付
AI：支付已确认！管家报备文案：
    "您好，我是3号楼2单元的业主，
     今晚18:00至明早08:00借用B-112车位，
     车牌号：桂A·88888，
     已通过平台完成支付。"
```

## 安装

### 方式一：Hermes Agent

```bash
# 安装 skill
npx skills add zhoutian1995/parking-hub-skill

# 或手动安装
cp SKILL.md ~/.hermes/skills/parking-hub/SKILL.md
```

### 方式二：Claude Code / Cursor / 其他 AI

将 `SKILL.md` 的内容作为系统 prompt 或 project instructions 添加到你的 AI 工具中。

## 配置

1. 注册 [ParkingHub](https://parking.willeai.cn) 账号
2. 进入「我的 → AI 密钥管理」生成 API Key
3. 配置环境变量：

```bash
export PARKINGHUB_API_KEY="parkinghub_xxxxxxxx"
export PARKINGHUB_BASE_URL="https://parking.willeai.cn/api"  # 可选
```

## License

MIT
