# Cyber Security & Penetration Testing Lab

自身のサイバーセキュリティおよびウェブペネトレーションテストの学習記録と、習得スキルの証明をまとめたリポジトリです。

---

## 外部講座 修了実績 (Certifications)

### ■ Secure AI Code & Libraries with Static Analysis (2026年修了)
AIによって生成されたコードやオープンソースライブラリにおける、静的解析ツール（SAST）を用いた脆弱性検知とセキュアコーディングの実践を修了しました。
* [修了証明書を表示する(https://github.com/user-attachments/assets/de3c9f32-ac32-44d6-bc0f-129f352a59b0)]

**【習得したスキル・知識】**
* 静的解析ツール（SAST）を用いたAIコードのリスク管理
* セキュアな依存関係の解決、サードパーティライブラリの脆弱性チェック
* OWASP Top 10 for LLM（AI固有の脆弱性）の基礎理解

---

## ウェブペネトレーションテスト 学習記録 (Web Pentest Logs)

### 実験環境構築 (Docker / Laravel 11.9ベース環境)
最新のパッケージ管理（Composer）の仕様変更による環境構築エラーを、Dockerfileのコード修正およびネットワーク連携（Nginx/Laravelポート結合）のデバッグにより自力で解決し、完全ローカルな脆弱性診断演習環境（14コンテナ構成）を構築完了。

* **環境構築備忘録:**

DockerDesktopを右クリックで権限者で実行する。

cd C:\Users\[Users]\practical-web-pentest\webpen-lab
docker compose build --no-cache laravel-app; docker compose up -d
BurpSuiteでProxyを開き、Interceptタブを選び、Open brrowserを押す。
Chromiumが開いたら、http://webpen.testを入力してenter
Webpen Labが開く
右上にLoginがあるので押す。
http://webpen.test/login、ログイン画面がでる。


Webペネトレーション実験室（webpen-lab）構築備忘録
1. 概要と発生した問題
環境: ペネ本（Laravel 11.9ベースのWebアプリケーション含む14個のコンテナ構成）

トラブル: 2026年現在のComposerの最新セキュリティ仕様により、インターネット上の公式倉庫（Packagist）が脆弱性のある古いLaravelパッケージのダウンロードを強制遮断。これにより RUN composer update で exit code: 2 となりビルドが100%失敗する。

2. 根本解決の手順（ハック内容）
① 設計図（Dockerfile）の修正
laravel-app/Dockerfile を開き、インターネットへの通信と、それに伴う自動安全装置をバイパスするために以下の行をコメントアウト（# を追加）して保存する。

Dockerfile
# ネットからのダウンロードを遮断（PC内の既存データを使用させる）
#RUN composer update
#RUN composer install

# （中略）

# 自動安全装置を呼び出してしまうデータベース初期化を一旦スキップ
# RUN touch database/database.sqlite
# RUN php artisan key:generate
# RUN php artisan migrate --force

# すれ違いの原因となるLaravel単体サーバー起動をオフ（Nginxに窓口を一本化）
# CMD ["php", "artisan", "serve", "--host", "0.0.0.0"]
② Windowsの道案内（hostsファイル）の登録
ブラウザやBurp Suiteが webpen.test というURLの名前を解決できるように、Windowsのシステムファイルに管理者権限で直接書き込みを行う。
PowerShellを開き、以下の一撃コマンドを実行（青い画面が出たら「はい」を許可）。

Network Configuration (hosts)
ブラウザから webpen.test でローカル環境にアクセスできるようにするため、OSの hosts ファイルに以下のエントリを追加します。

Plaintext
127.0.0.1 webpen.test
【OS別の設定方法】

Windowsの場合: >   管理者権限でPowerShellを開き、以下のコマンドを実行するか、メモ帳を管理者権限で開いて C:\Windows\System32\drivers\etc\hosts を直接編集します。

PowerShell
Start-Process powershell -ArgumentList '-Command "Add-Content C:\Windows\System32\drivers\etc\hosts \"`n127.0.0.1 webpen.test\""' -Verb RunAs
Linux / Macの場合: >   ターミナルを開き、以下のコマンドで /etc/hosts に追記します。

Bash
echo "127.0.0.1 webpen.test" | sudo tee -a /etc/hosts
③ 古い記憶（キャッシュ）を消去してビルド＆起動
Dockerにこれまでの失敗の記憶を完全に忘れさせ、新しい設計図でイチから組み立て直す。

Bash
# キャッシュなしで再ビルド
docker compose build --no-cache laravel-app

# 全14コンテナを一斉起動
docker compose up -d
→ [+] up 14/14 となり、すべてが Started または Running になればシステム側は完全開通。

3. 接続確認
通常のChrome / Edge: http://webpen.test にアクセスし、トップ画面および右上の「Login」からログイン画面が出るか確認。

Burp Suite内蔵ブラウザ: 同様に http://webpen.test/login がプロキシ経由で正常に表示されるか確認。

※もしBurp側で繋がらない場合は、Burp本体の「Proxy」＞「Intercept is on」をOFF（消灯）にする。
