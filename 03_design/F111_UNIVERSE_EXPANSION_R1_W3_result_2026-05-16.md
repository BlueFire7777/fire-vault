---
id: FIRE-F111-UNIVERSE-EXPANSION-R1-W3-RESULT-2026-05-16
phase: 実装結果 doc (= W3 staging write 完了 + universe 拡張発現確認)
priority: 最高
status: staging write 完了 / F111 universe 実拡張発現確認 / production-develop 完全 untouched
owner: BlueFire7777 (Fujiwara)
date: 2026-05-16
parent_w1: 03_design/F111_UNIVERSE_EXPANSION_R1_W1_result_2026-05-16.md
parent_w2: 03_design/F111_UNIVERSE_EXPANSION_R1_W2_result_2026-05-16.md
codex_lanes: 4
critical_high_count: 0
hq_approve: HQ_APPROVE_STAGING_SIGNAL_PERSISTENCE_WRITE (= 1 回 staging write 限定)
---

# F111-UNIVERSE-EXPANSION-R1-W3 実装結果 (= 2026-05-16)

## §1 承認範囲

**HQ_APPROVE_STAGING_SIGNAL_PERSISTENCE_WRITE** 承認下:
- signal_persistence top_n=100 の結果を staging research_watchlist_signals に **1 回 write** 限定
- production / develop DB write: 禁止 ✓ (= untouched)
- LINE / API / token / launchctl / plist / cron / VACUUM / workflow / git: 全禁止 ✓
- staging write 後の universe 拡張効果を F111 post-write validation で確認

## §2 pre-write 状態

| 項目 | 値 |
|---|---|
| staging total rows | 13,730 |
| staging latest base_date | 2026-05-13 |
| staging latest count | 35 件 (= D48 と同) |
| W3 source_version records | 0 (= 未書込) |
| F111 raw (= D48 同条件 base_date=2026-07-21) | 35 件 |
| entry / watch / excluded | 14 / 0 / 21 |
| F111 sectors | 12 |
| entry sectors | 7 |

## §3 pre-write dry-run (= /tmp/fire_universe_w3/top100_dry_run.json)

- top_n: 100
- mode: dry-run (= write_enabled=False)
- inserted: 109 件 (= records 3,708 → top-n filter 100 + cap_deferred 9)
- row_count: 13,730 → 13,730 (= 不変、dry-run 確認)
- safety: DB write 0 / LINE 0 / API 0 / token 0 / launchctl 0 ✓

## §4 staging write 結果 (= 承認範囲内 1 回限定)

**コマンド**:
```
FIRE_ENV=staging .venv/bin/python -m scripts.jobs.run_research_watchlist_signal_persistence \
  --db staging --base-date 2026-07-22 \
  --source-version w60_universe_expansion_r1_w3_top100 \
  --top-n 100 --write \
  --output-json /tmp/fire_universe_w3/top100_write.json
```

**結果**:
| 項目 | 値 |
|---|---|
| mode | **write** (= write_enabled=True) |
| inserted | **109** |
| replaced / skipped / failed | 0 / 0 / 0 |
| row_count | 13,730 → **13,839** (= +109) |
| target table | research_watchlist_signals (= SIGNALS_TABLE) |
| base_date | 2026-07-22 |
| source_version | w60_universe_expansion_r1_w3_top100 |
| staging-strict 4 段 guard | 全 PASS ✓ |

## §5 post-write staging 状態

| 項目 | pre-write | post-write | 変化 |
|---|---|---|---|
| total rows | 13,730 | **13,839** | +109 ✓ |
| latest base_date | 2026-05-13 | **2026-07-22** ✓ |
| latest count | 35 | **109** ✓ |
| W3 source records | 0 | 109 ✓ |
| **staging DB md5** | 6cb3885c... | **f6fa86df...** ✓ (変化、想定通り) |
| staging DB size | 4,804,108,288 | **4,804,222,976** (= +114,688 byte) |
| staging mtime | 5/14 17:14 | **5/16 14:29** (= write 反映) |
| **production DB md5** | b1df4673... | **b1df4673...** (= 不変) ✓ |
| **develop DB md5** | 0eed4ad2... | **0eed4ad2...** (= 不変) ✓ |
| F282 plist 3 file | 1772/1844/1772 | **1772/1844/1772** (= 不変) ✓ |

→ **承認範囲内、production/develop/F282 plist は完全 untouched** ✓

## §6 F111 post-write 結果 (= /tmp/fire_universe_w3/f111_after_top100_write.json)

| 項目 | D48 (pre-write) | W3 (post-write) | 変化 |
|---|---|---|---|
| F111 cli_version | 1.3.0 | 1.3.0 | 維持 |
| F111 max_candidates | 50 | 50 | 維持 |
| policy_version | 1.4.2 | 1.4.2 | 維持 |
| base_date | 2026-07-21 | 2026-07-22 | (= W3 用) |
| **F111 raw** | 35 | **50** | **+15 (= +43%)** ✓ |
| **entry** | 14 | **20** | **+6 (= 新顔)** ✓ |
| watch | 0 | 0 | 維持 |
| **excluded** | 21 | **30** | +9 |
| **F111 sectors** | 12 | **13** | +1 |
| **entry sectors** | 7 | **9** | **+2** ✓ |

## §7 universe 拡張発現

### §7.1 entry sectors 拡張 (= +2)
- 旧 7 sector: 情報通信 / 商社・卸売 / 不動産 / 食品 / 金融 / 素材化学 / 医薬品
- **新規追加 2 sector**: **運輸・物流** / **小売**
- 合計 9 sector

### §7.2 entry 14 → 20 件 (= 新顔 6 件)

| rank | code | sector | risk | warning |
|---|---|---|---|---|
| 1 | 9247 ＴＲＥ HD | 情報通信 | 8,060 | - |
| 2 | 9628 燦 HD | 情報通信 | 6,950 | - |
| 3 | 4404 ミヨシ油脂 | 食品 | 10,665 | ⚠ |
| 4 | 2146 ＵＴグループ | 情報通信 | 920 | - |
| 5 | 4828 ビジネスエンジニアリング | 情報通信 | 6,230 | - |
| 6 | 8699 ＨＳ HD | 金融 | 5,700 | - |
| 7 | 3089 テクノアルファ | 商社 | 5,275 | - |
| 8 | 3712 情報企画 | 情報通信 | 5,195 | - |
| 9 | 7803 ブシロード | 情報通信 | 1,305 | - |
| 10 | 9633 東京テアトル | 不動産 | 7,895 | - |
| 11 | 4914 高砂香料工業 | 素材化学 | 5,900 | - |
| 12 | 3479 ティーケーピー | 不動産 | 8,625 | ⚠ |
| **13** | **9417 (新規)** | 情報通信 | 1,550 | - |
| 14 | 4540 ツムラ | 医薬品 | 17,910 | ⚠ |
| 15 | 8057 内田洋行 | 商社 | 10,265 | ⚠ |
| **16** | **9008 (新規)** | **運輸・物流 (新セクター)** | 3,735 | - |
| **17** | **6196 (新規)** | 情報通信 | 6,165 | - |
| **18** | **7595 (新規)** | 情報通信 | 6,780 | - |
| **19** | **3134 (新規)** | **小売 (新セクター)** | 2,060 | - |
| **20** | **3962 (新規)** | 情報通信 | 4,605 | - |

→ **新顔 6 件 (9417/9008/6196/7595/3134/3962) 浮上** ✓
→ **新セクター 2 件 (運輸・物流 + 小売)** ✓

## §8 9130 / 9247 / 9628 / 4404 状態

| code | post-write 状態 | 評価 |
|---|---|---|
| 9130 | **excluded 維持** ✓ (= recently_seen_demoted) | demote 効果継続 |
| 9130 entry/watch 復帰 | **0** ✓ | |
| 9247 | entry **rank 1** | universe 拡張後も score 上位、固定化継続 |
| 9628 | entry **rank 2** | 同 sector 重複、固定化継続 |
| 4404 | entry rank 3 / risk_yen_warning / risk_over=True | warning 表示のみ entry 維持 ✓ |

→ universe 拡張は実発現したが、**9247/9628 固定化緩和は限定的**.
→ rank 1/2 のまま (= D44/D47/D48 と同位置).
→ 根本緩和には theme overlay / scoring 補正 / 単独 demote 別 wave 必要.

## §9 安全 gate 全 0 確認

| gate | 結果 |
|---|---|
| 低流動性 entry 混入 | 0 ✓ |
| letter-suffix entry 混入 | 0 ✓ |
| 非 100 株 entry 混入 | 0 ✓ |
| risk_yen 順位加点 | 0 ✓ (= v1.4.2 維持) |
| risk_yen entry 除外 gate | 0 ✓ (= warning 表示のみ) |
| 9130 entry/watch 復帰 | 0 ✓ |
| consumer validation_passed | True ✓ |
| violations / warnings | 0 / 0 ✓ |

## §10 CRITICAL/HIGH 判定

- **CRITICAL: 0**
- **HIGH: 0**

## §11 D49 handoff

| 観点 | 内容 |
|---|---|
| F111 baseline | max=50 / recently_seen 8 件版継続 |
| signal_persistence baseline | top-n=100 (= W2 適用継続 + W3 staging write 反映) |
| staging signals latest | 2026-07-22 / 109 件 (= W3 write 後) |
| 9130 demote | excluded 維持 (= W3 でも維持確認) |
| 4404 risk warning | entry rank 3 維持 (= warning 表示のみ) |
| 9247 / 9628 | rank 1/2 継続 (= universe 拡張後も固定化、別 wave で根本対策) |
| **新顔 entry 6 件** | 9417/9008/6196/7595/3134/3962 (= D49 朝 pilot で観察) |
| **新セクター 2 件** | 運輸・物流 + 小売 (= D49 朝 pilot で観察) |
| 朝判断負荷 | top 5 圧縮継続 (= entry 20 件 → 主役 3 + 新顔 sector 多様化 2) |

## §12 次 wave 候補

| ID | 内容 | 優先 |
|---|---|---|
| **D49 朝 pilot** | universe 拡張後の初回稼働 (= base_date 2026-07-22 等) | **最優先** |
| **F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W1** | seed theme list + assign runner (= 9247/9628 固定化根本対策) | 高 |
| F111-UNIVERSE-EXPANSION-R1-W4 | top-n=200 検証 + 必要なら baseline 化 | 中 |
| 9247 / 9628 単独 demote 検討 | universe 拡張後も rank 1/2 継続なら別 wave + HQ approve 後 | 中 |
| F111-JQUANTS-UNIVERSE-BUILDER-R1 | J-Quants universe builder (= API 承認後、業績 description / news theme) | 低 |
| 定期 staging signals 書込 cron | W2 baseline (top-n=100) を毎日 staging に反映する別 wave | 中 |

## §13 safety footer

- 本 wave は **staging write 1 回限定** (= HQ_APPROVE_STAGING_SIGNAL_PERSISTENCE_WRITE)
- staging DB md5 変化 (= write 反映): 6cb3885c → f6fa86df ✓ (想定通り)
- production / develop DB md5 不変 ✓
- F282 plist 3 file 不変 ✓ (= 1772 / 1844 / 1772、mtime 不変)
- API 0 / token 0 / LINE 0 / launchctl 0 / plist 0 / cron 0 / workflow 0 / VACUUM 0
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / sudo / rm -rf 0
- git add 0 / git commit 0 / git push 0 / --no-verify 0
- TODO Excel 更新 0
- forbidden phrase 0
- 9247 / 9628 demote 本実行: 0 (= rank 1/2 継続だが本 wave 範囲外)
- 9130 demote 状態: 維持

### F282 plist 3 file 別個確認注記
- production plist = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
- smoke plist = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
- LaunchAgents loaded plist = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)

## §14 関連 file

- W3 dry-run: `/tmp/fire_universe_w3/top100_dry_run.json`
- W3 staging write: `/tmp/fire_universe_w3/top100_write.json`
- F111 post-write: `/tmp/fire_universe_w3/f111_after_top100_write.json`
- consumer post-write: `/tmp/fire_universe_w3/consumer_after_top100_write.json`
- 親 W1: `~/fire-vault/03_design/F111_UNIVERSE_EXPANSION_R1_W1_result_2026-05-16.md`
- 親 W2: `~/fire-vault/03_design/F111_UNIVERSE_EXPANSION_R1_W2_result_2026-05-16.md`
- 統合 strategy: `~/fire-vault/03_design/F111_exploration_expansion_strategy_2026-05-16.md`
