---
id: F101-endpoint-resolution
phase: F101 / API behavior investigation
priority: 高
status: 設計 v1.0 (= Wave 21 W21-2 / W21-3、impl は W21-4)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-12
depends_on:
  - W19-1 F101 API 403 調査
  - HQ Wave 21 起票承認 (= 2026-05-12)
chapter: F101
---

# F101 endpoint resolution 設計

W19-1 で `/fins/announcement` が 403 "endpoint does not exist" を返す
ことを確認した。本 doc では **endpoint resolution の切替可能化** を
設計する。**実 API call は本 wave で発生させない**。

## 1. 現状 (= W18-2 時点)

```python
# materials/client.py L138
return self._request("GET", "/fins/announcement", params=params)
```

`/fins/announcement` (= 単数形) を hardcode。

base URL: `https://api.jquants.com/v2` (= market_data/client.py で定義、
materials/client.py が import)。

## 2. W19-1 で立てた 3 仮説

| 仮説 | 内容 | 検証コスト |
|---|---|---|
| A | endpoint 名違い (= 単数 / 複数の差 等) | 低 (= probe 1 回) |
| B | plan / subscription 制限 | 中 (= 上位 plan 申込必要) |
| C | deprecated / 別 endpoint で代替 | 中 (= V2 spec 再確認必要) |

## 3. V2 spec 確認 (= 静的調査範囲)

本 wave では **実 API call なし** で進めるため、以下の静的確認に限定:

1. **既存 F100 系 endpoint の動作確認** (= W18-1 で `/fins/statements/details`
   等は動作確認済 → base URL / 認証は問題なし)
2. **F100 が動く endpoint と F101 が動かない endpoint の比較** (= path
   構造差異の検出)
3. **endpoint 命名規則の推定** (= F100 は複数形 `/details` / `/summary`
   等が動作)

→ **endpoint 命名規則上、複数形 `/fins/announcements` が候補として浮上**。
ただし実 API call なしで確証は得られないため、**「候補リストを config 化
し、後続 wave で staging probe で確定」** が安全策。

## 4. candidate endpoint リスト (= 設計初版)

```python
ANNOUNCEMENT_ENDPOINT_DEFAULT = "/fins/announcement"

ANNOUNCEMENT_ENDPOINT_CANDIDATES = (
    "/fins/announcement",   # 現状 default (W18-2 で 403 確認)
    "/fins/announcements",  # 複数形候補 (= 仮説 A)
    # 将来追加候補 (= V2 spec / plan 確認後)
)
```

仮説 B (= plan 制限) と仮説 C (= deprecated) は **endpoint 名ではなく
plan / 別 API への切替** が必要なので、本 endpoint resolution の範囲外。
別 wave で対応。

## 5. resolution 戦略 (= L1b 副設計)

### 案 A: 単一 endpoint 切替 (= 推奨)

`JQUANTS_ANNOUNCEMENT_ENDPOINT` env var で endpoint を上書き可。
default は `/fins/announcement` 維持 (= backward compat)。

```python
def _resolve_announcement_endpoint(
    override: Optional[str] = None,
) -> str:
    if override:
        return override
    env_value = os.environ.get("JQUANTS_ANNOUNCEMENT_ENDPOINT")
    if env_value:
        return env_value
    return ANNOUNCEMENT_ENDPOINT_DEFAULT
```

**長所**:
- 1 つの endpoint しか試さない → 実 API call 数最小
- env で staging probe / production 切替容易
- ロジック単純

**短所**:
- 自動 fallback なし (= probe は人間 / runner が手動切替)

### 案 B: 順次試行 (= fallback 自動)

候補 endpoint を上から順に try、最初に 200 を返したものを採用。

```python
def _try_each_endpoint(...):
    for endpoint in ANNOUNCEMENT_ENDPOINT_CANDIDATES:
        try:
            response = self._request(..., path=endpoint)
            return response, endpoint
        except JQuantsAnnouncementError:
            continue
    raise ...
```

**長所**:
- 自動探索

**短所**:
- 実 API call が複数発生 (= 403 redundant、rate limit risk)
- 障害切分け困難 (= どの endpoint が成功したか log 必須)
- 本 wave の「実 API call 0」制約と相反

### 推奨: **案 A 単一切替**

理由:
- 実 API call は別 wave + HQ approve で限定実行
- env 切替で staging probe → production 切替が clean
- 案 B (自動 fallback) は staging で確定後に検討

## 6. implementation 案 (= W21-4 L3)

### 修正 file

- `materials/client.py` のみ

### 変更点

1. **モジュール定数追加**:
   ```python
   ANNOUNCEMENT_ENDPOINT_ENV = "JQUANTS_ANNOUNCEMENT_ENDPOINT"
   ANNOUNCEMENT_ENDPOINT_DEFAULT = "/fins/announcement"
   ANNOUNCEMENT_ENDPOINT_CANDIDATES: tuple[str, ...] = (
       "/fins/announcement",
       "/fins/announcements",
   )
   ```

2. **helper 関数追加**:
   ```python
   def _resolve_announcement_endpoint(
       override: Optional[str] = None,
   ) -> str:
       """Endpoint 解決優先度: override > env > default."""
       if override is not None and override != "":
           return override
       env_value = os.environ.get(ANNOUNCEMENT_ENDPOINT_ENV)
       if env_value:
           return env_value
       return ANNOUNCEMENT_ENDPOINT_DEFAULT
   ```

3. **`JQuantsAnnouncementClient.__init__` に `endpoint` 引数追加**:
   ```python
   def __init__(
       self,
       api_key: Optional[str] = None,
       base_url: str = JQUANTS_BASE_URL,
       endpoint: Optional[str] = None,  # ★ 新規
       ...
   ):
       ...
       self.endpoint = _resolve_announcement_endpoint(endpoint)
   ```

4. **`fetch_announcements` の path 参照変更**:
   ```python
   return self._request("GET", self.endpoint, params=params)
   ```

### backward compat

- `endpoint` 引数 default = None → resolve で default `/fins/announcement`
  → **既存挙動完全維持**
- env 未設定の運用は影響なし
- 引数 / env 設定で初めて切替

### 安全 (= W21-4 sub-task 固有)

- 実 API call 0 (= test は mock)
- token 値読出 0
- 既存 API シグネチャ (= fetch_announcements / fetch_all_announcements)
  維持
- 既存 contract 維持 (= JQuantsAuthError / JQuantsRateLimitError /
  JQuantsAnnouncementError 不変)

## 7. tests 設計 (= W21-5 L2)

### 新規 test file

`tests/materials/test_client_endpoint_resolution.py`:

1. `TestResolveDefault`: 引数 None + env 未設定 → default
2. `TestResolveOverride`: 引数指定 → override
3. `TestResolveEnv`: env 設定 → env value
4. `TestResolveOverridePriority`: 引数 + env 両指定 → 引数優先
5. `TestClientUsesResolvedEndpoint`: client 構築時に endpoint 反映確認
6. `TestFetchUsesEndpoint`: mock _request で endpoint パラメータ確認
7. `TestCandidatesListed`: 候補 tuple に default が含まれる
8. `TestEmptyOverrideTreatedAsDefault`: "" 渡し → default
9. `TestExistingApiSignatureMaintained`: 既存引数で構築 → 動作不変

mock は `unittest.mock.patch` + `MagicMock`。**実 HTTP リクエスト 0**。

### 既存 test 互換

既存 `tests/materials/test_client.py` (= 5 PASS) は **完全維持**。
新規追加で 9 件、合計 14 件想定。

## 8. audit 観点 (= W21-6 L4)

- A. 既存 API シグネチャ維持確認
- B. token 値読出ゼロ確認 (= os.environ.get(API_KEY_ENV) は構築時のみ、
  ログ出力なし)
- C. 実 HTTP 出力 0 確認 (= test 内 mock 完全性)
- D. backward compat (= default 不変)
- E. env name の typo / 衝突確認
- F. forbidden import 0
- G. file ownership 衝突 0 (= L3 = client.py / L2 = tests/materials/*)

## 9. 次 wave (= Wave 22 以降) の候補

本 endpoint resolution が merge された後:

- **Wave 22 W22-X**: staging probe 1 回実行 (= HQ 明示承認後、
  staging-only token、`JQUANTS_ANNOUNCEMENT_ENDPOINT=/fins/announcements`
  で試行)
- **Wave 22 W22-Y**: 成功 endpoint を default に切替 (= 別 commit)
- **Wave 23+**: 仮説 B / C の検討 (= plan 申込 or 代替 API)

## 10. 安全 (= 本 design doc 自体)

- 実 DB write 0
- 実 API call 0
- 実 LINE 送信 0
- token / secret 参照 0
- code 変更 0 (= 本 doc は設計のみ、impl は W21-4)
- cron / launchd 登録 0
- workflow 変更 0
- --no-verify 不使用
- TODO Excel 未更新

## 11. 関連リンク

- [[F101_API_403_investigation_2026-05-12|W19-1 F101 調査]]
- [[../02_todo/FIRE_CODEX_R1_WAVE21_plan|Wave 21 plan]]
- [[FIRE_CODEX_R2_10_lane_scaling_design_2026-05-12|R2 設計]]
- [[FIRE_CODEX_R2_codex_prompt_template_v1_2026-05-12|R2 prompt template v1.0]]
