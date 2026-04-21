---
name: parking-hub
description: 江南之星车位共享平台 AI 助手 — 搜索车位、预订、发布、生成管家报备文案。触发：用户提到"车位""停车""租车位""找车位"
---

# ParkingHub Skill

江南之星小区车位共享平台 AI 助手。

## 触发条件

用户说：
- "车位" "停车" "租车位" "找车位"
- "有没有车位" "车位出租"
- "帮我找车位" "帮我租车位"
- "发布车位" "共享车位"
- "管家报备" "报备文案"

## 前置配置

用户需要在 ParkingHub 生成 API Key 并配置：

```bash
export PARKINGHUB_API_KEY="parkinghub_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
export PARKINGHUB_BASE_URL="https://parking.willeai.cn/api"  # 可选
```

## 工具调用方式

所有操作通过 `execute_code` 调用 ParkingHub REST API：

```python
import os, json
from hermes_tools import terminal

API = os.environ.get('PARKINGHUB_BASE_URL', 'https://parking.willeai.cn/api')
KEY = os.environ.get('PARKINGHUB_API_KEY', '')

def api_call(method, path, body=None):
    cmd = f'curl -s -X {method} "{API}{path}" -H "Authorization: Bearer {KEY}" -H "Content-Type: application/json"'
    if body:
        cmd += f" -d '{json.dumps(body)}'"
    return terminal(cmd)
```

## 工具列表

### 1. parking_search — 搜索可用车位
- **触发词**：有没有车位、找车位、附近车位
- **API**: `GET /api/spots/search`
- **参数**: zone?, slot?

### 2. parking_book — 预订车位
- **触发词**：租、订、预订
- **API**: `POST /api/borrows`
- **参数**: spot_id, borrower_plate, start_time, end_time
- **流程**: 确认车牌号 → 调 API → 返回收款信息

### 3. parking_confirm_payment — 确认付款
- **触发词**：已付、付款了
- **自动触发报备文案生成**

### 4. parking_report_text — 管家报备文案
- **模板**: "您好，我是{building}{unit}的业主，{start}至{end}借用{spot}车位，车牌号：{plate}，已通过平台完成支付。"
- **数据来源**: users(building+unit) + borrows(start/end/plate) + spots(spot_code)

### 5. parking_my_spots — 我发布的车位
- **API**: `GET /api/spots/mine`

### 6. parking_my_bookings — 我的预订
- **API**: `GET /api/borrows/mine`

### 7. parking_share — 发布车位
- **API**: `POST /api/spots/{id}/share`
- **参数**: spot_code, available_from, available_until

## 完整对话示例

```
用户：今晚江南之星有车位吗？
AI：[parking_search] → 列出3个车位
用户：第二个
AI：请回复你的车牌号。
用户：桂A88888
AI：[parking_book] → 返回预订结果+收款信息
用户：已付
AI：[parking_confirm_payment] → 自动生成管家报备文案
```

## 注意事项

- 中文为主
- 每次预订必须确认车牌号（可能借朋友的车）
- 管家文案自动从用户资料和预订数据生成
- API Key 在平台"我的 → AI 密钥管理"生成