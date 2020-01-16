BookSharing
2020/1/14-16

## 発生した問題
DBのseedに仕込んだ初期データが本番環境で反映されなかった。画像のアップロードでエラーが発生した

## 原因の特定
heroku logs -t コマンドでログ確認

## 問題の解決
### herokuにプッシュできない
原因ハッキリせず。とりあえずプッシュできたときの状態に戻す。Devise導入時につくったDevise関連のファイルをすべて消去した。git cloneが使えなかったことも解決までの時間が長引いた要因か。
### [fog][deprecation] のエラーがfogファミリー全部に発生。「fog::storage::aws」は廃止されてるから「fog::aws::storage」を使え、的なメッセージ大量。
どうやらfogのバージョンアップに伴って発生している。バージョンをRailsチュートリアル第13章のものに固定。有効な解決策ではないかも。
### Access Denied
carrier_wave.rbにconfic.fog_public = falseの一文を書き加えた。fog_publicというのは、パブリックオブジェクトとしてアップロードするかどうかの設定（？）らしい。意味はわからん。
> config/initializers/carrier_wave.rb
```ruby
if Rails.env.production?
  CarrierWave.configure do |config|
    config.fog_credentials = {
      # Amazon S3用の設定
      :provider              => 'AWS',
      :region                => ENV['S3_REGION'],     # 例: 'ap-northeast-1'
      :aws_access_key_id     => ENV['S3_ACCESS_KEY'],
      :aws_secret_access_key => ENV['S3_SECRET_KEY']
    }
    config.fog_directory     =  ENV['S3_BUCKET']
+    config.fog_public = false
  end
end
```
### InvalidSignatureException: The request signature we calculated does not match the signature you provided.
AWSのアクセスキーとシークレットキーを発行して再設定した。
