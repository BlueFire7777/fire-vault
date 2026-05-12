---
id: FIRE-CODEX-R2-10-lane-scaling-design
phase: ガバナンス / R-01-08 / FIRE-CODEX-R2 設計
priority: 高
status: 設計初版 (= Wave 20 W20-1)、コード変更なし、HQ 評価待ち
owner: BlueFire7777 (Fujiwara)
date: 2026-05-12
depends_on:
  - FIRE-CODEX-R1 v1.1 (= 19 wave 連続 60-80% 短縮達成、現行運用)
  - HQ Wave 20 起票承認 (= 2026-05-12)
chapter: ガバナンス / R-01-08
---

# FIRE-CODEX-R2 / 10-Lane Scaling Design

最終更新: 2026-05-12

## 0. 要約

現行 **FIRE-CODEX-R1 v1.1** (= 本線 + Codex 5 lane) を **最大 10 lane** に
段階拡張する設計。**設計のみ**、本文書では **コード・DB・LINE・cron は
1 つも触らない**。R1 で確立した **6 段ガード / 三段ガード / staging-only
guard / HQ 明示承認 / iPhone 1 ブロック報告** 等の制約は **全て R2 に継承**。

R2 の本質は「lane を増やすこと」ではなく、

- **並列度を上げても本線 Integrator が破綻しない仕組み**
- **lane 数増加で安全要件が一度たりとも緩まないこと**
- **HQ の判断ループは現状以上に短く・明示的にすること**

この 3 点を満たす設計フレームワーク。

---

## 1. Background (= R1 実績と限界)

### R1 v1.1 実績 (= 2026-04-25 〜 2026-05-12)

| 指標 | 値 |
|---|---|
| 完了 wave 数 | **19 wave 連続成功** |
| 並列短縮率 | 60-80% (= 全 wave 平均) |
| 累計 PASS | **4,024** (= test 総数) |
| safety incident | 0 (= production write 事故なし) |
| CRITICAL audit verdict | 全 解消 |

### R1 で見えた飽和点

1. **大型 wave で sub-task ≥ 5 になると Codex 5 lane では足りない**
   (W8 / W14 / W17 で本線が「次の sub-task を待つ Codex」を抱える状況)
2. **L3 Implementation が並列化のボトルネック** (= 1 lane で複数 file
   修正するとファイル衝突)
3. **L4 Audit が直列実行** (= L3 完了後にしか走らない)
4. **vault doc 作成 (L5) が本線 Integrator の手で行われる** (= 並列化余地)
5. **regression 全 PASS 確認が wave 終盤に集中** (= 早期検出余地)

→ **10 lane 体制でこれらを並列化することで wave 実時間を更に短縮**。

---

## 2. 10 レーン役割定義

### 案 A: 職能別レーン (= R1 5 lane を職能で拡張)

| ID | 役割 | 担当者 | 並列数 | 用途 |
|---|---|---|---|---|
| L1a | Design 設計 (主) | Codex | 1 | 主要 sub-task の設計 doc |
| L1b | Design 設計 (副) | Codex | 1 | 補助 sub-task / 代替案 |
| L2a | Test テスト (主) | Codex | 1 | 主要 module の test |
| L2b | Test テスト (副) | Codex | 1 | 補助 module / regression |
| L3a | Implementation (主) | Codex | 1 | 主要 file 修正 |
| L3b | Implementation (副 1) | Codex | 1 | 別 file 並列実装 |
| L3c | Implementation (副 2) | Codex | 1 | 別 file 並列実装 |
| L4 | Audit | Codex | 1 | adversarial レビュー |
| L5 | Docs | Codex | 1 | vault doc / log.md draft |
| L6 | Regression | Codex | 1 | 全 PASS 監視 / 既存契約検証 |

合計 10 lane。本線 (= PM / Architect / Integrator / Final Reviewer) は Codex
外、これは R1 同様。

### 案 B: 機能別レーン (= domain split)

| ID | domain | 対象 |
|---|---|---|
| D1 | simulation 系 | simulation/* + tests/simulation/* |
| D2 | notifications 系 | notifications/* + tests/notifications/* |
| D3 | pnl + report 系 | pnl/* + fire/report/* |
| D4 | market_data + materials 系 | market_data/* + materials/* |
| D5 | patterns + agents 系 | patterns/* + agents/* |
| D6 | evaluation 系 | evaluation/* |
| D7 | risk 系 | risk/* |
| D8 | scripts/jobs 系 | scripts/jobs/* |
| D9 | scripts/setup + DB 系 | scripts/setup/* (= migration / schema) |
| D10 | tests cross-cutting | tests/* (= 横断) |

### 比較表

| 観点 | 案 A (職能別) | 案 B (機能別) |
|---|---|---|
| R1 互換性 | ★★★ (= 既存 L1-L5 拡張) | ★ (= 全面再構成) |
| file 衝突回避 | 中 (= L3a/b/c 内部ルール必要) | 高 (= domain で natural split) |
| 並列効率 | 高 (= 工程パイプライン化) | 中 (= domain サイズ不均一) |
| 教育コスト | 低 | 中 |
| HQ 判定の単純さ | 高 | 低 (= domain 横断 wave で混乱) |
| 大型 wave 適応 | 高 | 中 (= 1 domain 集中時に飽和) |
| 監査独立性 | 高 (= L4 専用) | 中 (= D10 兼用ぎみ) |

### 推奨: **Hybrid (= 案 A 主軸 + 案 B 補完)**

- 既定: **案 A 職能別** で運用 (= L1a/b, L2a/b, L3a/b/c, L4, L5, L6)
- 1 wave 内で **L3a/b/c に domain 別 file を割り当てる** (= 案 B の natural
  split を file ownership 層で利用)
- 例: L3a = simulation, L3b = notifications, L3c = pnl

これにより:
- R1 の既存 1〜5 lane 運用と滑らかに接続
- file 衝突は domain 別 L3 で自動回避
- HQ 報告は職能別で記述 (= 既存テンプレ継続)

---

## 3. file ownership ルール

### 基本原則

1. **同 file 同時編集禁止**: 1 つの file を 2 lane が同 wave 内で編集禁止
2. **L4 audit は read-only**: 既存 / 新規 file の検査のみ、修正は L3 系
3. **L5 docs は vault 配下のみ**: `~/fire-vault/02_todo/*`, `03_design/*`,
   `07_incidents/*`, `log.md` のみ (= fire コード触らず)
4. **L6 regression は test 実行と report のみ**: file 修正なし

### 競合解消ルート

```
file 衝突疑い検知
   ↓
本線 Integrator が file owner を決定 (= 1 wave 1 file 1 lane)
   ↓
他 lane は「待機 (= sub-task 後出し)」 or
       「別 file に置換」 or
       「合成済 patch 受領」 (= 本線が L3 完了後 merge)
```

### allowed_files / forbidden_files (= R1 継承)

- 各 sub-task prompt に **allowed_files / forbidden_files 明示** (R1 必須)
- 10 lane では更に **owner_lane_id** を明示 (= 「L3a だけが書ける」)
- forbidden_files の **共通 7 件** は固定:
  - scripts/seed_pattern_layer1.py
  - simulation/research_lane/historical_indicators.py
  - notifications/* (= LINE 送信経路、別承認時のみ)
  - scripts/setup/migrate_*.py (= production schema、別承認時のみ)
  - .github/workflows/*
  - data/fire.db / data/fire.develop.db (= production / develop)
  - TODO Excel

### file lock 表 (= 1 wave 内)

本線 Integrator が各 wave 開始時に作成:

```
| file | owner_lane | sub_task | status |
|---|---|---|---|
| scripts/jobs/fetch_X.py | L3a | W20-3 | active |
| tests/scripts/jobs/test_X.py | L2a | W20-3 | active |
| notifications/templates_Y.py | L3b | W20-4 | active |
```

衝突発見時は **wave 中断 → 本線が再分配 → HQ 報告** (= overhead 5-10 分)。

---

## 4. 同時実装できる / できないタスク分類

### 並列可 (= 10 lane 全開での並走 OK)

- **別 module の test + implementation** (= L2a + L3a)
- **別 file の implementation 並列** (= L3a / L3b / L3c が異 file)
- **vault doc 作成と code 修正の並走** (= L5 + L3*)
- **設計 doc と既存検査の並走** (= L1a + L4)
- **regression PASS 確認と新規 test 追加の並走** (= L6 + L2*)
- **複数 module の audit 並走** (= L4 主 + L4 派生 task = L4 兼任は不可、
  代替: L6 が補助 audit、ただし L6 audit は L4 が必ず 二次レビュー)
- **HQ 承認待ち sub-task の事前 staging** (= 本線で次 wave plan 起草)
- **異 wave の post-mortem** (= 過去 wave audit doc 更新)

### 並列不可 (= 単線必須、本線のみ実行)

| 項目 | 理由 |
|---|---|
| DB write (production) | 唯一の本番、原本 |
| DB write (develop) | F286-DATA-R3 D2.2 で唯一の develop |
| DB write (staging) | HQ 明示承認後、本線が 1 度だけ |
| schema migration apply | W14 で確立済、本線が 3 環境順次 |
| LINE 送信 (実) | 本線で send_guard 経由のみ |
| token / secret 参照 | Codex に渡さない、本線 env のみ |
| cron / launchd / crontab | 本番登録は HQ 明示承認後、本線 |
| .github/workflows/ 変更 | FIRE_ALLOW_WORKFLOW_CHANGE 必須、本線 |
| TODO Excel 更新 | Fujiwara 手動 (= 本線も触らない、表示のみ) |
| 楽天証券 操作 | 第 3 層 (= 人間、Computer Use 不採用) |
| Codex 直接 commit | R1 で禁止、本線が git add / git commit |
| 本番 DB スナップショット | 本線が ~/fire-backups/ に作成 |
| HQ 報告 1 ブロック | 本線が iPhone 形式で出力 |

---

## 5. 本線 Integrator 負荷上限

### 想定容量 (= 1 wave あたり)

| 指標 | R1 5 lane 実績 | R2 10 lane 目標 | R2 上限 |
|---|---|---|---|
| sub-task 数 / wave | 5-7 | 8-10 | **12** |
| Codex prompt 作成数 | 5-7 | 8-10 | **10** |
| 本線 file 修正 file 数 | 0-2 | 0-3 | **5** |
| audit doc レビュー数 | 1 | 1-2 | **3** |
| commit 数 / wave | 1-3 | 2-4 | **6** |
| HQ 報告 1 ブロック | 1 | 1 | **1 (= 必須)** |
| wave 実時間 (= 開始 → 報告) | 60-90 分 | 80-120 分 | **150 分** |

上限超過時は **wave 分割** (= W20a / W20b 等) を本線が判断、HQ 報告で
事前合意。

### 過負荷検出シグナル

- 1 lane の出力が **prompt 投入から 90 分以上未着** → 本線が打診
- 1 sub-task の patch サイズが **500 行以上** → file ownership 再分配
- 同 wave 内で **audit doc が 3 件以上** → wave 分割
- HQ 報告作成に **30 分以上** かかる → 報告内容過剰 (= split)

### 退避ルート

過負荷検出時は:

```
本線 → HQ に「Wave 分割提案」を即時通知 (= 報告 1 ブロック内に明示)
   ↓
HQ approve → Wave N-a / N-b に分割
   ↓
未完 sub-task は次 wave plan に移動
```

---

## 6. 1 Wave 最大 Codex 投入数

### 段階別上限

| step | 名称 | Codex 並列上限 | HQ approve 要件 |
|---|---|---|---|
| Step 0 | 現行 R1 5 lane | 5 lane | (= 既定) |
| Step 1 | R2 初期 | **6 lane** | HQ approve 1 回 |
| Step 2 | R2 拡張 | **8 lane** | Step 1 で 2 wave 連続 PASS |
| Step 3 | R2 完全 | **10 lane** | Step 2 で 2 wave 連続 PASS |
| Step 4 | (将来) | 12 lane 以上 | 別 HQ 承認、本設計外 |

### 同期点 (= 並列上限と独立に必須)

- HQ approve 待ち (= sub-task 起票時、wave 開始時)
- 本線 commit (= file ownership 解放点)
- L4 audit verdict 確定 (= CRITICAL / HIGH 0 確認)
- HQ 1 ブロック報告送出 (= wave 完了)

これらは **必ず単線**、Codex 並列数とは無関係。

---

## 7. commit 分割ルール

### 既存 R1 ルール (= R2 継承)

- main へ直 push しない (= develop ブランチ作業)
- Codex 直接 commit しない (= 本線のみ)
- pre-commit hook で Codex review 必須 (= --no-verify 禁止)
- workflow 変更時のみ FIRE_ALLOW_WORKFLOW_CHANGE=1
- Co-Authored-By: Claude Opus 4.7 を必ず付与

### R2 追加ルール (= 10 lane 対応)

1. **lane ごとに 1 commit は NG** (= 過分割、レビュー困難)
2. **関連 sub-task を 1 commit にまとめる** (= 例: L3a + L2a の同 module)
3. **vault doc / log.md は別 follow-up commit** (= R1 既存 pattern)
4. **audit doc は別 incident commit** (= 07_incidents/* に commit)
5. **多 sub-task wave (= sub ≥ 6) は最大 3 commit に分割可**:
   - commit 1: 主 impl + test (= L3a+L2a+L3b+L2b)
   - commit 2: 補助 impl + test (= L3c+L2c など)
   - commit 3: vault docs (= log.md + 02_todo + 03_design)
6. **commit message に lane ID を明記**:
   - `feat(F286-XXX): Wave N L3a+L3b X+Y impl, L2a+L2b tests, ...`

### commit 順序 (= 推奨)

```
1. fire develop に L3 系 commit (= 主 impl)
2. fire develop に L3 系 commit (= 補助 impl、必要なら)
3. fire-vault main に vault doc commit (= plan + results + design + audit)
4. fire-vault main に log.md follow-up commit (= milestone entry)
```

---

## 8. 完了報告テンプレ (= HQ 1 ブロック、R1 拡張)

### 構造 (= iPhone 単一コードフェンス形式維持)

```
=== FIRE-CODEX-R{N} v{V} Wave {W} 完了報告 ===

Wave {W}: {title}
最終更新: YYYY-MM-DD

★ 結果サマリ
- sub-task 件数 / Codex 投入 lane 数
- 安全 critical 0 / DB write 件数 / LINE 件数
- 並列効果 % / 通算達成 wave 数
- 回帰 PASS 数

Wave {W} 投入結果 (= sub-task table)
| sub | lane | task | 結果 |

W{W}-{n} 詳細 (= 主要 sub-task の発見)

fire develop commits (= 件数 + hash 列挙)

fire-vault main commits (= 件数 + hash 列挙)

changed files (= file count + 主要 path)

安全 (= Wave {W} 全 ✓ table)

並列効果 (= 実時間 / 単独推定 / 短縮率 / 通算)

回帰 (= PASS 数)

HQ 判断が必要な論点 (= n 件)

★ 引き続き禁止: ...

=== Wave {W} 完了 ===
```

### R2 追加要素

- **Codex 投入 lane 数 (= 何 lane / 10)** を結果サマリに明示
- **lane 別効率レポート** (= 各 lane の出力時間 / 修正行数)
- **file ownership 衝突件数** (= 0 が目標)
- **過負荷検出有無** (= Step 進行判断材料)

---

## 9. 10 lane 初回試験投入候補

### 候補 a (= 推奨、Step 1 = 6 lane 投入)

**Wave 21 候補**: F101 API behavior investigation **実装** (= W19-1 推奨対応)

| sub | lane | task |
|---|---|---|
| W21-1 | L1a | V2 spec 公式 doc 検証 設計 |
| W21-2 | L3a | endpoint 候補 try 用 client patch (= staging probe 専用、auth は本線) |
| W21-3 | L2a | client patch test (= mock response) |
| W21-4 | L4 | adversarial audit |
| W21-5 | L5 | vault doc (= Wave 21 plan + results) |
| W21-6 | L6 | regression PASS 確認 |

理由: 6 lane で Step 1 を慎重に踏み、十分な収束性を確認。

### 候補 b (= 補助、Step 1 補完)

**Wave 22 候補**: cron thaw design only (= sub-D3、設計のみ)

| sub | lane | task |
|---|---|---|
| W22-1 | L1a | cron 復活前の安全要件 設計 |
| W22-2 | L1b | launchd plist 設計 比較 |
| W22-3 | L4 | 既存 cron 残骸 audit |
| W22-4 | L5 | vault doc |
| W22-5 | L6 | 既存 plist 整合 確認 |

理由: 設計だけなので 5-6 lane で安全。

### 候補 c (= 後続、Step 2 = 8 lane 投入)

**Wave 23 候補**: DATA-R3 sub-D2.3.x final audit + LOW 統一 (= F101 forbidden path)

8 lane 投入: L1a + L2a + L2b + L3a + L3b + L4 + L5 + L6。

### 候補 d (= Step 3 = 10 lane 投入、最低 2 wave PASS 後)

**Wave 24+ 候補**: F286-REPORT-R1 LINE 実送信 token integration (=
複数 module 横断、6 段ガード継承、HQ 明示承認後)。

10 lane: L1a + L1b + L2a + L2b + L3a + L3b + L3c + L4 + L5 + L6。

---

## 10. rollback / abort 条件

### Hard abort (= 即時 wave 中断、本線 → HQ 報告)

| 条件 | 検出層 | 対応 |
|---|---|---|
| audit CRITICAL 1 件以上 | L4 | wave 即時 abort、修正後再 wave |
| safety violation (= production DB write 等) | 本線 / hook | wave 即時 abort、incident doc |
| LINE 実送信 (= 未承認) | 本線 / hook | wave 即時 abort、incident doc |
| token / secret leak | 本線 / hook | wave 即時 abort、token rotate |
| file ownership 衝突 ≥ 2 | 本線 | wave 中断、再分配後再開 |
| HQ 明示 abort 指示 | HQ | 即時停止、状態保存 |

### Soft rollback (= 部分復旧、wave 継続可)

| 条件 | 対応 |
|---|---|
| audit HIGH 1 件 | 同 wave 内で修正 (= -fix sub-task)、再 audit |
| 回帰 PASS 数 -1 以上 | 該当 test 修復、PASS 戻し後継続 |
| 1 lane の出力 90 分以上未着 | 当該 sub-task の prompt 修正 or 別 lane へ振替 |
| 本線 Integrator 過負荷検出 | wave 分割 (= W{N}a / W{N}b) |

### Step 後退 (= Codex 並列数の段階退避)

| 条件 | 対応 |
|---|---|
| Step 2 で 1 wave 失敗 (= audit FAIL or rollback) | 1 wave Step 1 (= 6 lane) に後退、再 PASS で復帰 |
| Step 3 で 1 wave 失敗 | 2 wave Step 2 に後退、2 連続 PASS で復帰 |
| Step 1 で 1 wave 失敗 | R1 5 lane に後退、根本原因分析 |

---

## 11. 5 lane → 10 lane 段階移行案

### Phase 移行表

| Phase | 期間 | lane 数 | 主目的 | exit 条件 |
|---|---|---|---|---|
| **P0** | 現状 | 5 | R1 v1.1 安定運用 | (= 現状継続) |
| **P1** | 2026-05-12 〜 | 6 | R2 初導入、L1b or L3b 追加 | 2 wave 連続 PASS |
| **P2** | P1 完了後 | 8 | L2b / L6 追加 | 2 wave 連続 PASS |
| **P3** | P2 完了後 | 10 | L1b/L2b/L3c/L6 全活性 | 安定運用継続 |

### Phase 別 lane 構成案

#### P1 (= 6 lane)
```
本線 (PM/Architect/Integrator/Final Reviewer)
   + L1a Design (Codex)
   + L2a Test (Codex)
   + L3a Impl (Codex)
   + L3b Impl 副 (Codex) ★ 新規
   + L4 Audit (Codex)
   + L5 Docs (Codex)
```

#### P2 (= 8 lane)
```
本線 (PM/Architect/Integrator/Final Reviewer)
   + L1a Design (Codex)
   + L2a Test (Codex)
   + L2b Test 副 (Codex) ★ 新規
   + L3a Impl (Codex)
   + L3b Impl 副 (Codex)
   + L4 Audit (Codex)
   + L5 Docs (Codex)
   + L6 Regression (Codex) ★ 新規
```

#### P3 (= 10 lane、最終)
```
本線 (PM/Architect/Integrator/Final Reviewer)
   + L1a Design 主 (Codex)
   + L1b Design 副 (Codex) ★ 新規
   + L2a Test 主 (Codex)
   + L2b Test 副 (Codex)
   + L3a Impl 主 (Codex)
   + L3b Impl 副 1 (Codex)
   + L3c Impl 副 2 (Codex) ★ 新規
   + L4 Audit (Codex)
   + L5 Docs (Codex)
   + L6 Regression (Codex)
```

### Phase 移行 HQ 承認ループ

```
P0 → P1: HQ 明示承認 (= Wave 20 完了後、別 HQ approve)
P1 → P2: HQ 明示承認 (= P1 で 2 wave 連続 PASS 後)
P2 → P3: HQ 明示承認 (= P2 で 2 wave 連続 PASS 後)
```

各 Phase 移行 wave で:
- **必ず簡易 wave (= sub ≤ 5)** で試験
- audit verdict CRITICAL 0 / HIGH 0 必須
- 安全 violation 0 必須
- 本線過負荷検出 0 必須

### Phase 関係なく適用される単線項目 (= R1 継承)

(再掲、Phase が進んでも単線維持)

- DB write 全
- LINE 送信
- token / secret 参照
- cron / launchd / crontab 登録
- production schema apply
- workflow 変更
- HQ 報告

---

## 12. 安全 (= 本設計 doc 自体の安全要件)

| 項目 | 結果 |
|---|---|
| 実 DB write | 0 (= 本 doc は設計のみ) |
| 実 LINE 送信 | 0 |
| 実 API call | 0 |
| code 変更 | 0 (= fire/* / scripts/* 未接触) |
| token / secret 参照 | 0 |
| cron / launchd / crontab 登録 | 0 |
| .github/workflows/ 変更 | 0 |
| --no-verify | 不使用 |
| TODO Excel | 未更新 |
| 楽天 / 自動発注 / Computer Use | なし |
| 既存 R1 v1.1 安全契約 | 全 継承 |

---

## 13. R1 v1.1 → R2 差分要約

| 観点 | R1 v1.1 | R2 |
|---|---|---|
| Codex lane 上限 | 5 | 10 (= 段階導入) |
| L3 並列 | 1 lane | 最大 3 lane (= L3a/b/c) |
| L1 並列 | 1 lane | 2 lane (= L1a/b) |
| L2 並列 | 1 lane | 2 lane (= L2a/b) |
| Regression 専用 lane | なし (= 本線兼任) | L6 |
| 段階導入 | 単一 | Phase P1 / P2 / P3 |
| 本線負荷上限 | 暗黙 | **明示 (= sub 12 / commit 6 / 時間 150 分)** |
| file ownership 表 | 暗黙 | **wave 開始時に明示作成** |
| 過負荷検出 | 手動 | **明示シグナル + Step 後退ルート** |
| HQ 報告 | 1 ブロック | 1 ブロック + lane 別効率 |

---

## 14. 関連リンク

- [[FIRE_CODEX_R1_multi_lane_parallel_orchestration_2026-05-11|R1 v1.1 既存運用]]
- [[../02_todo/FIRE_CODEX_R1_WAVE20_plan|Wave 20 plan]]
- [[../log]]
- 要件書: `~/fire-vault/01_requirements/FIRE_要件書_第33章_*.md` (= R-01-08)

---

## 15. HQ 判断論点 (= 本 doc 完成後)

1. **本設計の採用可否** (= 採用 = Phase P1 着手、保留 = R1 継続)
2. **Phase P1 着手 wave** (= 推奨: Wave 21 = F101 API 修正実装)
3. **lane 名前空間** (= L1a/L1b 案 vs 機能別 D1-D10 案、推奨: 案 A 主軸)
4. **Codex prompt template の R2 改訂** (= owner_lane_id 必須化)
5. **HQ 報告テンプレ更新** (= lane 別効率レポート追加)
6. **既存 19 wave の retroactive R2 記録** (= 不要、本 doc 以降から適用)
