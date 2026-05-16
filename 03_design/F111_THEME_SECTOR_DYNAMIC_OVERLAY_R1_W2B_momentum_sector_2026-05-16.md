# FIRE F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W2-B momentum / sector_relative_strength simulation (2026-05-16)

doc_id: FIRE-F111-OVERLAY-W2B-MOMENTUM-SECTOR-2026-05-16
status: read-only simulation 完了 (= helper + runner + 61 tests PASS、151 PASS regression、既存 production code 変更 0、DB write 0、LINE 0、API 0、token 0)
related:
- F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_2026-05-16.md (= W1 design 起源)
- F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_W2A_result_2026-05-16.md (= W2-A overlay 実装結果)
- F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_W2A_SMOKE_2026-05-16.md (= W2-A staging smoke)
- F062_W2A_OVERLAY_BUYABILITY_CARD_2026-05-16.md (= overlay card 実装)
- F062_OVERLAY_CARD_DAILY_FLOW_PREVIEW_2026-05-16.md (= daily flow preview)


## §1 目的

W2-A で表示可能になった **overlay top5 (= 9247/9628/4404/9008/3134)** に対し、
「9247/9628 が継続上昇主役か惰性固定か」を数値で判別する材料を提供する。

volume_momentum / turnover_momentum / price_momentum / sector_relative_strength を
staging DB から read-only で算出し、overlay top5 / 新顔 6 件 / 任意 entry 候補について
4 分類 (継続上昇主役 / 惰性固定注意 / 個別強気 / sector 強気 / neutral / insufficient) を出す。

★ 本 wave の鍵:
- LINE / 通知 / UX ではなく **銘柄選定品質を上げる wave**
- 9247/9628 を「消す」のではなく「継続上昇期待が薄ければ判断材料を提示する」


## §2 実装範囲

### §2.1 新規 file (= 3 件、既存 0 変更)

| file | 行数 | 役割 |
|---|---|---|
| scripts/jobs/_f111_overlay_momentum_sector.py | ~270 | helper module (= 純関数 9 + MomentumMetrics / SectorRelativeStrength / OverlayClassification dataclass + 4 分類定数) |
| scripts/jobs/run_f111_overlay_momentum_sector_sim.py | ~320 | runner (= staging DB URI mode=ro 接続 + 4 file 出力 + 2 path guard + 9 別 codes 一括処理) |
| tests/scripts/jobs/test_run_f111_overlay_momentum_sector_sim.py | ~480 | 61 tests (helper 純関数 27 + runner path guard 14 + smoke 8 + error 4 + safety 3 + その他) |

### §2.2 既存 file への変更 (= 0)

`git status --short` で確認:
- 3 untracked file のみ (`??` 状態)
- 既存 production code 一切不変 (= notifications/templates / scripts/jobs/_v1_4_1_* /
  scripts/jobs/run_f111_real_batch_staging.py / agents / etc 全保全)
- DB schema 変更 0、theme_tags table 追加 0


## §3 利用データ

| 項目 | 値 |
|---|---|
| staging DB | `/Users/bluefire/fire/data/fire.staging.db` (= URI mode=ro、immutable=1 推奨) |
| as_of_date | `2026-05-14` (= staging market_prices_daily 最新日付、自動検出) |
| 価格テーブル | `market_prices_daily` (= code / date / open / high / low / close / volume / turnover_value) |
| sector テーブル | `market_listings` (= code / company_name / sector_17_name / scale_category) |
| 価格データ深度 | 22-25 営業日分 (= MOMENTUM_LONG_DAYS 20 + buffer) |
| sector peer 数 | 情報通信 68 / 食品 29 / 運輸・物流 33 / 小売 33 (= 全 ≥ MIN 3 を満たす) |
| code format 変換 | 4-digit → 5-digit (= `_to_db_code()` で末尾 0 付加、J-Quants 形式) |


## §4 指標定義

### §4.1 momentum (個別銘柄)

| 指標 | 式 | 期間 |
|---|---|---|
| volume_momentum_5d | `latest_volume / avg_5d_volume - 1` | 5 営業日 |
| volume_momentum_20d | `latest_volume / avg_20d_volume - 1` | 20 営業日 |
| turnover_momentum_5d | `latest_turnover / avg_5d_turnover - 1` | 5 営業日 |
| turnover_momentum_20d | `latest_turnover / avg_20d_turnover - 1` | 20 営業日 |
| price_momentum_5d | `(latest_close / close_5d_ago) - 1` | 5 営業日 return |
| price_momentum_20d | `(latest_close / close_20d_ago) - 1` | 20 営業日 return |

fallback:
- avg ≤ 0 で 0 除算回避 → None
- latest / avg None → None
- 短い履歴 (= 5d 未満) で price_momentum None

### §4.2 sector_relative_strength

```
sector_relative_strength = individual_return - peer_median_return
```

- peer: 同 `sector_17_name` + 同 scale 帯 (TOPIX Core30 / Large70 / Mid400) で自分を除く
- individual_return = price_momentum_20d
- peer_returns = 各 peer の price_momentum_20d (= 20 営業日 return)
- peer 数 < `MIN_SECTOR_PEER_COUNT (=3)` で fallback (= srs=None / method=insufficient_peers)
- median (= 中央値) を使う (= 外れ値耐性、sector index 不在の代替)

### §4.3 4 分類 logic

neutral_band = ±5%:

| 条件 | 分類 (英) | 日本語ラベル |
|---|---|---|
| vol_mom > +5% かつ srs > +5% | `leader_continue` | 継続上昇主役候補 |
| vol_mom < -5% かつ srs < -5% | `leader_inertia` | 惰性固定注意 |
| vol_mom > +5% かつ srs < -5% | `individual_strong` | 個別強気 / sector 警戒 |
| vol_mom < -5% かつ srs > +5% | `sector_strong` | sector 強気 / 個別弱め (= 比較枠) |
| 両方とも band 内 (or 片方のみ band 越え) | `neutral` | neutral (= 中立、追加情報待ち) |
| vol_mom or srs is None | `insufficient_data` | データ不足 (= 判定保留) |

★ ±5% neutral_band の意図: 微小値での誤判別を防ぐ (= 沈降相場で srs ±1% を `leader_continue` と
誤分類しないため)。tuning は W2-C 以降のデータドリブン調整候補。


## §5 momentum 結果 (as_of_date = 2026-05-14)

### §5.1 overlay top5

| code | name | sector | vol_mom_5d | vol_mom_20d | tov_mom_20d | price_mom_5d | price_mom_20d |
|---|---|---|---|---|---|---|---|
| 9247 | ＴＲＥホールディングス | 情報通信 | -32.5% | **-35.9%** | -37.8% | +1.4% | -1.9% |
| 9628 | 燦ホールディングス | 情報通信 | +18.9% | +1.4% | +1.2% | +0.9% | +0.07% |
| 4404 | ミヨシ油脂 | 食品 | -4.9% | +3.9% | +0.2% | -1.5% | -4.9% |
| 9008 | 京王電鉄 | 運輸・物流 | +10.5% | **+24.9%** | +20.1% | -1.0% | -7.7% |
| 3134 | Ｈａｍｅｅ | 小売 | -46.9% | **-19.2%** | -33.2% | **-21.1%** | **-20.5%** |

### §5.2 新顔 6 件

| code | name | sector | vol_mom_20d | price_mom_20d |
|---|---|---|---|---|
| 3962 | チェンジホールディングス | 情報通信 | -17.8% | -1.8% |
| 6196 | ストライクグループ | 情報通信 | +1.2% | -11.9% |
| 7595 | アルゴグラフィックス | 情報通信 | **+35.1%** | -7.8% |
| 9417 | スマートバリュー | 情報通信 | **-95.9%** | -19.5% |
| 9008 | (overlay と同じ) | (上記) | (上記) | (上記) |
| 3134 | (overlay と同じ) | (上記) | (上記) | (上記) |

特筆:
- **9247 vol_mom -36%**: 出来高大幅減退 (= 主役継続に疑問符)
- **9008 vol_mom +25%**: 出来高顕著増 (= 新顔としての勢いあり)
- **3134 vol_mom -19%**: 出来高減 (= 新顔としても弱め)
- **9417 vol_mom -96%**: 出来高ほぼ消失 (= 朝判断から外す材料強い)


## §6 sector_relative_strength 結果

| code | sector | individual_return (20d) | peer_median (20d) | srs | peers | method |
|---|---|---|---|---|---|---|
| 9247 | 情報通信 | -1.9% | -0.5% | -1.4% | 68 | peer_median |
| 9628 | 情報通信 | +0.07% | -0.5% | +0.6% | 68 | peer_median |
| 4404 | 食品 | -4.9% | -6.1% | +1.1% | 29 | peer_median |
| 9008 | 運輸・物流 | -7.7% | -4.1% | -3.7% | 33 | peer_median |
| 3134 | 小売 | -20.5% | -5.8% | **-14.7%** | 33 | peer_median |
| 3962 | 情報通信 | -1.8% | -0.5% | -1.3% | 68 | peer_median |
| 6196 | 情報通信 | -11.9% | -0.5% | **-11.4%** | 68 | peer_median |
| 7595 | 情報通信 | -7.8% | -0.5% | -7.3% | 68 | peer_median |
| 9417 | 情報通信 | -19.5% | -0.5% | **-19.0%** | 68 | peer_median |

特筆:
- **3134 srs -14.7%**: 小売 sector 内で大幅劣位 → leader_inertia 強化
- **9417 srs -19.0%**: 情報通信 sector 内で最劣位 → leader_inertia 強化
- **9247 srs -1.4%**: 情報通信 sector とほぼ同水準 (= 出来高減は強いが、sector もソフト)


## §7 9247/9628 分類

| code | classification | label_jp | rationale |
|---|---|---|---|
| 9247 | **neutral** | neutral (= 中立、追加情報待ち) | \|srs\|=1.4% が band ±5% 内のため neutral 判定。出来高は明確に減退 (-36%) だが sector も全体 softening。 |
| 9628 | **neutral** | neutral (= 中立、追加情報待ち) | vol_mom +1.4% / srs +0.6% 両方とも band 内、判定保留。 |

解釈:
- **9247**: 「主役継続」と確証する材料は薄い (= 出来高 -36% は明確な weakness)。
  ただし srs band 内のため「惰性固定」とも断定不可。**追加指標 (= breadth / catalyst proximity) 必要**。
- **9628**: 完全に中立、W2-B 単独では判別不能。

朝判断への示唆:
- W2-A overlay で「主役継続枠」配置の **9247 は warning 寄りに表示する選択肢あり** (= vol_mom -36% を表示)
- 9628 は **追加情報待ちで現状維持** が穏当


## §8 新顔 / 新sector 分類

### §8.1 W2-A 新顔+新sector 比較枠 (overlay top5 配置)

| code | sector | vol_mom | srs | classification | 解釈 |
|---|---|---|---|---|---|
| 9008 | 運輸・物流 | +24.9% | -3.7% | **neutral** | 出来高顕著増だが srs band 内、半歩 individual_strong 寄り |
| 3134 | 小売 | -19.2% | -14.7% | **leader_inertia (惰性固定)** | W2-A 新顔配置と W2-B 数値判定は乖離。**選定見直し候補** |

### §8.2 新顔 6 件のうち overlay 未配置

| code | sector | vol_mom | srs | classification | 解釈 |
|---|---|---|---|---|---|
| 3962 | 情報通信 | -17.8% | -1.3% | neutral | 出来高減、sector 中立 |
| 6196 | 情報通信 | +1.2% | -11.4% | neutral | 個別中立、sector 劣位 |
| 7595 | 情報通信 | +35.1% | -7.3% | **individual_strong** | **出来高急増、sector 弱め — W2-C で overlay 候補** |
| 9417 | 情報通信 | -95.9% | -19.0% | **leader_inertia** | **朝判断除外材料、新顔から外す** |

### §8.3 W2-C 以降の入れ替え示唆

W2-B 数値判別を overlay 選定に組み込むなら:
- **入** : 7595 (individual_strong)
- **入** : 9008 (neutral だが vol_mom +25% で活発)
- **出** : 3134 (leader_inertia)
- **除外** : 9417 (leader_inertia 強烈、新顔候補から外す)


## §9 4404 扱い (= W2-A risk warning 枠との整合)

| 観点 | W2-A 評価 | W2-B 評価 | 整合性 |
|---|---|---|---|
| role | risk_warning_track (= 100 株 risk_yen 10665 が pilot 上限超過) | vol_mom +3.9% / srs +1.1% → **neutral** | ✓ 矛盾なし |

解釈: W2-A の risk warning は **risk_yen 上限超過の警告**、W2-B の neutral は **momentum/sector 指標で
中立**。2 つの軸は独立評価可能。両者並行表示が朝判断に有用。


## §10 9130 確認 (= excluded 維持)

| 確認項目 | 結果 |
|---|---|
| 9130 が target_codes に含まれるか | **含まれない** (= overlay/new face/extra いずれも未指定) |
| 9130 が classifications に出るか | **出ない** (= ロジック対象外) |
| 9130 が market_listings に存在するか | **存在 (excluded_recovery_code_confirmed_in_listings=True)** |
| 9130 が entry/watch に復帰するか | **しない** (= W2-A excluded_recovery_codes_active 維持、W2-B でも判定対象外) |


## §11 tests (= 61 新規、全 PASS、regression なし)

| クラス | 件数 | カバレッジ |
|---|---|---|
| TestComputeRatioMomentum | 6 | simple increase/decrease / None × 2 / zero avg / negative avg |
| TestComputePctChange | 4 | up / down / None / zero |
| TestComputeMomentumMetrics | 7 | n_days / latest / avg_5d / price_mom_5d / short_history / empty / volume_zero |
| TestComputeSectorRelativeStrength | 5 | above_median / below_median / insufficient_peers / None / peers_with_none |
| TestClassifyOverlayCandidate | 10 | 4 象限 + neutral × 2 + insufficient × 2 + rationale |
| TestCodeConversion | 3 | 4-digit / 5-digit (letter-suffix) / 3-digit |
| TestDbPathGuard | 4 | staging allowed / production refused / develop refused / unknown refused |
| TestOutputPathGuard | 8 | tmp / etc / data_db / repo_source / git / workflows / launchagents / fire_vault |
| TestRunnerSmoke | 8 | happy path / 3 file 生成 / metrics payload / classification payload / 9247 leader_continue pattern / 9628 leader_inertia pattern / 9130 not in classifications / md present |
| TestRunnerErrorPaths | 4 | refuse_production / refuse_develop / db_not_found / unsafe_output |
| TestSafetyImports | 3 | runner no dangerous / helper no dangerous / runner uses URI mode=ro |
| **合計** | **61** | **全 PASS** |

regression sanity: W2-B 61 + W2-A overlay card 51 + daily flow 39 = **151 PASS** (= 既存 90 不変、新 61 追加)


## §12 安全 gate (= 本 wave 行為)

| 項目 | 結果 |
|---|---|
| DB write (production / develop / staging) | 0 |
| schema migration | 0 |
| theme_tags table 追加 | 0 |
| LINE 送信 | 0 (= linebot SDK / push_message / reply_message 不使用) |
| API call (J-Quants / TDnet / 外部 HTTP) | 0 (= requests / aiohttp import 0) |
| token / channel_token / secret / .env 値参照 | 0 (= os.environ 不参照) |
| launchctl / plist 配置 / cron 編集 | 0 |
| workflow 変更 | 0 |
| git add / commit / push | 0 |
| PR 作成 / merge | 0 |
| VACUUM | 0 |
| sudo / rm -rf / --no-verify | 0 |
| auto-order / 楽天 / iSPEED / Computer Use | 0 |
| TODO Excel 更新 | 0 |
| 既存 production code 変更 | 0 (= git diff origin/develop で 3 untracked file のみ) |
| TestSafetyImports で independent 検査済 | linebot / requests / aiohttp / subprocess 全 import 0、helper は sqlite3 も import 0 |
| 出力先 | /tmp/fire_w2b_momentum_sector/ + ~/fire-vault/03_design/ (= 安全 prefix 限定) |
| DB 接続 mode | URI mode=ro のみ (= TestSafetyImports.test_runner_uses_uri_mode_ro で assert) |


## §13 Codex 4 lane 監査結果

| Lane | scope | verdict | severity |
|---|---|---|---|
| A | 指標定義 / calculation validity | **CONCERN** | CRITICAL 0 / HIGH 0、LOW: rationale 文字列の表現 (= 後述 §13.1) |
| B | 9247/9628 継続主役 vs 惰性固定分類 | **CONCERN → 解消** / RECOMMENDED_INTERPRETATION: **X1** | HIGH 1 検出 (= rationale 文字列のバグ) → 本 wave 内で **修正済** |
| C | 新顔/新sector 候補の評価妥当性 | **APPROVE** | CRITICAL/HIGH 0、軽微 CONCERN: 同 rationale 文字列 issue → §13.2 で fix 同時解消 |
| D | safety / read-only / W2-C 以降の切り分け | **APPROVE** / RECOMMENDED_NEXT: **5** | CRITICAL/HIGH 0 |

集約: HIGH 1 検出 → 本 wave 内で fix 済 (= rationale 文字列バグ)、CRITICAL 0、機能 / 安全への影響なし。
再 smoke 後 61 tests 全 PASS / regression 維持。

### §13.1 Lane A 詳細
- 指標式の単位整合 OK (= decimal return 同士)
- None / 0 除算 / 空 peer fallback OK
- runner 側で `code != exclude_code` で self を peer から除外 OK
- 9417 vol_mom -95.9% は計算上有効 (= latest 7,500 / avg 182,495 - 1 ≈ -0.959)
- peer 数 68 / 29 / 33 は median に十分
- **as_of_date=2026-05-14 vs 9417 最新行 2026-05-08** のズレあり (= 実装は date <= as_of_date で
  「以前」を取るため、5-08 が最新として扱われる、合理的だが doc に明記)
- **LOW**: rationale 文字列が「両方 band 内」と誤読しうる → §13.2 fix で同時解消

### §13.2 Lane B 詳細 (HIGH 検出 → 解消)
**HIGH 指摘**:
- 9247 の rationale `|vol_mom|=0.359 かつ |srs|=0.014 が neutral_band 0.05 以下 → 中立` が
  間違い (= vol_mom 0.359 は band 超え、srs 0.014 のみ band 内)

**本 wave 内対応 (= 修正済)**:
`_f111_overlay_momentum_sector.py::classify_overlay_candidate` の neutral 分岐を 4 場合分けに細分:
1. 両方 band 内 → 「両方が neutral_band 以下 → 中立」
2. vm band 内 / srs band 外 → 「srs 側は判別保留 → 片側 only、neutral 扱い」
3. srs band 内 / vm band 外 → 「vol_mom 側は判別保留 → 片側 only、neutral 扱い」 ← 9247 はこのパス
4. defensive fallback (= 数学上到達しない)

再 smoke 後の 9247 rationale:
```
|srs|=0.014 ≤ 0.05 (band 内) かつ vol_mom=-0.359 が band 外、
srs 側は判別保留 → 片側 only、neutral 扱い
```
→ 正確、61 tests 全 PASS、機能 logic 不変。

**Lane B 解釈推奨 (= 採用)**: **X1 = 9247 を warning 寄りに降格** (vol_mom -36% を重く見る)。
本 wave では実装変更せず、W2-C 以降の overlay 選定 logic 改修で反映候補。

### §13.3 Lane C 詳細
- 数値主張は JSON/MD と一致確認
- 3134 leader_inertia は W2-A 新顔配置と矛盾ではなく「新顔だが個別力なし」と読むのが自然
- 9008 は出来高 +24.9% / SRS -3.7% で **案 Y1 (neutral 維持) 推奨** (= sector 側のプラス確認不足で
  leader_continue 昇格には早い)
- 7595 individual_strong は W2-C で 3134 と入替候補
- 9417 leader_inertia 強烈、新顔から除外材料として強い
- 4404 risk warning と W2-B neutral は独立軸で矛盾なし
- 軽微 CONCERN: rationale 文字列 issue → §13.2 fix で解消済

### §13.4 Lane D 詳細
- helper: sqlite3 / API / LINE / file I/O / env / network すべてなし、純計算のみ
- runner: `file:{db_path}?mode=ro` + `uri=True`、DB write / migration / VACUUM / send 系なし
- 出力 write は guarded output dir の 3 file のみ
- production DB / develop DB は basename guard で refuse
- /tmp 実生成物 JSON/MD に forbidden token 不在
- git 状態: 新規 3 untracked のみ、send 系 / notifications / agents に tracked diff なし
- **RECOMMENDED_NEXT: 5 = paper_pnl 連携** (= W2-B label が実際のエッジに繋がるか実検証、即採用ではなく検証先行)


## §13.5 集約結論
- HIGH 1 検出 → 本 wave 内で fix 済 (= rationale 文字列、logic 不変、61 tests PASS 維持)
- CRITICAL 0、CONCERN は文字列精度のみ
- 9247/9628 解釈推奨 = X1 (= 9247 warning 降格、9628 neutral 維持)
- 次 wave 推奨 = 5 (= paper_pnl 連携で実エッジ検証)
- merge / safety / data 経路 への影響なし


## §14 結論 / W2-C 以降の切り分け

### §14.1 結論

- W2-B simulation は read-only で正常動作
- 9247/9628 は **neutral** (= 主役継続の数値根拠は薄いが、惰性固定とも断定不可)
- 9008 は neutral (vol_mom +25%)、3134 は **leader_inertia** (新顔配置と乖離)
- 7595 は **individual_strong** (新顔候補として有望)、9417 は **leader_inertia** (除外材料)
- DB write 0 / API 0 / LINE 0 / token 0 / theme_tags 追加 0
- 既存 production code 変更 0

### §14.2 W2-C 以降の切り分け (= 本 wave 範囲外)

1. **W2-B 判別を overlay 選定に組み込む** (= 表示変更 / role 再分類)
2. theme_tags table 追加 (= sector 横断テーマ管理)
3. **momentum / srs 閾値 tuning** (= ±5% でいいか、データドリブン調整)
4. breadth / catalyst proximity / value rotation 等の追加指標
5. paper_pnl との連携 (= W2-B 分類がエッジに繋がるか実検証)
6. cron / launchd 組み込み (= 朝 daily flow に W2-B sim を統合)

### §14.3 関連 file

```
新規 production:
  scripts/jobs/_f111_overlay_momentum_sector.py
  scripts/jobs/run_f111_overlay_momentum_sector_sim.py
新規 tests:
  tests/scripts/jobs/test_run_f111_overlay_momentum_sector_sim.py
出力 (= /tmp 限定):
  /tmp/fire_w2b_momentum_sector/momentum_sector_metrics.json
  /tmp/fire_w2b_momentum_sector/overlay_classification.json
  /tmp/fire_w2b_momentum_sector/overlay_classification.md
  /tmp/fire_w2b_momentum_sector/codex_lane_{A,B,C,D}_prompt.txt
  /tmp/fire_w2b_momentum_sector/codex_lane_{A,B,C,D}_result.txt
本 doc:
  ~/fire-vault/03_design/F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_W2B_momentum_sector_2026-05-16.md
```
