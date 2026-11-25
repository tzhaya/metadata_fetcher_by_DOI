# CLAUDE.md

DOI から Unpaywall・Open policy finder・SCPJ（島根大学のツール経由）の情報を取得する Excel / Power Query ツール。

## ファイル構成と正本

- **正本は配布 Excel（`*.xlsx`）内の Power Query コード**。`source/` 配下は Excel から手作業でエクスポートした写しにすぎない。
  - `source/*.txt`：クエリ・カスタム関数のテキスト版
  - `source/クエリ - *.odc`：Office データ接続ファイル形式のエクスポート
- `source/*.txt` と xlsx は同期がずれやすい。過去に `source/getSCPJhtml.txt` で、存在しないステップを参照する転記漏れがあった。
- `source/legacy/` と `source/ver0.1/` は旧版の保管場所。現行コードとして扱わない。

## 配布 Excel の分岐

- 配布 Excel は版ごとにクエリ `DOI` が分岐している。
  - 通常版：Open policy finder の呼び出し（`getOpenpolicyfinder` + パラメーター `OpfApiKey`）があり、`uri` 列を出力する。
  - ss 版（Skip Sherpa service）：Open policy finder の呼び出しがなく、`uri` 列もない。
- 共通の関数（`getUnpaywall` など）を変更したときは、すべての版の xlsx と `source/` の両方に反映する。

## xlsx 内のコードを確認する方法

Power Query のコードは `customXml/item1.xml` の `<DataMashup>` 要素に入っている。このファイルは UTF-16 で書かれている。要素の中身は base64 エンコードされたバイナリで、先頭 4 バイトがバージョン、次の 4 バイトがパッケージ長、その後に ZIP が続く。ZIP 内の `Formulas/Section1.m` にすべてのクエリがある。

```python
import zipfile, base64, re, io, struct, sys
z = zipfile.ZipFile(sys.argv[1])
t = z.read("customXml/item1.xml").decode("utf-16")
b = base64.b64decode(re.search(r"<DataMashup[^>]*>([^<]+)</DataMashup>", t).group(1))
ln = struct.unpack("<I", b[4:8])[0]
print(zipfile.ZipFile(io.BytesIO(b[8:8 + ln])).read("Formulas/Section1.m").decode("utf-8-sig"))
```

## xlsx 内のコードを更新する方法

- xlsx のクエリ更新は、Excel の COM オブジェクトモデル（`Workbook.Queries`）を PowerShell から操作して行う。Excel 自身が保存するため、クエリのメタデータ、接続、シート上のテーブルの整合性が保たれる。
  - 置き換え：`$wb.Queries.Item("DOI").Formula = $m`
  - 追加：`$wb.Queries.Add("getOASIS", $m, "説明")`
  - 削除：`$wb.Queries.Item("getSCPJdata").Delete()`
- 流し込むコードは `source/*.txt` から読み込む。こうすると、正本の xlsx と `source/` が一致する。
- 作業時の注意：
  - 対象の xlsx は Excel で開いていない状態にする。
  - 保存前に元ファイルのバックアップを取る（OneDrive の同期と保存が衝突しないよう、作業中は同期状況に注意する）。
  - `Excel.Application` は `Quit()` と `ReleaseComObject` で確実に終了させる。
- 更新後は、xlsx から抽出したコードが `source/*.txt` と一致することを確認する。資格情報・プライバシーレベル・「すべて更新」の動作は、Excel 上で手動で確認する。
- DataMashup（`customXml/item1.xml`）をスクリプトで直接書き換えない。メタデータや改ざん検知用の領域との整合が取れず、ファイルが壊れる危険がある。

## plan 作成時の確認事項

- 変更が全版の xlsx に及ぶか（通常版・ss 版のどちらで、クエリ `DOI` のどの分岐か）。
- `source/*.txt` と `.odc` の更新・移動（旧関数は `source/legacy/` へ）が手順に入っているか。
- `source/*.txt` を根拠にする前に、xlsx から抽出したコードと突き合わせたか。
- 出力列を変更する場合、README の出力項目表・例示行・「動作の概要」・更新履歴を更新するか。
- 外部サービス（Unpaywall、Open policy finder、島根大学のツール）の仕様変更や移行が関係する場合、公式の告知を確認したか。
- 利用者側の初期設定（資格情報、プライバシーレベル、APIキー）に影響するか。Formula.Firewall エラーが出ないか。
