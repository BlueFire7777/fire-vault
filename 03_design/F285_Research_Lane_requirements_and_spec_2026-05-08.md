---
title: F285 Research Lane 中長期銘柄発掘エンジン 仕様書 + 要件定義 v1.1
date: 2026-05-08
phase: F285-R0
status: 仕様設計完了 (v1.1、HQ 5 点修正 + Fujiwara 初期要求 7 項目対応表反映、Phase R0 feasibility precheck 着手前)
related: F281_Lane_C_design_2026-05-08, F210_phase_1b, F200_3レーン戦略構造, F119_Evaluation, F101_TDnet, F100_J-Quants
trigger: HQ 並行作業指示 (2026-05-08、F284/F105 c6 backfill PID 92822 走行中の vault 作業)
---

# F285 Research Lane 中長期銘柄発掘エンジン 仕様書 + 要件定義 v1.1

★ 本仕様書は **中長期 Lane (持ち越し前提)** 専用、デイトレ Lane
   (A/B/C) と別系統。HQ 必須 15 項目 + メタ情報を網羅。HQ 5 点修正
   反映済 (v1.0)。**v1.1**: Fujiwara 初期要求 7 項目との対応表を
   §1.1 に追加 + 各 Agent 役割文に要求番号を明記 (HQ 追加指示
   2026-05-08)。

## 1. Research Lane の役割定義

FIRE 既存の短期 Lane (A デイトレ / B スイング数日 / C 当日中初押し) と
別系統の **中長期持ち越し前提** リサーチ機能。Fujiwara の中長期
ポートフォリオ判断 (1 ヶ月 〜 6 ヶ月以上) を支援する。

主要発掘軸:
1. セクター資金循環 (= Sector Flow)
2. シクリカル業種のバリュー底打ち (= Cyclical Value)
3. 決算先回り (= Earnings Preview)
4. 増収増益増配 / 株主還元強化 (= Dividend Growth)

提案のみ、自動発注禁止 (R-03-01 / R-13-08 準拠)。Fujiwara が手動発注。

### 1.1 Fujiwara 初期要求 7 項目との対応 (HQ 確認用)

★ HQ 並行作業指示 (2026-05-08) に基づき、Fujiwara の **F285 初期要求
   7 項目** が本仕様書のどの章 / Agent / 要件 ID に対応するかを明示。
   F285 は単なる「リサーチレーン」ではなく、Fujiwara が要望した中長期
   企業発掘エンジンとして位置付ける。

| # | Fujiwara 初期要求 | F285 内の対応 Agent / 章 | 要件 ID | 不足有無 |
|---|---|---|---|---|
| 1 | 今後資金循環されるセクターを予想する | Sector Flow Agent / §4-1 | R-285-02 | なし |
| 2 | そのセクター内のシクリカルバリュー株を探す | Cyclical Value Screener / §4-2 | R-285-03 | なし |
| 3 | 増収増益になりそうな銘柄を探す | Earnings Preview Agent §4-3 (判定 2: 過去 4 期業績トレンド = 増収増益) | R-285-04 | なし |
| 4 | 増配しそうな銘柄を探す | Dividend Growth Agent / §4-4 (判定 1-3: 連続増配 + 増配率 + 配当性向) | R-285-05 | なし |
| 5 | 決算が良さそうな銘柄を先回りで発掘する | Earnings Preview Agent / §4-3 | R-285-04 | なし |
| 6 | 中長期の企業リサーチを FIRE に組み込む | Research Lane 全体 / §1 + §2 + §11 | R-285-01, R-285-13 | なし |
| 7 | 短期 Lane だけでなく、ポートフォリオ判断・毎朝レポート・LINE 通知に接続する | §8 (Lane C) + §9 (Portfolio) + §10 (毎朝レポート/LINE) | R-285-08, R-285-09, R-285-10 | なし |

各要求の網羅状況:
- **要求 1-2 (セクター + シクリカル)**: §4-1 + §4-2 で Agent 化、weighted_score の構成要素として §6 に組込
- **要求 3-4 (増収増益 + 増配)**: §4-3 (Earnings Preview の業績トレンド判定) と §4-4 (Dividend Growth) で個別 Agent 化、Watchlist Ranker §4-5 で統合格付け
- **要求 5 (決算先回り)**: §4-3 の核心ロジック (発表予定日 5-15 営業日先 + 4 期業績トレンド + TDnet 前兆指標 + アナリスト予想比較)
- **要求 6 (FIRE 組込)**: §1 役割定義 + §2 戦略仮説 + §3 Lane 間関係 + §11 Phase R0-R6 着手計画で全体設計
- **要求 7 (3 接続先)**: §8 Lane C 5 段階接続 (修正 2) + §9 Portfolio 接続 + §10 毎朝レポート/LINE (修正 4 で短期エントリー部屋と分離)

★ 結論: **Fujiwara 初期要求 7 項目すべてを F285 仕様 v1.0 で網羅**、
   不足なし。各要求は対応 Agent / 章 / 要件 ID で追跡可能。

## 2. 戦略仮説 + 全体アーキ図

仮説: 高再現性パターンの蓄積は短期 Lane に有効、一方中長期は
ファンダ × マクロ × セクター循環で「先回り」できる銘柄を選ぶ
"narrative-driven research" が有効。FIRE のエッジは再現性優位だが、
中長期は **ファンダ整合性 × マクロ整合性 × 公開情報先回り** で
別軸のエッジを構築する。

```
[情報源]
  J-Quants daily 四本値
  J-Quants /fins/details / dividend (契約状況により取得可否)
  J-Quants /fins/announcement (F101 既)
  TDnet announcement (F101 Phase 2/3 既)
  業種コード (TOPIX 33 + J-Quants 17)
  指数四本値 (F104 候補)
        ↓
[Research Lane (本仕様)]
  Sector Flow Agent
  Cyclical Value Screener
  Earnings Preview Agent
  Dividend Growth Agent
        ↓ (各 Agent が banner score 出力)
  Watchlist Ranker
        ↓ (weighted score + 主要 Agent 強度で A1/A2/B/C/D 格付け)
  Watchlist Top10 (banner / 根拠 Agent 含む)
        ↓
[出力]
  毎朝 8:30 JST 朝レポート (Research 専用 LINE 部屋 or 朝レポート枠)
  Portfolio Engine 接続 (現 holding 評価 + 追加 / 縮小提案)
  短期 Lane C 接続 (A1/A2 boost / B neutral / C risk flag / D HQ レビュー)
        ↓
[Fujiwara]
  手動発注 / 手動クローズ判断 (R-03-01)
        ↓
[Evaluation Agent (F119)]
  月次レビュー (1m/3m/6m リターン、TOPIX 相対、業種指数相対、Sharpe 等)
```

## 3. Lane 間関係

| Lane | 期間 | 発注頻度 | エッジ源 | Research Lane との関係 |
|---|---|---|---|---|
| A デイトレ | 当日中 | 1-3 銘柄/日 | 再現性 | 不接続 (期間軸違い) |
| B スイング | 数日 〜 数週 | 数銘柄/週 | 再現性 + 業績 | 不接続 (期間軸違い、業績軸が前哨候補) |
| C 当日初押し | 当日中 | 数銘柄/日 | 再現性 | **接続** (Rank 別 5 段階接続、§8) |
| **Research** | **1 ヶ月 〜 6 ヶ月** | **月次 watchlist 更新** | **ファンダ + マクロ** | **本 Lane** |

短期 Lane との競合回避: Research が抽出する銘柄が短期 Lane で当日
trade されることは妨げない (Lane 違いの独立判断)。Lane C との接続は
§8 の 5 段階で規定。

## 4. Agent 構成

### 4-1. Sector Flow Agent

役割: **今後資金循環されるセクターを予想する** (Fujiwara 初期要求 1)。
業種別資金循環パターンを抽出、強い業種を up-flow / 弱い業種を
down-flow として日次・週次にラベリング、Cyclical Value Screener
(§4-2) や Watchlist Ranker (§4-5) の入力となる。

入力データ:
- 全銘柄 daily 四本値 (J-Quants Standard で取得可)
- 業種コード (TOPIX 33 業種 + J-Quants 17 業種)

判定ロジック:
1. 業種別売買代金集計 (1 営業日)
2. SMA20 / SMA60 比較で資金流入率算定
3. 上位 5 業種 = up-flow / 下位 5 業種 = down-flow
4. up-flow 業種内の銘柄に Sector Flow Agent score を加点

出力: `{ sector_id, sector_name, flow_score: -1.0..+1.0, classification: up/neutral/down }`
銘柄単位 score: `flow_score = 0.4 * sector_score + 0.3 * sector_momentum + 0.3 * relative_breadth` (0.0-1.0 にクリップ)

### 4-2. Cyclical Value Screener

役割: **セクター内のシクリカルバリュー株を探す** (Fujiwara 初期要求 2)。
Sector Flow Agent (§4-1) で up-flow と判定された業種、特にシクリカル
業種 (海運 / 鉄鋼 / 化学 / 機械 / 自動車 等) でバリュエーション
底打ちを検出する。

入力データ:
- 全銘柄 daily 四本値 (J-Quants Standard で取得可)
- PBR / PER (J-Quants /fins/details 系、契約状況による、§5 + Phase R0 precheck)
- 業種別 PBR / PER 中央値 (Phase R1 で算出)

判定ロジック:
1. シクリカル業種フラグ (海運 / 鉄鋼 / 化学 / 機械 / 自動車 等)
2. PBR が業種中央値の 0.7 倍以下 + PBR < 1.0
3. PER が業種中央値の 0.8 倍以下 (赤字除外)
4. 直近 60 営業日で底打ちパターン (W 字 / 二番底 / 横ばい後の出来高
   増加等)

出力: `{ code, value_score: 0.0..1.0, bottom_pattern: W/double/none, pbr, per, sector_id }`
score 内訳: `value_score = 0.4 * pbr_disc + 0.3 * per_disc + 0.3 * bottom_pattern_strength`

### 4-3. Earnings Preview Agent

役割: **決算が良さそうな銘柄を先回りで発掘** + **増収増益候補の抽出**
(Fujiwara 初期要求 3, 5)。決算発表前 N 営業日の銘柄を先回り抽出し、
過去 4 期の業績トレンド (増収増益) を判定軸の中核に据える。

入力データ:
- TDnet announcement (F101 Phase 2 既、決算予告含む)
- J-Quants /fins/announcement (F101 Phase 1 既)
- 業績ヒストリー (J-Quants /fins/statements 系、§5 + Phase R0 precheck)
- アナリストコンセンサス (商用 API or 代替手段、§5 + Phase R0 precheck)

判定ロジック:
1. 決算発表予定日が 5 〜 15 営業日先
2. 過去 4 期の業績トレンド (増収増益 / 上方修正履歴)
3. 直近の TDnet 上方修正 / 自社株買い等の前兆指標
4. 自社ガイダンスとアナリスト予想中央値の比較 (取得可能な範囲で)

出力: `{ code, preview_score: 0.0..1.0, expected_announce_date, surprise_potential, last_4q_trend }`
score 内訳: `preview_score = 0.4 * trend_score + 0.3 * announcement_signal + 0.3 * surprise_potential`

### 4-4. Dividend Growth Agent

役割: **増配しそうな銘柄を探す** + **株主還元強化銘柄を抽出**
(Fujiwara 初期要求 4)。連続増配 / 増配率 / 配当性向 / 自社株買い・
株式分割の前兆 announcement で複合判定。

入力データ:
- 配当履歴 (J-Quants /fins/dividend 系、§5 + Phase R0 precheck)
- 自社株買い announcement (TDnet F101 Phase 2 既)
- 株式分割 announcement (TDnet)

判定ロジック:
1. 過去 5 期連続増配
2. 直近の増配率 >= 5%
3. 配当性向 30% 〜 60% の健全レンジ (財務無理ない範囲)
4. 自社株買い / 株式分割の前兆 (announcement)

出力: `{ code, dividend_score: 0.0..1.0, consecutive_years, dividend_yield, payout_ratio, last_increase_pct }`
score 内訳: `dividend_score = 0.4 * consecutive + 0.3 * recent_rate + 0.3 * payout_health`

### 4-5. Watchlist Ranker

役割: 4 Agent 出力を統合し、A1/A2/B/C/D の 5 段階に格付け、
Top10 銘柄を抽出。

統合ロジック (HQ 修正 1 反映):

```
weighted_score = (w1 * flow_score
                + w2 * value_score
                + w3 * preview_score
                + w4 * dividend_score) * 100
```

初期 calibration: w1 = w2 = w3 = w4 = 0.25。Phase R5 受入評価で
weight を調整可能 (F119 提案 + Fujiwara 承認)。

主要 Agent 強度: 各 Agent の score >= 0.7 を「強シグナル」として
カウント。

格付けは §7 を参照。

## 5. 必要データ一覧 (HQ 必須⑦)

★ HQ 修正 3 反映: J-Quants 財務系 API / 配当履歴 / TDnet の必要範囲は
**Phase R0 feasibility precheck で別途確認**。Standard で取得可能な
範囲を最大化する設計を優先する。**追加課金が必要な場合は Fujiwara
判断必須**。

| データ | 用途 | 取得元 (推定) | プラン状況 (precheck 前) |
|---|---|---|---|
| daily 四本値 (全銘柄) | Sector Flow / Cyclical Value | J-Quants /equities/bars/daily | Standard 既 ✅ |
| 業種コード | Sector Flow | J-Quants /equities/master | Standard 既 ✅ |
| PBR / PER | Cyclical Value | J-Quants /fins/details (推定) | **Phase R0 precheck 要** |
| 業績ヒストリー | Earnings Preview | J-Quants /fins/statements (推定) | **Phase R0 precheck 要** |
| 配当履歴 | Dividend Growth | J-Quants /fins/dividend (推定) | **Phase R0 precheck 要** |
| 決算予告 | Earnings Preview | J-Quants /fins/announcement (F101 既) | Standard 既 ✅ |
| TDnet announcement | Earnings Preview / Dividend | TDnet (F101 Phase 2 既) | 既 ✅ |
| TOPIX / 業種指数 | 受入評価 | J-Quants /indices/bars/daily (F276 既) | Standard 既 ✅ |
| アナリストコンセンサス | Earnings Preview (補助) | 商用 API or 代替 (例: 公開有報) | **Phase R0 precheck 要、追加課金は Fujiwara 判断** |

precheck で Standard 範囲を確定後、不足分は以下の手順:
1. Standard 範囲内で代替手段検討 (TDnet 既 + 公開有報 PDF パース等)
2. それでも不足する場合のみ Fujiwara に課金判断を仰ぐ

## 6. スコアリング項目 (HQ 必須⑧)

各 Agent score (0.0-1.0):

| Agent | score 構成 | 強シグナル閾値 |
|---|---|---|
| Sector Flow | sector_score 0.4 + sector_momentum 0.3 + relative_breadth 0.3 | >= 0.7 |
| Cyclical Value | pbr_disc 0.4 + per_disc 0.3 + bottom_pattern_strength 0.3 | >= 0.7 |
| Earnings Preview | trend_score 0.4 + announcement_signal 0.3 + surprise_potential 0.3 | >= 0.7 |
| Dividend Growth | consecutive 0.4 + recent_rate 0.3 + payout_health 0.3 | >= 0.7 |

統合 weighted_score (Watchlist Ranker):

```
weighted_score = (0.25 * flow_score
                + 0.25 * value_score
                + 0.25 * preview_score
                + 0.25 * dividend_score) * 100
```

Phase R5 受入評価で各 weight (w1-w4) を調整可能 (F119 提案 + Fujiwara
承認)。weight 合計は 1.0 に正規化。

## 7. A1/A2/B/C/D ランク定義 (HQ 必須⑨ + 修正 1)

★ HQ 修正 1 反映: hit 数だけでなく **weighted_score (総合点) AND
   主要 Agent 強度** の AND 条件で判定。両方を満たす場合のみ A1/A2
   認定。

| Rank | 条件 (AND) | 推奨アクション (Fujiwara 手動) |
|---|---|---|
| **A1** | weighted_score **>= 80** AND **主要 Agent (score >= 0.7) が 2 つ以上** | 強い買い候補、初期 entry 検討 |
| **A2** | weighted_score **>= 65** AND **主要 Agent が 1 つ以上** | 買い候補、押し目で entry 検討 |
| **B** | weighted_score **35 〜 65** (主要 Agent 0 〜 1) | neutral / 観察継続 |
| **C** | weighted_score **15 〜 35** (主要 Agent 0) | caution、優先度低下 / 追加条件要求 |
| **D** | weighted_score **< 15** (主要 Agent 0) | avoid、原則除外 |

A1/A2 銘柄は watchlist 上で **どの Agent が強シグナルか** を必ず明示
(例: `[Sector Flow ★ / Cyclical Value ★]`)。根拠の透明性を確保。

A1 と A2 の境界 (weighted_score 65-80 で主要 Agent 2 以上) は A2 扱い
(score 不足で A1 不適格)。weighted_score 80 以上でも主要 Agent 1 つ
だけなら A2 扱い。

## 8. 短期 Lane C との接続 (HQ 必須⑩ + 修正 2)

★ HQ 修正 2 反映: 「Research C/D で排除」ではなく、5 段階で接続。

Lane C (前日強銘柄初押し戦略、F281) との接続:

| Research Rank | Lane C 候補に対する効果 |
|---|---|
| **A1 / A2** | **confidence boost** (Lane C confidence × 1.2 等、calibration は Phase R5 で調整) |
| **B** | **neutral** (Lane C 判断に影響なし) |
| **C** | **risk flag** (Lane C 候補としての優先度低下、追加条件 = 当日強い momentum / 出来高急増 / 明確材料 等を要求) |
| **D** | **原則除外**、ただし以下の場合は **HQ レビュー対象**: |
| | - 当日主役テーマに該当 (主要ニュース / セクターローテ等) |
| | - 出来高急増 (前日比 3 倍以上 等) |
| | - 明確な材料あり (TDnet 当日重要発表等) |

D 銘柄が HQ レビュー経由で Lane C trade される場合、その回ごとに
Fujiwara 判断 + 後日 review (F119 月次評価で抽出 + 統計検証)。

## 9. Portfolio Engine との接続 (HQ 必須⑪)

Portfolio Balance Agent (R-08-XX 系) との接続:

| 状況 | Research Lane の役割 |
|---|---|
| 現 holding 評価 | 各 holding の Research Rank を表示 (A1〜D) |
| 追加 entry 判断 | A1/A2 銘柄を追加候補として Portfolio に提案 |
| 縮小 / exit 判断 | C/D 落ちした holding を縮小 / exit 提案 (Fujiwara 手動承認) |
| セクター集中リスク | up-flow セクター内の Portfolio 比率を提案表示 |

提案のみ、自動発注禁止 (R-03-01 / R-13-08 準拠)。Fujiwara が手動承認
してから手動発注。

## 10. 毎朝レポート / LINE 通知との接続 (HQ 必須⑫ + 修正 4)

★ HQ 修正 4 反映: LINE 通知先は「LINE エントリー部屋」**ではなく**、
**Research Lane 専用 LINE 部屋 (新設)** または **朝レポート枠 (= REPORT
部屋に Research セクション追加)** で配信。短期エントリー通知 (Lane
A/B/C) と中長期 Research 通知は **混同しない設計** とする。

朝レポート構成:
- 配信時刻: 毎朝 **8:30 JST** (寄付き 30 分前)
- 配信先: Research Lane 専用 LINE 部屋 / 朝レポート枠 (Phase R4 で
  HQ 承認、本仕様では選択肢を提示)

朝レポート内容:
1. Research Lane Watchlist Top10 (Rank A1/A2 を太字、B 以下は参考)
2. 各銘柄の根拠 (どの Agent が強シグナルか、ranked badges)
3. 業種別資金循環サマリ (up-flow 5 業種 / down-flow 5 業種)
4. 注目決算予告 (2 営業日先〜10 営業日先の Earnings Preview)
5. Portfolio 接続: 現 holding の Research Rank 一覧

LINE 通知部屋の選択肢 (Phase R4 で HQ 承認):

| 案 | 内容 | 利点 | 欠点 |
|---|---|---|---|
| **案 A** | Research 専用 LINE 部屋 (新設、6 部屋目) | 短期と完全分離、Research 集中 | 部屋追加 (LINE bot 設定変更) |
| **案 B** | 朝レポート枠 (= REPORT 部屋に Research セクション追加) | 既存部屋活用 (5 部屋維持) | REPORT に複数コンテンツ混在 |

短期 Lane の ENTRY 部屋には Research 通知を**入れない** (混同回避)。

## 11. 将来実装 Phase 分割 (HQ 必須⑬)

### Phase R0: 仕様設計 + Universe / data feasibility precheck

- 仕様書 (本ドキュメント) ✅ 完成
- data feasibility precheck (J-Quants Standard で取得可能な範囲確定):
  - /fins/details (PBR/PER) が Standard で動作するか実検証
  - /fins/statements (業績ヒストリー) が Standard で動作するか実検証
  - /fins/dividend (配当履歴) が Standard で動作するか実検証
  - 不足分は Standard 範囲内で代替手段検討、不足ならば Fujiwara 判断
- 業種コードマスタの取得経路確定
- 受入: Phase R1-R4 の実装に必要なデータが揃うことを確認

### Phase R1: Sector Flow Agent + Cyclical Value Screener

- J-Quants Standard daily で動く 2 Agent を実装
- ローカル DB (intraday と別 schema) に業種別集計を保存
- 受入: 60 営業日分で Sector Flow / Cyclical Value が安定動作、
  false positive 率 < 30%

### Phase R2: Earnings Preview Agent + Dividend Growth Agent

★ HQ 修正 3 反映: Premium 前提は断定しない。Phase R0 precheck 結果で
取得可能 API のみ使う、不足分は Standard 範囲内で代替手段を検討
(例: TDnet 既 + 公開有報 PDF パース等)。**追加課金が必要な場合は
Fujiwara 判断必須**。

- 取得可能データに基づく Agent 実装
- 受入: 決算予告 N 件 / 月、増配 announcement N 件 / 月の検出

### Phase R3: Watchlist Ranker 統合 + A1/A2/B/C/D 格付け

- R1 + R2 出力を weighted_score で統合
- 修正 1 準拠の rank 閾値で銘柄分類
- Top10 抽出ロジック (rank A1 優先 / 次 A2 / ...)
- 受入: A1/A2 銘柄が N 銘柄 / 月以上抽出可能、根拠 Agent 表記必須

### Phase R4: 毎朝レポート / LINE 通知 / Portfolio 接続

- 8:30 JST 配信 (Research 専用部屋 or 朝レポート枠、HQ 承認)
- Portfolio Balance Agent と接続 (現 holding Rank 表示 + 追加/縮小提案)
- 短期 Lane C との接続実装 (修正 2 準拠の 5 段階)
- 受入: 毎朝の LINE 配信が安定稼働、Portfolio 接続も動作

### Phase R5: 受入評価 (3-6 ヶ月実運用、shadow mode)

- 修正 5 反映の 7 + α 指標で評価 (§12 受入基準を参照)
- Fujiwara 受容判定

### Phase R6: Live 接続 (Phase R5 受入合格後)

- 朝レポート Live 配信
- Evaluation Agent (F119) 月次評価対象に組込

## 12. 受入基準 (HQ 必須⑭ + 修正 5)

### Phase R0 受入

- data feasibility precheck で必要 data の取得可否確定
- Standard プランで取得可能な範囲を最大化
- 追加課金が必要な API を特定 (Fujiwara 判断対象として明記)

### Phase R1 受入

- Sector Flow / Cyclical Value Agent が 60 営業日分で安定動作
- false positive 率 < 30% (= rank A2 以上で 30% 以上が翌週 -5% 等)

### Phase R2 受入

- Earnings Preview / Dividend Growth Agent が動作
- 決算予告 N 件 / 月、増配 announcement N 件 / 月の検出

### Phase R3 受入

- Watchlist Top10 が rank A1/A2 で N 銘柄 / 月以上抽出可能
- 主要 Agent 強度の根拠表記が watchlist に明示

### Phase R5 受入 (本番運用前の最終評価) — HQ 修正 5 反映

| # | 指標 | 評価方法 | 受入閾値 (案、Phase R5 で最終調整) |
|---|---|---|---|
| 1 | **Top10 1 ヶ月リターン** | 月次平均 | 市場平均超 (TOPIX +1% 以上) |
| 2 | **Top10 3 ヶ月リターン** | 3 ヶ月平均 | 市場平均超 (TOPIX +3% 以上) |
| 3 | **Top10 6 ヶ月リターン** | 6 ヶ月平均 | 市場平均超 (TOPIX +5% 以上) |
| 4 | **TOPIX 相対リターン** | Top10 - TOPIX | 1m/3m/6m 各正値 |
| 5 | **業種指数相対リターン** | Top10 - 該当業種指数 | 業種内で +α アウトパフォーム |
| 6 | **最大下落率** | watchlist 期間中の最大 drawdown | -15% 以下 |
| 7 | **決算後ギャップ勝率** | Earnings Preview 銘柄の決算後翌日 GAP UP 率 | 55% 以上 |
| 8 | **増配 / 上方修正的中率** | Dividend Growth / Earnings Preview の announcement 的中率 | 60% 以上 |
| 9 | **買わなかった候補の機会損益** | rank C/D 銘柄の事後リターン vs A1/A2 | A1/A2 と統計的有意な差 (selection bias 検証) |
| 10 | **Sharpe ratio** | 全 watchlist 6ヶ月 | 1.0 以上 |
| 11 | **winrate × 期待値** | watchlist 単位 | 期待値 +α |
| 12 | **Fujiwara 受容** | 主観評価 | 受容明示 |

各指標は F119 Evaluation Agent (R-13-XX 系) で月次測定、Phase R5
期間中の集計で受入判定。

## 13. リスクと制約 (HQ 必須⑮)

### リスク

1. **Premium プラン契約が必要となる可能性** (修正 3 反映): Phase R0
   feasibility precheck で Standard 取得範囲を確定後、不足分は追加
   課金 (Fujiwara 判断必須)。precheck 結果次第で Phase R2 設計が
   変わる。
2. **業種分類の曖昧性**: TOPIX 33 業種 + J-Quants 17 業種の使い分け、
   仮想業種 (例: 半導体材料) のカバーが不完全。
3. **中長期評価には時間**: 受入評価まで 3-6 ヶ月、データ蓄積が必要。
4. **selection bias 検証**: 買わなかった C/D 銘柄が後で大化けする
   ケースの統計補正 (機会損益測定で対応、§12 # 9)。
5. **アナリストコンセンサス取得**: 商用 API なしで代替手段が困難、
   Earnings Preview の精度低下リスク。
6. **短期 Lane C との設計接続の安全性**: Research D 銘柄が短期 C で
   trade される場合の HQ レビュー判断 (修正 2)、判断遅延リスク。

### 制約

1. **自動発注禁止** (R-03-01 / R-13-08): 提案のみ、Fujiwara 手動発注。
2. **持ち越し前提**: 短期 Lane と期間軸が違う、Lane 間の独立性を保つ。
3. **Stage 飛ばし禁止** (FIRE 全体ルール): Phase R5 受入合格まで本番
   運用しない。
4. **Evaluation Agent 提案制** (R-13-08): rank 閾値 / weight 等の
   調整は F119 提案 + Fujiwara 承認。
5. **LINE 通知混同禁止** (修正 4): 短期エントリー部屋に Research
   通知を入れない。
6. **追加課金 Fujiwara 判断必須** (修正 3): Premium / 商用 API 等の
   課金は Fujiwara 個別判断。

## 14. 要件 ID 表

| ID | 内容 | 関連章 |
|---|---|---|
| R-285-01 | Research Lane は中長期 (持ち越し有り) 専用、デイトレ Lane に侵食しない | §1, §3 |
| R-285-02 | Sector Flow Agent: 業種別資金循環抽出 | §4-1 |
| R-285-03 | Cyclical Value Screener: シクリカルバリュー底打ち検出 | §4-2 |
| R-285-04 | Earnings Preview Agent: 決算先回り抽出 | §4-3 |
| R-285-05 | Dividend Growth Agent: 増配局面・株主還元 | §4-4 |
| R-285-06 | Watchlist Ranker: 4 Agent 出力統合 + 5 段階格付け | §4-5 |
| R-285-07 | A1/A2/B/C/D 定義 = weighted_score AND 主要 Agent 強度 (修正 1) | §7 |
| R-285-08 | 朝レポート: Research 専用 LINE 部屋 or 朝レポート枠、ENTRY 部屋と混同禁止 (修正 4) | §10 |
| R-285-09 | Portfolio Engine 接続: 現 holding Rank 表示 + 追加/縮小提案 | §9 |
| R-285-10 | Lane C 接続 5 段階 (修正 2): A1/A2 boost / B neutral / C risk flag / D HQ レビュー | §8 |
| R-285-11 | 自動発注禁止、提案のみ (R-03-01 / R-13-08 準拠) | §13 |
| R-285-12 | 必要データ: Standard 範囲を Phase R0 precheck で確定、追加課金 Fujiwara 判断 (修正 3) | §5 |
| R-285-13 | Phase R0-R6 段階着手、各 Phase 受入合格まで次 Phase 進まない | §11, §12 |
| R-285-14 | F119 評価対象、月次レビュー必須 (修正 5: 12 指標) | §12 |

## 15. 関連 Vault ファイル / 改訂履歴

### 関連

- [[F281_Lane_C_design_2026-05-08|F281 Lane C 設計]] — 短期 Lane C 接続先 (§8 / 修正 2)
- [[F210_phase_1b|F210 Phase 1B]] — 3000 万円 2 年目標、Research も含む長期視点
- [[F119_Evaluation|F119 Evaluation Agent]] — 受入評価実施者 (§12)
- [[F101|F101 TDnet]] — Earnings Preview / Dividend Growth のソース
- [[F100|F100 J-Quants V2]] — daily / fins データソース
- [[F200|F200 3 レーン戦略構造]] — Lane 全体設計
- [[F276_indices_spec_2026-05-05|F276 指数 spec]] — 受入評価の業種指数
- [[F285_Research_Lane中長期銘柄発掘エンジン|F285 TODO]] — Phase 着手計画

### 改訂履歴

- v1.0 (2026-05-08): 初版、HQ 並行作業指示中の vault 化、HQ 5 点修正
  反映 (A1/A2 weighted+strength / Lane C 5 段階 / Phase R2 Premium
  非断定 / LINE 別部屋 / 受入 12 指標)
- v1.1 (2026-05-08): HQ 追加指示 (Fujiwara 初期要求 7 項目との対応
  明示) を反映:
  - §1.1 「Fujiwara 初期要求 7 項目との対応」表を追加 (HQ 確認用、
    要求 1-7 → Agent / 章 / 要件 ID 一目で追跡可能)
  - §4-1 〜 §4-4 各 Agent 役割文に Fujiwara 要求番号を明記
    (要求 1 → §4-1 / 要求 2 → §4-2 / 要求 3,5 → §4-3 / 要求 4 → §4-4)
  - 結論: 不足なし、Fujiwara 初期要求 7 項目すべてを v1.0 で網羅済、
    v1.1 で対応の明示性を強化
