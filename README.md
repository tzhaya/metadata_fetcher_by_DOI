# metadata_fetcher_by_DOI

DOIからUnpaywallのREST APIを検索してISSNとOAの場合のライセンスを取得し、このISSNからSherpa Serviceを検索して論文掲載誌のOAポリシーが記載されたURLを取得します。
あわせて、「日本の学協会の著作権ポリシー確認ツール」のREST APIへのリンクを生成します。

機関リポジトリの登録業務への活用を期待して作成しました。

動作にはMicrosoft ExcelとPower Queryを使用しています。

## Requirement
- Power queryが動作するExcel
- Sherpa ServicesのAPIキー
  - Sherpa Sericesを検索するため、APIキーが必要です。お持ちでない場合は https://v2.sherpa.ac.uk/cgi/register からアカウントを作成し、APIキーを入手してください。

## Setup
- APIキーの設定が必要です。初回の検索の際に資格情報の入力を求められます。「Web API」を選択して設定します。
  - または、「クエリ」→「編集」の「データソース設定」で設定できます。
- Unpaywallはメールアドレス、Sherpa Servicesは事前に取得したAPIキーで設定します。
![スクリーンショット 2024-07-15 205339_2](https://github.com/user-attachments/assets/8413275e-9070-4bd9-81f6-b82bddf73f6a)
- 初回の起動時に「プライバシーレベル」を設定する必要があります。「パブリック」に設定します。
![スクリーンショット 2024-07-25 135620](https://github.com/user-attachments/assets/0a4daeac-559e-4c9a-9897-dcf3494bbfce)


## Usage
1. シート「DOI」のA2セルに、検索したい文献のDOIを入力します。形式は 10.1038/s41587-024-02248-6 のようにしてください。
2. Excelメニューの「クエリ」を開き「クエリと接続」にある「クエリと接続」を押します。
![スクリーンショット 2024-07-26 084902](https://github.com/user-attachments/assets/6af7a8bb-1ddc-4bf7-8775-1ecfb0a91596)
3. 「クエリと接続」の「get UnPayWall」「get sherpa service」の順に更新します。
![スクリーンショット 2024-07-26 085054](https://github.com/user-attachments/assets/0e9d699f-8eb1-4ec6-9089-3d8592fd0eda)
4. 入力したDOIを元に、Unpaywallを検索してISSNを、このISSNからSherpa Servicesを検索して以下の情報を取得します。項目名に UnPayWall とあるものがUnpaywallから取得したデータです。
5. シート「get Unpaywall」にUnpaywallから取得したデータが収められています。これは動作確認用です。
6. Unpaywallに登録されていないDOI、またはISSNがSherpa Serviceでヒットしない場合はエラーとなります。

|DOI|uri|UnPayWall.issn|UnPayWall.journal_name|UnPayWall.article title|UnPayWall.is_os|UnPayWall.oa_status|UnPayWall.oa_location.license|scpj|
|:-|:-|:-|:-|:-|:-|:-|:-|:-|
|DOI|OAポリシー記載のURL(Sherpa Service)|論文掲載誌のISSN(Unpaywall)|掲載誌名(Unpaywall)|論題(Unpaywall)|DOIの先の論文がOAか否か（OAなら TRUE）(Unpaywall)|OAのステータス（gold, hybrid, bronze, green or closed）(Unpaywall)|DOIの先の論文がOAの場合のライセンス(Unpaywall)|「日本の学協会の著作権ポリシー確認ツール」(https://app.lib.shimane-u.ac.jp/policy_checker/scpj.php) へのリンク|
|10.1038/s41587-024-02248-6|https://v2.sherpa.ac.uk/id/publication/1643|1087-0156|Nature Biotechnology|High-throughput discovery of MHC class I- and II-restricted T cell epitopes using synthetic cellular circuits|TRUE|hybrid|cc-by|https://app.lib.shimane-u.ac.jp/policy_checker/scpj.php?mode=getPolicyFromIDs&ids=1087-0156|

## etc.
- その他の項目の取得が必要な場合は、適宜クエリを書き換えてご利用ください。

## links
- Unpaywall
  -   REST API : https://unpaywall.org/products/api
  -   Data Format : https://unpaywall.org/data-format
- Sherpa Services
  -  Metadata Schema : https://www.sherpa.ac.uk/api/metadata-schema.html
  -  Object Retrieval By ID API : https://v2.sherpa.ac.uk/api/object-retrieval-by-id.html

## Update

 - 2024/07/25
   - 「Formula.Firewall: クエリ ～ は他のクエリまたはステップを参照しているため、データ ソースに直接アクセスできません。このデータの組み合わせを再構築してください。」の表示がないようクエリをまとめるなど修正しました。
   - クエリをOffice データ接続ファイル形式で出力し /source におきました。テキスト形式でも置いています。

## To Do
- まとめて複数のDOIを検索したい。
- エラーチェックを組み込みたい。
- もうすこしコードをきれいにしたい。

## Author
Takanori Hayashi
