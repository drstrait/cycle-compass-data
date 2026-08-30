# canonical.json 数据契约 v1.0

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
