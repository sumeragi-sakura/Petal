# Petal Email Client

Petalは、Windows向けに開発されたAL-Mailライクなメールクライアントです。`almail`ブランチのコードベースから読み解いた仕様に基づき、古いAL-Mail環境の移行サポートと、現代的なメール管理機能を両立します。

## 概要

- AL-Mailのデータ移行を支援するメールソフト
- IMAP/POP3受信、SMTP送信、OAuth2(Google)認証に対応
- ローカルSQLiteデータベースでメッセージとアカウントを管理
- Tkinter/TkinterWebを使ったGUIを提供

## 主要機能

### 1. AL-Mail移行機能

- `ALMAIL.INI` からアカウントサーバー設定を抽出してPetalのアカウントへ登録
- `ALMAIL.ADR` からアドレス帳をインポート
- AL-Mailのメールボックスフォルダ（`Mail`配下の`.al`ファイル）を読み取り、件名／送信者／受信者／本文／HTML本文／添付ファイルをSQLiteに保存
- フォルダ名をPetal側のフォルダ名に変換して再現（Inbox / Sent / Drafts / Trash等）
- 起動時に標準的なAL-Mailインストールディレクトリを探索し、インポートを促す自動探索機能

### 2. マルチアカウント管理

- `accounts` テーブルで複数アカウントを保持
- 各アカウントに対してIMAP/POP3受信、SMTP送信設定を保存
- `use_oauth2` フラグによりGoogle OAuth2認証を利用可能
- 受信通知、トレイアイコン最小化、自動受信間隔をアカウント単位で設定

### 3. メール表示／セキュリティ解析

- 受信メールのプレーンテキスト/HTML本文を保存
- `Authentication-Results` や `X-Authentication-Results` からSPF/DKIM/DMARC結果を抽出
- ドメイン不一致検出によるなりすまし警告表示
- Googleセキュリティアラートを自動判別し、デバイス／場所／時間をカード形式で表示
- Google Meet URL抽出とカレンダー登録用リンク生成

### 4. UIと通知

- Tkinterベースのメインウィンドウ
- コンテキストメニュー、右クリックメニュー、ドラッグ対応の入力欄
- `pystray`を使ったシステムトレイアイコン対応（最小化時にトレイへ格納）
- `win10toast`を使った新着メール通知表示

## 実行環境と依存関係

### 必要なPythonパッケージ

- google-auth-oauthlib
- google-api-python-client
- keyring
- pywin32-ctypes
- tkinterweb
- Pillow
- pystray
- win10toast

### 実行方法

1. Python環境を準備
2. `requirements.txt` をインストール

```powershell
python -m pip install -r requirements.txt
```

3. `modern_almail.py` を実行

```powershell
python modern_almail.py
```

- 開発実行時はソースフォルダ内に`google_client_secrets.json`を配置するとOAuth機能が有効になります
- 実行ファイル化した場合、`%APPDATA%\Petal` 以下に `mailbox.db` が作成されます

## データベース構成

PetalはSQLiteを利用し、以下のテーブルを管理します。

- `messages`
  - メール本文、HTML本体、送信者/受信者、フォルダ、認証結果、ヘッダー、添付情報など
- `accounts`
  - メールアカウントのサーバー情報、UI設定、OAuthフラグ、自動受信設定など
- `address_book`
  - 連絡先の名前、メールアドレス、ニックネーム
- `attachments`
  - 添付ファイルのバイナリデータ
- `app_versions`
  - インストールバージョン履歴
- `window_settings`
  - ウィンドウ位置と状態の保存
- `app_config`
  - 全体設定キー/値

## 使い方

### 1. アカウント追加

- メニューまたはツールバーからアカウントを追加
- IMAP/POP3の受信サーバーとSMTP送信サーバーを設定
- `OAuth2 (Google)` タブからGoogle認証を実行すると、パスワード入力不要でGmailへの接続が可能
- `自動受信する` を有効にすると、指定した分おきに受信を自動実行
- `HTMLメールをテキストで表示する` を有効にすると、HTML本文をテキストとして表示

### 2. AL-Mailデータのインポート

- `AL-Mailフォルダをインポート...` メニューからインポート先アカウントを選択
- AL-Mailのメールボックスフォルダを指定して `.al` ファイルを読み込む
- 既定のAL-Mailパスが存在する場合、起動時にインポート確認ダイアログが表示される

### 3. メールの表示と解析

- メールを選択すると本文とHTMLプレビューを表示
- `Authentication-Results` に基づき、SPF/DKIM/DMARCの結果を評価してバナー表示
- Googleアラートメールを検知すると、デバイス・場所・時間を抽出して専用形式で表示
- Google Meet URLが本文に含まれる場合、Googleカレンダー登録機能を利用可能

## ビルド手順

`build_app.bat` を実行すると、以下の処理が自動で行われます。

1. Python実行環境の検出
2. 依存ライブラリの検証と必要ならインストール
3. `modern_almail.py` 内の `APP_VERSION` を更新
4. `version.json` の生成
5. `Petal_icon.png` から `Petal_icon.ico` の生成
6. PyInstaller によるバイナリビルド
7. `move_exe.bat` による生成物の移動
8. Inno Setup を使ったインストーラー作成（`ISCC.exe` が存在する場合）

### 注意点

- `google_client_secrets.json` がない場合、Google OAuth関連機能は無効になります
- `petal_installer.iss` を使ったインストーラー作成には Inno Setup のインストールが必要です
- `pystray` と `win10toast` はWindowsトレイおよび通知機能のために必要です

## 参考

- `modern_almail.py`: メインGUIとアプリケーション制御
- `petal_core.py`: DB初期化、アプリ設定、Google認証情報読み込み
- `petal_utils.py`: HTML/テキスト処理、アドレス抽出、セキュリティ解析補助
- `petal_security.py`: SPF/DKIM/DMARC判定ロジックと表示向けメッセージ
- `almail_importer.py`: SQLite DB 作成、AL-Mail設定/アドレス帳/メールインポート、OAuthトークン管理

---

(c) 2026 Petal Development Project.
