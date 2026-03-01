# SHERPA/RoMEO API 新旧比較レポート

## 比較対象

| ソース | ファイル | 説明 |
|---|---|---|
| 旧APIレスポンス | `jc-import-file-maker/samples/sherpa_publication_1643_1087-0156.json` | ISSN 1087-0156 (Nature Biotechnology) |
| 新APIレスポンス | `source/sample/1546-1696.json` | ISSN 1546-1696 (Nature Biotechnology, 同一雑誌) |
| 新APIレスポンス | `source/sample/2156-5376.json` | ISSN 2156-5376 (Advances in Nutrition, 別雑誌) |
| 新APIスキーマ (publication) | `source/sample/schema_publication.json` | Open Policy Finder API publication スキーマ |

---

## 1. 旧APIレスポンス vs 新APIレスポンス（同一雑誌での比較）

### 結論：構造・内容ともに完全に同一

同一雑誌 Nature Biotechnology のデータを旧API (ISSN 1087-0156) と新API (ISSN 1546-1696) で取得した結果、レスポンス構造に差異はなかった。

- 同一の `publisher_policy` id:7145
- 同一の `permitted_oa` id:10695/10696/11461/11462
- 同一の publisher id:3286 (Nature Research)
- フィールドの過不足なし

**APIエンドポイントが変わっただけで、レスポンス構造には変更なし。**

### 異なる雑誌間での差異（Nature Biotechnology vs Advances in Nutrition）

`items[]` 直下のフィールドで、雑誌データの違いに起因する差異：

| フィールド | Nature Biotechnology | Advances in Nutrition | 備考 |
|---|---|---|---|
| `tj_status` / `tj_status_phrases` | あり | なし | TJ対象雑誌のみ |
| `permitted_oa[].embargo` | あり | なし | エンバーゴがある雑誌のみ |
| `permitted_oa[].copyright_owner` / `_phrases` | あり | なし | 明示する場合のみ |
| `permitted_oa[].publisher_deposit` | なし | あり | PMC deposit がある場合のみ |
| `publisher.imprint_of_id` | あり | なし | imprint の場合のみ |
| `publisher.imprints_id` | なし | あり | imprint を持つ場合のみ |

これらはすべてオプショナルフィールドであり、API仕様変更による差異ではない。

---

## 2. 新APIスキーマ (publication) vs 実際のレスポンス

### スキーマに記載されているが、レスポンスに存在しないフィールド

| フィールド | スキーマ定義 | 備考 |
|---|---|---|
| `title[].acronym` | あり | 略称がない雑誌では省略 |
| `title[].preferred` | あり | 同上 |
| `notes` | あり | notesがない雑誌では省略 |
| `publishers[].publisher.issns` | あり | レスポンスに存在しない |
| `publishers[].publisher.type` | あり | レスポンスに存在しない |
| `permitted_oa[].license[].version` | あり | レスポンスに存在しない |
| `publisher_policy[].urls[].type` | あり | レスポンスに存在しない |

### レスポンスに存在するが、スキーマに記載されていないフィールド

| フィールド | 備考 |
|---|---|
| `system_metadata` | id, uri, date_created, date_modified, publicly_visible 等 |
| `type_phrases` | items直下 |
| `issns[].type_phrases` | |
| `title[].language_phrases` | |
| `publishers[].publisher.url` | |
| `publishers[].publisher.imprint_of_id` | |
| `publishers[].publisher.name` が配列形式 | スキーマではテキスト型 |
| `permitted_oa[].embargo` | `{units, amount, units_phrases}` |
| `permitted_oa[].location.named_repository` | |

### 重要な型不一致：`publishers[].publisher.name`

| 定義元 | 型 |
|---|---|
| 新APIスキーマ | `"name": "text"` （単一テキスト） |
| 実際のレスポンス | `"name": [{name, language, language_phrases}]` （配列） |

スキーマドキュメントが publication エンドポイント内の埋め込み publisher を簡略表記している可能性がある。

---

## 3. 総合結論

| 観点 | 結論 |
|---|---|
| 旧API → 新API のレスポンス変更 | **なし**（同一構造・同一データ） |
| スキーマの正確性 | **不完全** — `_phrases`系、`system_metadata`、`embargo`、`named_repository` 等が未記載 |
| スキーマとの型不一致 | `publisher.name` がスキーマではテキスト、実データでは配列 |
| コード移行への影響 | レスポンス構造が同一のため、**既存のパース処理はそのまま流用可能** |
| 注意点 | スキーマを信じてコードを新規作成する場合、実レスポンスとの乖離に注意が必要 |
