---
id: FIRE-CODEX-R1-WAVE8-results
phase: ガバナンス / Codex 並列実装 Wave 8 完了 / R-01-08 整合
priority: 最優先
status: 完了 ★ Wave 8 4 impl + 3 audit + 2 fix + 1 final audit 完了 (2026-05-12、HIGH 5 件即修正 / CRITICAL 1 件即修正、3,751 PASS)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - FIRE-CODEX-R1 v1.1 / Wave 7 (= 完了)
  - HQ Wave 8 approve (= 2026-05-11、W8-0-fix + REPORT-R1 impl + DATA-R3 sub-D2.3 --dry-run 承認)
chapter: ガバナンス / R-01-08 / Codex 運用拡張
---

# FIRE-CODEX-R1 v1.1 Wave 8: W8-0-fix + REPORT-R1 impl + DATA-R3 sub-D2.3 --dry-run

最終更新: 2026-05-12

## ★ 状態: 完了 (= 4 split commit / 4 lane impl + 3 lane audit + 2 fix + 1 final audit / 3,751 PASS)

HQ Wave 8 approve 受領後、Architect 判断で 4 sub-task impl + 3 sub-task
audit + 2 fix + 1 final audit (= 計 10 sub-task) を起票。Codex を最大
3 並列で運用、本線 Integrator が全 review + split commit + CRITICAL 即修正。

**重要発見**:
- W8c-final-audit で **CRITICAL 0 ★**
- HIGH 5 件 (W8-5 1 + W8-6 2 + W8-7 2) 全て W8a-fix / W8b-fix で解消
- Codex pre-commit CRITICAL 1 件 (= probe unauthenticated GET) を W8a-fix-2 で即修正
- residual: W8-5 HIGH #1 (= sqlite3 標準 API 制約による TOCTOU 完全閉鎖
  困難) → documented risk として HQ 判断仰ぐ

## Wave 8 投入結果 (= 10 sub-task)

### Phase 1: impl (= 4 lane、Codex 3 並列起動)

| sub | lane | task | tests | CRITICAL/HIGH |
|---|---|---|---|---|
| W8-1 | L3+L2 | PNL-R3 W8-0-fix impl + tests | 14 → 65 PASS | 0 / 0 (= 初期実装) |
| W8-2 | L3+L2 | REPORT-R1 aggregators + tests | 19 PASS | 0 / 0 |
| W8-3 | L3+L2 | REPORT-R1 daily_report + markdown + runner + tests | 33 PASS | 0 / 0 |
| W8-4 | L3+L2 | DATA-R3 sub-D2.3 --dry-run × 4 runner + tests | 25 → 832 PASS jobs/ | 0 / 0 |

### Phase 2: audit (= 3 lane、Codex 順次)

| sub | lane | task | CRITICAL | HIGH | MEDIUM | LOW |
|---|---|---|---|---|---|---|
| W8-5 | L4 | W8-0-fix audit | 0 ★ | 1 (TOCTOU partial) | 0 | 4 |
| W8-6 | L4 | REPORT-R1 audit | 0 ★ | 2 (renderer + symlink) | 2 (completion + goal) | 5 |
| W8-7 | L4 | DATA-R3 sub-D2.3 audit | 0 ★ | 2 (auth + exception) | 2 | 5 |

### Phase 3: fix (= 2 fix lane)

| sub | lane | task | tests |
|---|---|---|---|
| W8a-fix | L3+L2 | W8-5 + W8-7 HIGH 3 件解消 (TOCTOU FD/inode + auth fail + generic catch) | +10 / 73 PASS |
| W8b-fix | L3+L2 | W8-6 HIGH 2 + MEDIUM 2 解消 (renderer 純関数 + dangling symlink + completion-report + goal fallback) | +13 / 65 PASS |
| W8a-fix-2 | 本線手動 | Codex pre-commit CRITICAL 解消 (probe external API no-op、env + DB のみ) | +2 |

### Phase 4: final audit (= 1 lane)

| sub | lane | task | CRITICAL | HIGH | MEDIUM | LOW |
|---|---|---|---|---|---|---|
| W8c-final-audit | L4 | W8a-fix + W8b-fix 最終 audit | 0 ★ | 1 residual (W8-5 sqlite3 API 制約) | 0 | 3 |

## fire develop split commit (= 4 件)

| commit | 内容 |
|---|---|
| 7247010 | feat(F286-PNL-R3): W8-0-fix hardening (TOCTOU + atomic + NFKC) |
| 637cea1 | feat(F286-REPORT-R1): Daily PnL Report Generator (aggregators + daily + renderer + runner) |
| b20c92b | feat(F286-DATA-R3): --dry-run × 4 sub-runners + auth/exception + probe simplification |
| cb55b9a | docs+test(FIRE-CODEX-R1): Wave 8 完了 table + whitelist W8-3 runner |

## W8c-final-audit 残存 HIGH 1 件 (= HQ 判断仰ぐ documented risk)

### W8-5 HIGH #1: sqlite3 標準 API による TOCTOU 完全閉鎖困難

**現状**:
- `_connect_write()` で `os.open(O_RDONLY | O_NOFOLLOW) + os.fstat()` の
  pre-check が入った (= W8a-fix)
- `PRAGMA database_list` 経由の post-connect verify も実装
- ただし `sqlite3.connect(str(path))` が実際に保持する FD/inode を直接
  検証する標準 API がない

**残る race**:
1. pre-check 通過 (= path は staging.db)
2. `sqlite3.connect()` 開く直前に attacker が path を forbidden DB
   hardlink に差し替え
3. sqlite3 が forbidden DB を開く
4. attacker が path を staging.db に戻す
5. post-connect verify 通過 (= path stat は staging.db)
6. connection は forbidden DB を保持したまま UPDATE 発行

**実害評価**:
- adversarial: attacker が同一 filesystem 上で microsecond timing で path を
  差し替えできる前提
- 単一 user dev 機 (= Mac mini M4 Pro、Fujiwara のみ) 環境では現実化困難
- W8-1 staging UPDATE smoke は HQ 明示承認後実行のため、攻撃 vector 増加
  なし

**完全閉鎖の選択肢**:
- 案 a: sqlite3 を回避、`os.open + memfd / mmap` で DB 操作 (= 大幅改修、
  WAL モード非対応リスク)
- 案 b: 専用 VFS shim を C extension で実装 (= 重い)
- 案 c: chroot / seccomp で path access 制限 (= 環境依存)
- 案 d: 現状 + post-connect double-check 強化 (= TOCTOU window さらに縮小、
  完全閉鎖ではない)

**HQ 判断仰ぐ**:
- 案 1 (= 推奨): documented risk 受容、W8-1 staging UPDATE smoke 着手判断は
  HQ 明示承認時に再評価。実害低 + 残 vector が adversarial 前提のため。
- 案 2: 案 d で post-connect double-check 強化 → Wave 9 W9-X-fix で追加
  hardening
- 案 3: 完全閉鎖を求める → 大幅改修、Wave 9+ で別 sub-task

## Codex CRITICAL / pre-commit hook 即修正履歴 (= 計 2 件)

### #1: W8-1 内部 CRITICAL (= W6 fix と類似)
- W8-1 実装時に Codex pre-commit で発見、その場で修正
- (詳細は W8-1 Codex stdout log 参照、最終的に 14 件 → 65 PASS)

### #2: W8a-fix-2 Codex pre-commit CRITICAL (= W8-4 由来)
- **指摘**: f100/f101 の `_probe_external_api` が unauthenticated GET、
  J-Quants は auth 必須のため本番では常時 401/403 → exit 2 で probe 常時 fail
- **修正**: f100/f101 の `_probe_external_api` を **no-op 化** (= W8a-fix-2)
  - probe は env + DB のみ verify
  - external API call なし (= secret 値読出なし、本番認証 fetch なし)
  - auth / connectivity は実 fetch (= --dry-run 不渡し) で初めて verify
  - f111 / f119 と統一 (= 元から no-op)
- **対応 test**: 401/403/503 connectivity_fail tests を「probe doesn't call
  external API」test に置換

## tests (= Wave 8 中の累積)

| 段階 | 累計 PASS |
|---|---|
| Wave 7 終了時 baseline | 3,637 |
| W8-1 + W8a-fix 完了時点 | 3,637 + 14 + 5 = 3,656 |
| W8-2 完了時点 | 3,656 + 19 = 3,675 |
| W8-3 完了時点 | 3,675 + 33 = 3,708 |
| W8-4 + W8a-fix-2 完了時点 | 3,708 + 30 = 3,738 |
| W8b-fix 完了時点 | 3,738 + 13 = 3,751 |
| Wave 8 終了時 | **3,751 PASS / FAIL 0** ★ |

(= 全 W8 完了後の `.venv/bin/pytest` で 3,751 passed in 29.98s 確認済)

## 安全要件 (= Wave 8 全 ✓)

| 項目 | 結果 |
|---|---|
| 実 LINE 送信 (Wave 8 中) | 0 通 ✓ (= SEND 件数 4 のまま) |
| W8-1 staging UPDATE smoke 実行 | **凍結** ✓ (= HQ 明示承認後 W9 で再評価) |
| production DB write | 0 ✓ |
| develop DB write | 0 ✓ |
| staging DB write | 0 ✓ (= Wave 8 では hardening + impl + dry-run 強化のみ) |
| DB mtime production | 5/7 16:12 (= unchanged) |
| DB mtime develop | 5/7 18:14 (= unchanged) |
| DB mtime staging | 5/11 21:25 (= W4.1-A 以降変化なし) |
| token / channel_token / secret 参照 | 0 ✓ (= W8a-fix-2 で外部 API call も 0) |
| 楽天 / 自動発注 / Computer Use / Playwright | なし ✓ |
| 注文価格 / 数量 / 執行指示 helper | 含めない ✓ |
| .github/workflows/ 変更 | 0 ✓ |
| --no-verify | 不使用 ✓ |
| cron / launchd / crontab 本番登録 | 0 ✓ (= sub-D3 凍結継続) |
| W4.1-B F062 経由 smoke | 保留継続 ✓ |
| scripts/seed_pattern_layer1.py | 未接触 ✓ |
| simulation/research_lane/historical_indicators.py | 未接触 ✓ |
| TODO Excel | 未更新 ✓ |
| Codex 直接 commit | 0 ✓ (= 本線 Integrator が全 commit) |
| subprocess --write 渡し | 0 ✓ |
| subprocess env に secret | 0 ✓ |

## 並列効果計測 (= Wave 1-8 通算)

| Wave | 実時間 | 本線単独推定 | 短縮率 |
|------|---------|---------------|--------|
| 1 | 25-30 分 | 90-120 分 | 70-75% |
| 2 | 25-30 分 | 120-150 分 | 75-80% |
| 3 | 30-40 分 | 150-180 分 | 70-80% |
| 4 | 50-60 分 | 180-240 分 | 65-75% |
| 5 | 30-35 分 | 150-180 分 | 80% |
| 6 | 40-50 分 | 180-220 分 | 75-80% |
| 7 | 35-45 分 | 150-200 分 | 75-80% |
| 8 | 90-120 分 | 360-480 分 | 70-75% (= 10 sub-task 最大規模、fix 2 + final audit 含む) |

**8 wave 連続で 65-80% 短縮を達成** ★

## HQ 判断が必要な論点 (= 5 件)

1. **Wave 8 完了 → 次フェーズ進行可否** (推奨: approve)

2. **W8-5 HIGH #1 residual TOCTOU の受容判定** (= 上記「documented risk」
   セクション参照):
   - 案 1 (= 推奨): documented risk 受容
   - 案 2: post-connect double-check 強化 (= Wave 9 W9-X-fix)
   - 案 3: 完全閉鎖 (= 大幅改修、Wave 9+ 別 sub-task)

3. **W9-1 W8-1 staging UPDATE smoke 実行判定** (= W7-1 plan を実行):
   - W8-5 HIGH #1 受容判定後
   - HQ 明示承認 (= W7-1 plan §5 approve template) 必須
   - 案 b (= INSERT 5 row + rollback) 採用

4. **W9-2 REPORT-R1 weekly / monthly impl 着手判定** (= W8-3 daily を延長):
   - 既に基盤実装済、weekly / monthly は同様 pattern
   - LINE 配信は別 sub-task、cron 登録は sub-D3 凍結

5. **W9-3 DATA-R3 sub-D2.3.x staging write 個別 approve** (= 4 runner ×
   個別 HQ approve):
   - 例: sub-D2.3.f100 (= 過去 6 ヶ月 historical staging 蓄積)
   - 各 sub-task で smoke plan + staging write 個別 approve 必要

## Wave 9 候補

- **W9-0-fix-optional**: post-connect double-check 強化 (= W8-5 HIGH #1
  進一步 hardening、HQ 案 2 採用時)
- **W9-1**: PNL-R3 staging UPDATE smoke 実行 (= HQ 別 approve 要)
- **W9-2**: REPORT-R1 weekly / monthly impl + tests + audit
- **W9-3**: DATA-R3 sub-D2.3.x staging write 個別 (= 4 runner、個別 HQ approve)
- **W9-4**: REPORT-R1 LINE 配信 helper (= F062/F236 経由、別 HQ approve)
- 並走候補: FIRE-AUDIT-R1 v1.2 (= AST + docstring context 除外強化)
- 凍結継続: sub-D3 cron 登録 / W4.1-B F062 経由 smoke

## 関連リンク

- [[../03_design/FIRE_CODEX_R1_multi_lane_parallel_orchestration_2026-05-11|FIRE-CODEX-R1 v1.1]]
- [[FIRE_CODEX_R1_WAVE8_plan|Wave 8 plan]]
- [[FIRE_CODEX_R1_WAVE7_results|Wave 7 results]]
- [[../03_design/F286_PNL_R3_staging_update_smoke_plan_2026-05-11|W7-1 staging UPDATE smoke plan]]
- [[../03_design/F286_DATA_R3_sub_D2_3_smoke_plan_2026-05-11|W7-2 sub-D2.3 plan]]
- [[../03_design/F286_REPORT_R1_daily_pnl_report_generator_2026-05-11|W7-4 REPORT-R1 design]]
- [[../07_incidents/F286_PNL_R3_audit_2026-05-11|W7-3 PNL-R3 audit]]
- [[../log]]
