# HDD価格履歴の保存仕様

定期調査の各回を `data/YYYY/MM/YYYY-MM-DDTHH-mm-ss+0900.json` にUTF-8 JSONで保存する。時刻は日本時間。既存ファイルは上書きしない。同じ実行の再試行では重複保存しない。

## JSON構造（v1）

- `schema_version`: 1
- `run_id`: 実行を一意に識別する文字列
- `observed_at`: ISO 8601形式の実行日時（+09:00）
- `market_results`: 配列。各要素は `market`, `status`（success / partial / failed）, `queries`, `notes` を持つ。
- `observations`: 商品・販売店・販売条件ごとの観測値の配列。各要素は `observed_at`, `market`, `seller`, `listing_id`, `product_name`, `manufacturer`, `model`, `capacity_gb`, `type`（internal / external / unknown）, `condition`（new / used / refurbished / recertified / unknown）, `item_price_jpy`, `shipping_jpy`, `total_price_jpy`, `jpy_per_gb`, `stock`（in_stock / out_of_stock / unknown）, `url`, `evidence_url`, `verification`（confirmed / partial / unverified）, `notes` を持つ。

不明値は null とし、送料不明を0円にしない。税込価格・送料が確定した場合のみ合計と円/GBを計算する。容量は1TB=1000GB。検索一覧のみで確認した情報と個別ページで確認した情報を区別する。購入可能と確認した新品だけを主ランキングに使用する。価格不明・売り切れ・予算超過も観測として保存可能だが、主ランキングには含めない。検索できなかった商品を売り切れと判定しない。

取得できた観測を上位5件に限定せず保存し、価格不変でも観測を追加する。調査失敗時も取得状況と空配列を保存し、架空の観測は作らない。公開リポジトリのため秘密情報、個人情報、認証情報は保存しない。

将来のグラフでは容量・型番・状態・販売店・市場別の比較を行い、観測範囲が変動することを明記する。検索結果の統計は市場全体の代表値ではない。
