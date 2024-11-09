# metadata_fetcher_by_DOI

DOIからUnpaywallのREST APIを検索してISSNとOAの場合のライセンスを取得し、このISSNからSherpa Legacy APIを検索して論文掲載誌のOAポリシーが記載されたURLを取得します。
あわせて、「日本の学協会の著作権ポリシー確認ツール」経由で「学協会著作権ポリシーデータベース」（SCPJ）でのデータを取得します。

DOIがわかっている論文が、オープンアクセスかどうかを確認することができます。
機関リポジトリの登録業務や、文献複写業務などへの活用を期待して作成しました。

動作にはMicrosoft ExcelとPower Queryを使用しています。

## 動作環境
- Power queryが動作するExcel
- Sherpa ServicesのAPIキー
  - Sherpa Sericesは2024年11月から [Open policy finder](https://openpolicyfinder.jisc.ac.uk/) に変わりました。これに伴いAPIも変更が予定されているとのことです。
  - このサービスでは legacy Sherpa Services API を引き続き使用しています。APIキーの入手は直接問い合わせる必要があるようです。詳しくは https://openpolicyfinder.jisc.ac.uk/help/developers/use-our-api をご覧ください
  - ~~Sherpa Sericesを検索するため、APIキーが必要です。お持ちでない場合は https://v2.sherpa.ac.uk/cgi/register からアカウントを作成し、APIキーを入手してください。~~

## 初期設定
- APIキーの設定が必要です。初回の検索の際に資格情報の入力を求められます。「Web API」を選択して設定します。
  - または、「クエリ」→「編集」の「データソース設定」で設定できます。
- Unpaywallはメールアドレス、Sherpa Servicesは事前に取得したAPIキーで設定します。
![スクリーンショット 2024-07-15 205339_2](https://github.com/user-attachments/assets/8413275e-9070-4bd9-81f6-b82bddf73f6a)
- 初回の起動時に「プライバシーレベル」を設定する必要があります。「パブリック」に設定します。
![スクリーンショット 2024-07-25 135620](https://github.com/user-attachments/assets/0a4daeac-559e-4c9a-9897-dcf3494bbfce)

## 使い方
1. シート「DOI」の左端のテーブルに、検索したい文献のDOIを入力します。形式は 10.1038/s41587-024-02248-6 のようにしてください。
2. Excelメニューの「クエリ」を開き「クエリと接続」にある「すべて更新」を押します。
3. 入力したDOIを元に、Unpaywallを検索してISSNを、このISSNからSherpa Servicesと[日本の学協会の著作権ポリシー確認ツール](https://app.lib.shimane-u.ac.jp/policy_checker/scpj.php)（以下、「Policy Checker (SCPJ) 」といいます。）を検索して以下の表にある情報を取得します。項目名に UnPayWall とあるものがUnpaywallから取得したデータです。件数が多いと、取得には数秒以上かかります。
4. Unpaywallに登録されていないDOI、またはISSNがSherpa ServiceやPolicy Checker (SCPJ)でヒットしない場合は、セルに入力がされません。
5. 「すべて更新」でシートにあるすべてのDOIに対してもう一度検索が実行されます。実行後は、別のシートやファイルにデータを転記し、シート「DOI」には残さない運用をおすすめします。

|DOI|uri|Unpaywall.issn|Unpaywall.journal_name|Unpaywall.article title|Unpaywall.is_oa|Unpaywall.oa_status|Unpaywall.oa_location.license|Unpaywall.oa_location.url|Unpaywall.oa_location.url_for_pdf|ポリシー|Title|出版社版の利用|公開場所|公開条件|備考|
|:-|:-|:-|:-|:-|:-|:-|:-|:-|:-|:-|:-|:-|:-|:-|:-|
|DOI|OAポリシー記載のURL(Sherpa Service)|論文掲載誌のISSN(Unpaywall)|掲載誌名(Unpaywall)|論題(Unpaywall)|DOIの先の論文がOAか否か（OAなら TRUE）(Unpaywall)|OAのステータス（gold, hybrid, bronze, green or closed）(Unpaywall)|DOIの先の論文がOAの場合のライセンス(Unpaywall)|OAの場合のDOI解決先URL(Unpaywall)|OAの場合のPDFのURL(Unpaywall)|ポリシー(Policy Checker (SCPJ))|掲載誌名(Policy Checker (SCPJ))|出版社版利用可否(Policy Checker (SCPJ))|公開場所の指定(Policy Checker (SCPJ))|公開条件(Policy Checker (SCPJ))|備考(Policy Checker (SCPJ))|
|10.2964/jsik_2016_001|N/A|0917-1436|Joho Chishiki Gakkaishi|Structure analysis and a schema definition method for creating Linked Open Data from complex information resources|TRUE|bronze| |https://www.jstage.jst.go.jp/article/jsik/26/1/26_2016_001/_pdf|https://www.jstage.jst.go.jp/article/jsik/26/1/26_2016_001/_pdf|Green(査読前・査読後どちらでも認める)|情報知識学会誌|利用可能です|著者個人のWebサイト, 機関リポジトリ, 研究資金助成機関のWebサイト, 非営利電子論文アーカイブ|出典表示を行うこと|　|
|10.1038/7036|https://v2.sherpa.ac.uk/id/publication/1643|1087-0156|Nature Biotechnology|Improving plant drought, salt, and freezing tolerance by gene transfer of a single stress-inducible transcription factor|FALSE|closed||||ヒットなし|||||

## 動作の概要
1. テーブル DOI にあるDOIを取得します
2. カスタム関数 getUnpwall にDOIを渡します。この関数は、DOIをUnpaywallに送信し、結果のJSONから必要な項目を取り出してテーブルを返します。
3. 結果のテーブルを作成します。
4. カスタム関数 getSharpa に、結果のテーブルに含まれるISSN-Lを渡します。この関数は、ISSNをSherpa Serviceに送信し、取得したJSONから詳細情報が記載されたURL（のみ）を返します。
5. カスタム関数 getSharpa から返ってきたURLを、結果のテーブルに追加します。
6. カスタム関数 getSCPJdata に、結果のテーブルに含まれるISSN-Lを渡します。この関数は、ISSNをPolicy Checker (SCPJ)に送信し、結果のWebページのうちテーブル（のみ）を返します。
7. カスタム関数 getSCPJdata から返ってきたテーブルを、結果のテーブルに追加します。
8. これで完成です。

## その他
- その他の項目の取得が必要な場合は、クエリを適宜書き換えてご利用ください。
- 項目の追加・変更は、取得用の関数と、項目を展開するクエリ DOI 両方の書き換えが必要です。
- 存在しないDOI、Unpaywall未登録のDOI、ISSNがSherpa ServiceやPolicy Checker (SCPJ)でヒットしない場合は入力がない（null値または"N/A"など）と表示されます。

## リンク
- Unpaywall
  - 非営利団体 OurResearch によって運営されている、論文のオープンアクセスの状況を集約しているサービスです。
  -   REST API : https://unpaywall.org/products/api
  -   Data Format : https://unpaywall.org/data-format
- Open policy finder (Formerly Sherpa services)
  - 世界中の出版社のOA（オープンアクセス）ポリシーを集約・分析し、各ジャーナルのセルフアーカイビングの許諾、著者の権利条項などの概要を閲覧・検索できるオンラインツールSherpa ROMEOを含む検索サービスです。英国のJiscが運営しています。2024年11月4日に Open policy finder v1.0 としてリニューアルされました。
  -  Sherpa services https://openpolicyfinder.jisc.ac.uk/
  -  Sherpa Legacy API Documentation :  https://openpolicyfinder.jisc.ac.uk/sherpa_legacy_api.pdf
- 日本の学協会の著作権ポリシー確認ツール（島根大学附属図書館） : https://app.lib.shimane-u.ac.jp/policy_checker/scpj.php
  - 学協会著作権ポリシーデータベースの情報をソースとして、日本の学協会著作権ポリシーについてISSNやNCIDから検索できるツールです。
- 学協会著作権ポリシーデータベース（オープンアクセスリポジトリ推進協会（JPCOAR））： http://id.nii.ac.jp/1458/00000186/
  - 日本国内の学協会等のOA方針を調べられるデータベースです。

## 更新
 - 2024/08/03
   - 日本の学協会の著作権ポリシー確認ツールの結果を、URLではなくテーブルに直接展開するようにしました。これを実行する関数 getSCPJdata を追加しています。

 - 2024/07/28
   - 「クエリと接続」にある「すべて更新」の実行により、データを取得できるようになりました。
   - 一シートで動作を完結できるよう、コードを見直しました。これに伴い、シート「get UnPayWall」は削除されました。
   - 複数のDOIをまとめて取得できるようになりました。
   - 存在しないDOI、Unpaywall未登録のDOI、ISSNがSherpa Serviceでヒットしない場合は入力がない（null値または"N/A"）ように修正しました。

 - 2024/07/25
   - 「Formula.Firewall: クエリ ～ は他のクエリまたはステップを参照しているため、データ ソースに直接アクセスできません。このデータの組み合わせを再構築してください。」の表示がないようクエリをまとめるなど修正しました。
   - クエリをOffice データ接続ファイル形式で出力し /source におきました。テキスト形式でも置いています。

## To Do
- Open policy finder APIへの対応 [issue #5](https://github.com/tzhaya/metadata_fetcher_by_DOI/issues/5) 
- ~~まとめて複数のDOIを検索したい~~ [issue #2](https://github.com/tzhaya/metadata_fetcher_by_DOI/issues/2) で対応. 複数行で検索できます。
- ~~エラーチェックを組み込みたい~~[issue #3](https://github.com/tzhaya/metadata_fetcher_by_DOI/issues/3) で対応。存在しないDOIはnullとしてテーブルに出力します。
- ~~もうすこしコードをきれいにしたい~~[issue #2](https://github.com/tzhaya/metadata_fetcher_by_DOI/issues/2) で対応。

## 作者
林 賢紀　（Takanori Hayashi）
