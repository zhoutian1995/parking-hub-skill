# ParkingHub Skill

ParkingHub 用户侧 AI Skill。让任何 agent 通过 `https://parking.willeai.cn` 帮小区用户搜索车位、申请借用、查看记录、发布/下架自己的车位，并生成物业管家报备文案。

这个仓库是公开给其他 agent 使用的，只包含用户侧 API 使用说明，不包含服务器运维、数据库、SSH、AppSecret 或管理员操作。

## 能力

| 能力 | API |
|---|---|
| 搜索可借车位 | `GET /spots/search?q=A001` |
| 附近推荐 | `GET /spots/nearby?limit=5` |
| 区域统计 | `GET /spots/stats` |
| 申请借用 | `POST /borrows` |
| 查看我的记录 | `GET /borrows/mine` |
| 车主确认/拒绝 | `POST /borrows/{id}/accept` / `reject` |
| 取消/结束借用 | `POST /borrows/{id}/cancel` / `done` |
| 我的车位 | `GET /spots/mine` |
| 发布/下架车位 | `POST /spots/{id}/share` / `unshare` |
| 管家报备文案 | 从 `/borrows/mine` 记录生成 |

## 安装

### Hermes Agent

```bash
npx skills add zhoutian1995/parking-hub-skill
```

或手动安装：

```bash
mkdir -p ~/.hermes/skills/parking-hub
cp SKILL.md ~/.hermes/skills/parking-hub/SKILL.md
```

### Claude Code / Cursor / Codex / 其他 Agent

把 `SKILL.md` 放进对应 agent 的 skills/project instructions 目录即可。

## 配置

用户需要先登录 ParkingHub，进入「我的 -> AI 密钥管理」生成 API Key。

```bash
export PARKINGHUB_API_KEY="parkinghub_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
export PARKINGHUB_BASE_URL="https://parking.willeai.cn/api"
```

API Key 只会明文显示一次，请妥善保存；不要把它提交到 GitHub。

## 示例

```python
import os
import requests

BASE = os.getenv("PARKINGHUB_BASE_URL", "https://parking.willeai.cn/api")
KEY = os.getenv("PARKINGHUB_API_KEY")

headers = {"Authorization": f"Bearer {KEY}"}

# 搜索 A 区可借车位
spots = requests.get(f"{BASE}/spots/search", params={"q": "A0"}, headers=headers).json()

# 申请借用第一个车位
if spots:
    borrow = requests.post(
        f"{BASE}/borrows",
        json={"spot_id": spots[0]["id"], "plate": "浙A12345"},
        headers=headers,
    ).json()
    print(borrow)
```

## 使用原则

- 借用前必须确认车位和车牌号。
- 发布/下架/确认/拒绝/取消/结束等写操作必须先征得用户确认。
- 平台只做邻居之间车位撮合，不收款；付款线下完成。
- 不要编造房号、电话、车牌、付款状态。
- 不处理服务器运维、数据库、Nginx、PM2、微信 AppSecret 等内部事务。

## License

MIT
