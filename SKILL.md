---
name: parking-hub
description: ParkingHub 江南之星车位共享平台用户助手。帮助用户通过 https://parking.willeai.cn 搜索可借车位、申请借用、查看记录、发布/下架自己的车位、生成物业管家报备文案。适用于任何支持 HTTP/curl/Python 的 agent。触发：车位、停车、找车位、租车位、借车位、发布车位、共享车位、管家报备、ParkingHub、parking.willeai.cn。
---

# ParkingHub User Agent Skill

Use ParkingHub's public API to help residents find, borrow, share, and manage parking spots in Jiangnan Star.

This is a user-facing skill. Do not include server operations, secrets, private SSH paths, database access, or admin-only instructions.

## Setup

The user must sign in at `https://parking.willeai.cn`, open `我的 -> AI 助手`, create an API Key, and provide it to the agent.

Required environment variable:

```bash
export PARKINGHUB_API_KEY="parkinghub_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

Optional:

```bash
export PARKINGHUB_BASE_URL="https://parking.willeai.cn/api"
```

Authentication:

```http
Authorization: Bearer parkinghub_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

## API Helper

Use whatever HTTP client your agent supports. With Python:

```python
import os
import requests

BASE = os.getenv("PARKINGHUB_BASE_URL", "https://parking.willeai.cn/api").rstrip("/")
KEY = os.getenv("PARKINGHUB_API_KEY")

if not KEY:
    raise RuntimeError("PARKINGHUB_API_KEY is required")

def ph(method, path, **kwargs):
    headers = kwargs.pop("headers", {})
    headers["Authorization"] = f"Bearer {KEY}"
    res = requests.request(method, f"{BASE}{path}", headers=headers, timeout=20, **kwargs)
    if res.status_code >= 400:
        try:
            detail = res.json()
        except Exception:
            detail = {"error": res.text}
        raise RuntimeError(f"ParkingHub API {res.status_code}: {detail}")
    return res.json()
```

With curl:

```bash
curl -s "$PARKINGHUB_BASE_URL/me" \
  -H "Authorization: Bearer $PARKINGHUB_API_KEY"
```

## Common Workflows

### Identify the User

Call this first when the request depends on "my spots", "my records", or personalized nearby results.

```http
GET /me
```

Example:

```python
me = ph("GET", "/me")
```

### Search Available Spots

Use this for "有没有 A 区车位", "找 B112", "附近有没有可借车位".

```http
GET /spots/search?q=A001
```

Notes:
- `q` must be at least 2 characters.
- Search returns at most 20 available spots.
- If the API Key is provided, the user's own spots are filtered out.
- Spot code format is letter + 3 digits, such as `A001`, `B112`; do not use `A-001`.

Example:

```python
spots = ph("GET", "/spots/search", params={"q": "A0"})
```

If the user asks for nearby recommendations:

```http
GET /spots/nearby?limit=5
```

This requires an API Key and uses the user's preferred zone when available.

### List Spots or Zone Stats

Public list:

```http
GET /spots?zone=A&status=available
GET /spots/stats
GET /zones
GET /buildings
```

Use stats when the user asks "哪个区车位多" or "现在有多少可借车位".

### Borrow a Spot

Before creating a borrow request, always confirm:
- the exact spot
- the user's plate number

```http
POST /borrows
Content-Type: application/json

{
  "spot_id": 123,
  "plate": "浙A12345"
}
```

Example:

```python
borrow = ph("POST", "/borrows", json={"spot_id": 123, "plate": "浙A12345"})
```

Expected behavior:
- The request starts as `pending`.
- The owner must accept it.
- Payment is handled offline between residents; ParkingHub only coordinates.

Common error meanings:
- `请填写车牌号`: ask the user for the plate.
- `车位当前不可用`: search again and offer alternatives.
- `不能借用自己发布的车位`: do not proceed.
- `你已申请过该车位`: show existing records instead.

### View My Records

```http
GET /borrows/mine
```

Use this for:
- "我的借用记录"
- "车主有没有确认"
- "帮我生成管家报备"

### Owner Actions

For a spot owner:

```http
POST /borrows/{id}/accept
POST /borrows/{id}/reject
POST /borrows/{id}/done
```

For the borrower:

```http
POST /borrows/{id}/cancel
POST /borrows/{id}/done
```

Do not accept/reject/cancel/done without explicit user confirmation.

### Generate Property Manager Report Text

Use a borrow record from `/borrows/mine`. The preferred format is:

```text
1. 姓名：{borrower_name}
2. 房号：{borrower_building}{borrower_unit}
3. 联系电话：{owner_or_borrower_phone_if_available}
4. 车牌号：{borrower_plate}
5. 车位号：{spot_code}
```

If a field is missing, say it is missing and ask the user to fill it in. Do not invent names, room numbers, phone numbers, or plate numbers.

### Manage My Spots

```http
GET /spots/mine
POST /spots/bind
POST /spots/{id}/share
POST /spots/{id}/unshare
```

Bind a spot:

```json
{
  "spot_code": "A001"
}
```

Publish a spot:

```json
{
  "price_hour": 4,
  "price_cap": 20,
  "notes": "今晚可用",
  "available_from": "2026-04-22T18:00",
  "available_until": "2026-04-23T08:00"
}
```

Publishing requires:
- the spot belongs to the user
- payment method is bound in ParkingHub
- contract status is approved or legacy-approved

If the API says "请先绑定收款方式" or "请先上传车位租售合同", tell the user to complete that step in the website UI.

### Payment Information

```http
GET /me/payment
PUT /me/payment/alipay-account
```

Agents may help read or update the Alipay account only after the user explicitly asks. Payment and settlement are offline between residents; do not claim the platform has collected money.

## Response Style

- Reply in Chinese unless the user asks otherwise.
- Be concise and action-oriented.
- Always confirm plate number before borrowing.
- Always confirm before mutating actions: borrow, accept, reject, cancel, done, bind, share, unshare, payment update.
- Never expose API Keys in responses.
- Do not ask users for WeChat AppSecret, server credentials, database access, or admin-only details.

## Error Handling

If the API returns 401:
- Tell the user the API Key is missing, invalid, or revoked.
- Ask them to regenerate it at `我的 -> AI 助手`.

If rate-limited:
- Wait and retry later.

If a spot is unavailable:
- Search again and offer alternatives.

If the user asks for unsupported admin operations:
- Explain that this public skill only supports user-facing actions through the website API.
