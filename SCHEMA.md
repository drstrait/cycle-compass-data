# canonical.json 数据契约 v1.1(v1.0 + 交易日历新鲜度字段)

```json
{
  "version": "1.0",
  "chain": "2130",              // "1550"=东财延时即时链 | "2130"=Tushare对账正典链
  "trade_date": "20260828",
  "generated_at": "2026-08-28T21:35:00+08:00",
  "market": {
    "volume_yi": 21015,          // 两市成交额(亿)
    "volume_ma20_yi": 22442,
    "volume_ratio": 0.94,        // 相对MA20;前端展示须双口径标注周期
    "breadth_up": 2796, "breadth_down": 2270, "breadth_pct": 53.7
  },
  "ew": {                        // 等权全A(黄线)
    "value": 916.6, "ma20": 900.7, "ma60": 895.7,
    "spread_pct": -0.32          // 白-黄;负=个股强于指数
  },
  "regime": {
    "state": "震荡(修复)", "streak_days": 11,
    "pending_transition": "上升(第2日)"   // 无则 null
  },
  "limits": {
    "up_count": 82, "down_count": 1, "max_streak": 7,
    "ladder": [{"height":7,"count":1},{"height":5,"count":1},{"height":4,"count":3}],
    "promotion_rate_pct": 23, "fried_rate_pct": 16,
    "yesterday_premium_pct": 3.3,
    "auction_one_word_count": 4   // 疯狂表:竞价一字家数
  },
  "sectors": {
    "top": [{"name":"氮肥","pct":7.3}],
    "bottom": [{"name":"疫苗","pct":-3.9}]
  },
  "mustread": [
    {"code":"000017","name":"深中华A","streak":7,"industry":"饰品",
     "tor_f":46,"first_seal":"093300","open_times":14,"seal_yi":0.1}
  ],
  "j1": {"threshold_yi": 23000, "value_yi": 21015, "green": false}
}
```

## 规则
1. 判定类字段(regime/limits/j1)只允许来自本文件;`limits` 任一字段为 0 且当日为交易日时,前端必须显示「传感器待验证」而非将其计入证据。
2. `chain=1550` 为盘后快照,`2130` 覆盖同日数据;前端以最新 `generated_at` 为准并展示新鲜度。
3. 字段增删走 PR 修改本契约,版本号递增。

## regime.state 法定词表(v1.0.1 增补,2026-08-30)

基态五值,判定按**前缀匹配**;括注限定词(如"修复""衰竭")仅供人读,不参与判定:

| 基态 | 含义 | 司南四态映射 |
|---|---|---|
| 上升 | 状态机确认的上升段 | 通 |
| 下降 | 状态机确认的下降段 | 穷 |
| 高位震荡 | 高位滞涨/分歧区 | 久 |
| 低位震荡 | 低位盘整区 | 变 |
| 震荡 | 未定向震荡(可带修复/衰竭等括注) | 变 |

- `pending_transition` 格式:`目标基态(第N日)`,如 `上升(第2日)`;无候选为 null。
- 词表外的 state = 数据违约:exporter 推送前必须校验基态 ∈ 上表,不合格不推送并告警;前端遇词表外值按"变"降级并显式标注"词表外",不得静默吞掉。
- 前端映射必须是纯前缀表,禁止关键词正则猜测("冰点""退潮"等字样一律不参与判定)。

## 字段口径注记(2026-09-02,exporter 落地时定案;不改字段不升版本)

- 生产者:观照 `src/export_canonical.py`,15:50 主链尾 L4 自动推送(`chain=1550`);`chain=2130` 覆盖待夜间链产出对账后 metrics 时启用。
- 全部数值从观照 `data/metrics/<date>.json` 复算,可审计,无估算项;推送前按法定词表校验,不合格不推送并 Discord 告警。
- `limits.auction_one_word_count`:竞价一字家数 = Tushare `stk_auction` 中竞价价 ≥ 涨停价×0.995 的家数(按板块涨幅上限 10%/20%/30%);当日竞价数据缺失时为 0(触发规则 1「传感器待验证」)。8/28 手工引导版的 6 为估值,按本口径应为 13。
- `mustread`:观照必翻清单中的连板条目,按连板数降序取前 10(与日报涨停梯队一致);`tor_f`=自由流通换手%,`seal_yi`=封单亿元,`first_seal`=首次封板时间。
- `sectors.top/bottom`:东财行业板块涨跌幅前 5 / 后 5(概念板块不入总线)。
- `market.volume_yi` / `j1.value_yi`:沪深两市成交额(不含北交所)。

## v1.1(2026-09-02):新鲜度按交易日历(用户裁决,取代固定小时数)

新增顶层字段,由观照发布(交易日历权威 = Tushare SSE 日历),司南只做比较、不自算日历:

- `next_trade_date`:`"YYYYMMDD"`,`trade_date` 之后的第一个交易日;
- `valid_until`:ISO8601 带时区,= `next_trade_date` 的 `16:10:00+08:00`(15:50 链 + 20 分钟宽限)。

前端「当前」判定 = `now ≤ valid_until` ∧ `status=ready` ∧ 无传感器故障 ∧ `version` 主号 = 1。
由此周一盘中沿用上周五正典、节后沿用节前最后交易日正典自动成立;`valid_until` 一过即为过期 → 降级。
**缺 `valid_until` 的旧文件 = schema 不兼容 → 降级**。降级态下禁止方向性/买卖性 Bark,只允许中性数据故障提示(cycle-compass #20)。
`version` 升为 `"1.1"`;1.0 消费者忽略新增字段即可(向后兼容)。

> **过渡期(2026-09-02 起)**:线上司南 main 的引擎硬判 `version !== "1.0"` 即抛错,故观照暂以 `version: "1.0"` 发布并**同时携带** `next_trade_date` / `valid_until`(旧引擎忽略新字段;#20 分支按主版本号兼容并读取)。cycle-compass #20 合并上线后,观照 `export_canonical.SCHEMA_VERSION` 切回 `"1.1"`。
