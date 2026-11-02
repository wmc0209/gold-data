# gold-data

通过 AkShare 获取的日线行情原始快照，用于历史数据可视化。

抓取日期：2026-11-02（Asia/Tokyo）。AkShare 版本：1.20.22。
安装来源：https://pypi.tuna.tsinghua.edu.cn/simple

## 文件与覆盖范围

行数不含表头。起止日期为实际返回记录，不代表产品的完整历史。

| 文件 | 品种与口径 | AkShare 调用 | 数据来源 | 起始日期 | 截止日期 | 行数 | 状态 |
| --- | --- | --- | --- | --- | --- | ---: | --- |
| [GC.csv](GC.csv) | COMEX黄金期货 | `ak.futures_foreign_hist(symbol='GC')` | 新浪财经 | 2016-11-02 | 2026-11-02 | 2589 | 已保存 |
| [SI.csv](SI.csv) | COMEX白银期货 | `ak.futures_foreign_hist(symbol='SI')` | 新浪财经 | 2016-11-02 | 2026-11-02 | 2589 | 已保存 |
| [PL.csv](PL.csv) | 铂金期货 | `ak.futures_foreign_hist(symbol='PL')` | 新浪财经 | 2019-04-18 | 2019-04-18 | 1 | 覆盖不足：仅1条旧记录 |
| [PA.csv](PA.csv) | 钯金期货 | `ak.futures_foreign_hist(symbol='PA')` | 新浪财经 | 2019-04-18 | 2019-04-18 | 1 | 覆盖不足：仅1条旧记录 |
| [NDX.csv](NDX.csv) | 纳斯达克100指数 | `ak.index_us_stock_sina(symbol='.NDX')` | 新浪财经 | 2014-02-18 | 2026-10-30 | 3195 | 已保存 |
| [BTC.csv](BTC.csv) | CME比特币期货（非比特币现货） | `ak.futures_foreign_hist(symbol='BTC')` | 新浪财经 | 2017-12-18 | 2026-11-02 | 2117 | 已保存 |

## 来源与接口

- GC、SI、PL、PA：严格调用指定的 `ak.futures_foreign_hist(symbol=...)`。
- NDX：严格调用 `ak.index_us_stock_sina(symbol=".NDX")`。
- BTC：调用 `ak.futures_foreign_hist(symbol="BTC")`。该代码在 AkShare 的新浪品种映射中是 **CME比特币期货**，不是比特币现货或 BTC/USD 现货指数。
- [新浪外盘期货](https://finance.sina.com.cn/money/future/hf.html)
- [新浪纳斯达克100](https://stock.finance.sina.com.cn/usstock/quotes/.NDX.html)
- [AkShare 外盘历史接口文档](https://akshare.akfamily.xyz/data/futures/futures.html)
- [AkShare 美股指数接口文档](https://akshare.akfamily.xyz/data/index/index.html)
- [AkShare 外盘历史接口源码](https://github.com/akfamily/akshare/blob/main/akshare/futures/futures_foreign.py)
- [AkShare 品种映射源码（含CME比特币期货）](https://github.com/akfamily/akshare/blob/main/akshare/futures/futures_hq_sina.py)

## 抓取异常与覆盖限制

- PL、PA 接口均只返回2019-04-18的一条旧记录，未取得完整日线历史。为保留指定接口的真实返回，仍保存这两份单行CSV，并明确标记覆盖不足；不补齐、不伪造其他日期。
- BTC现货历史未取得。当前版本没有旧版 `crypto_hist` 日线接口；官方更新日志记载该接口在1.15.34移除。当前可用的CME期货数据从2017-12-18开始，无法覆盖用户期望的2010年起点。没有混用现货与期货，也没有补造早期行情。
- [旧 crypto_hist 接口移除记录](https://github.com/akfamily/akshare/blob/main/docs/changelog_archive.md)
- [当前 AkShare 加密货币接口文档](https://github.com/akfamily/akshare/blob/main/docs/data/dc/dc.md)
- 所有日期范围均受上游接口限制，未额外拼接第三方数据。期货数据是新浪返回的品种序列；未自行构造连续合约、复权或计算展期收益。
- 2026-11-02的期货记录可能是抓取时尚未完成的当日行情，不保证为最终日结数据。

## 数据校验

| 文件 | 重复日期数 | OHLC边界与缺失值检查 |
| --- | ---: | --- |
| GC.csv | 0 | OHLC边界不一致 41 行，保留源值 |
| SI.csv | 0 | OHLC边界不一致 31 行，保留源值 |
| PL.csv | 0 | 未发现缺失值或OHLC边界异常 |
| PA.csv | 0 | 未发现缺失值或OHLC边界异常 |
| NDX.csv | 0 | 未发现缺失值或OHLC边界异常 |
| BTC.csv | 0 | OHLC边界不一致 83 行，保留源值 |

OHLC边界检查指 high 不应低于 open/close/low，low 不应高于 open/close/high。异常计数只是源数据检查结果；本仓库保留源值，不擅自修改成交价。用于绘制蜡烛图前应逐条复核这些记录。

## CSV格式与处理

- UTF-8编码，逗号分隔，第一行为字段名，不输出DataFrame索引。
- 日期统一为 `YYYY-MM-DD`，按日期升序稳定排序。
- 保留接口返回的全部字段，不重采样、不补交易日、不填补缺失值、不改变价格单位。
- `open/high/low/close` 为开高低收；`volume` 为接口原始成交量。
- 期货附加字段 `position`、`s`、`settlement` 以及NDX的 `amount` 均原样保留，字段可随上游变化。
- 已重新读取保存的CSV，核对字段名与行数一致。

## 字段列表

- `GC.csv`：`date`, `open`, `high`, `low`, `close`, `volume`, `position`, `s`, `settlement`
- `SI.csv`：`date`, `open`, `high`, `low`, `close`, `volume`, `position`, `s`, `settlement`
- `PL.csv`：`date`, `open`, `high`, `low`, `close`, `volume`, `position`, `s`
- `PA.csv`：`date`, `open`, `high`, `low`, `close`, `volume`, `position`, `s`
- `NDX.csv`：`date`, `open`, `high`, `low`, `close`, `volume`, `amount`
- `BTC.csv`：`date`, `open`, `high`, `low`, `close`, `volume`, `position`, `s`, `settlement`

## 文件校验值（SHA-256）

```text
1540a86e5eef1e2fb8860ea90344aa8fc667d3e9142fa78774df150f2ff0b2ec  GC.csv
813b63eca83050b16f71d90a1894df8096fe2b5e55a36cf4551ed95dfdfe60fe  SI.csv
578d53e3bb719dda8d881125882b81bcd8fb3db7beb540155c062001d5a5b444  PL.csv
67e03e84592c40659fa72425ebc60b57f488474f51f8c62c8934331e6e0ba262  PA.csv
f5a91acbf1d2778f324c71bd7f7b145a707ff5ac620add412e3766e6b3a84e2a  NDX.csv
2d85d53c82f115bd99f778803dc117a6bed1f9b863fa2e510d53a459515d4591  BTC.csv
```

仓库仅包含CSV与本README；安装环境、抓取脚本及本地日志不上传。
