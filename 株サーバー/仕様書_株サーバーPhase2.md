# 1銘柄 + 日経平均 Fourier試作 v2

日足データを使うPhase 1試作に、ローリング・バックテストを追加した版です。

## 試作の前提
- 銘柄と日経平均を同一営業日にそろえる
- 各ローリング窓の初日を1として `P(t) / P(0)` で無次元化
- 5高調波Fourier回帰
- 銘柄、日経平均、銘柄/日経の相対比率の直近傾きを計算
- 売買代金近似値は `(High + Low + Close) / 3 * Volume`
- シグナル確定日は終値確定後
- エントリーは翌営業日の始値。シグナル日の終値では買わない
- 5営業日以内に高値がエントリー比+5%へ到達したら、+5%で利確したものと仮定
- 未達なら5営業日目の終値で時間切れ退出
- 同時に1銘柄だけ保有する想定。保有中に出た新規シグナルは見送る
- 手数料、税、スリッページは初期値0。設定で往復コストを差し引ける

## 注意
これは研究・検証用です。売買推奨や利益保証ではありません。高値が目標へ触れた場合に目標価格で売れたと仮定するため、ギャップ、特別気配、値幅制限、流動性不足を完全には再現しません。

## インストール
```sh
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp config.example.yaml config.yaml
```

## 実行
現在値分析とバックテストを一度に実行します。
```sh
python main.py --config config.yaml
```

テスト:
```sh
python -m unittest discover -s tests
```

## 出力
- `output/analysis.csv`: 全期間の現在値分析
- `output/summary.json`: 最新状態
- `output/comparison.png`: 現在値分析グラフ
- `output/backtest_trades.csv`: 仮想取引ごとの結果
- `output/backtest_summary.json`: 集計
- `output/backtest_equity.png`: 1取引1単位で複利化した参考曲線

## 初期シグナル条件
初期値は過去R版の再現値ではなく、コードを動かすための仮置きです。

- 銘柄Fourier直近傾き > 0
- 相対Fourier直近傾き > 0
- 相対比率 >= 1.0
- 日経Fourier直近傾き >= 0
- 売買代金 / 20日中央値 >= 1.2

閾値は `config.yaml` で変更できます。意地でも候補を出す設計ではありません。該当ゼロは正常です。

## バックテスト集計
平均ではなくmedian中心です。
- 取引数
- +5%到達数・到達率
- 到達日数median
- 実現リターンmedian
- MFE median
- MAE median
- 勝率
- 週あたり取引頻度
- 最大ドローダウン（参考複利曲線）

## 構成
```text
stock_fourier_backtest/
├── main.py
├── config.example.yaml
├── requirements.txt
├── src/
│   ├── data.py
│   ├── fourier.py
│   ├── features.py
│   ├── analysis.py
│   ├── backtest.py
│   └── report.py
└── tests/
    ├── test_fourier.py
    └── test_backtest.py
```

