# Google Workspace CSE 復号ツール 利用ガイド

Google Workspace のクライアントサイド暗号化（CSE）で暗号化されたデータ（メール・ファイル）を復号するためのツールの利用ガイドです。

---

## 1. 前提・システム要件

### 対応OS
- **macOS**: 12 (Monterey) 以降（Apple シリコン / Intel 両対応）
- **Windows**: 10 / 11 (64ビット版)
- **Linux**: x86_64

### 対応データ形式
- **Gmail CSE メッセージ**: **MBOX 形式のみ**となります。
- **Google ドキュメント/スプレッドシート/スライド**: `.gdoczip` などのアーカイブ形式。
- **注意**: **PST 形式**のエクスポートデータには対応していません。

---

## 2. 事前準備（IdPの設定）

復号には、鍵アクセス制御リストサービス（KACLS）へアクセスするための認証が必要です。利用している IdP（Google または Entra ID）に応じて以下の準備を行ってください。

> [!IMPORTANT]
> **クライアントIDについて**: 
> 既存の CSE 設定（IdP側）で使用しているクライアント ID とは別に、**本ツール専用のクライアント ID（インストール型アプリケーション用）**を別途作成する必要があります。

### Google IdP の場合（GCP）
1. Google Cloud コンソールで新しいプロジェクト（または既存のプロジェクト）を選択します。
2. 「API とサービス」 > 「認証情報」から、**OAuth クライアント ID** を作成します。
3. アプリケーションの種類として「**デスクトップ アプリ**」を選択してください。

### Entra IdP の場合（Azure）
1. Microsoft Entra 管理センターで「エンタープライズ アプリケーション」を新規登録します。
2. 本ツール専用のアプリケーションとして構成し、「アプリケーション (クライアント) ID」を取得します。

![ntra ID 設定画面](image-3.png)
*図：Entra ID でのクライアント ID 取得例（アプリケーション ID がクライアント ID となります）*

---

## 3. Google Vault からのデータ取得

復号対象のデータは Google Vault から取得します。

1. Google Vault で検索を実行します。
2. 暗号化メールを検索する場合、キーワードに `filename:smime.p7m` を指定すると効率的です。
3. エクスポート時の形式として必ず **MBOX** を選択してください。


---

---

## 4. 復号の実行

### Step 1: 認証設定ファイルの作成
IdP の情報を保存した設定ファイル（`idp_config.json`）を作成します。

```bash
# 基本的な作成コマンドの例
./decrypter -action createconfig -config idp_config.json \
  -email your-email@example.com \
  -client_id <取得したクライアントID> \
  -issuer <各IdPのソースURL>
```

**設定例（Entra ID の場合）:**
```json
{
  "kacls_url": "https://cse.example.com",
  "client_id": "1df1d8a4-aa62-4ce5-bd40-96fd27148d5f",
  "issuer": "https://login.microsoftonline.com/<テナントID>/v2.0",
  "discovery_uri": "https://login.microsoftonline.com/<テナントID>/v2.0/.well-known/openid-configuration",
  "grant_type": "authorization_code"
}
```

### Step 2: 復号コマンドの実行
作成した設定ファイルを指定して復号を実行します。

```bash
./decrypter -config idp_config.json -input ./cse-maildata -output ./decrypted_results
```

実行時、ブラウザが起動して認証・認可を求められます。管理者アカウントで承認してください。

| 認証要求 | 認証成功 |
| :---: | :---: |
| (画像削除済み) | ![認証成功](image-1.png) |

復号が完了すると、ターミナルに結果が表示されます。

(画像削除済み)

---

## 5. 復号データの閲覧方法

復号された MBOX 形式のデータは、**Mozilla Thunderbird** とアドオンを使用して閲覧できます。

### 必要なツール
- **Mozilla Thunderbird**: [公式サイト](https://www.thunderbird.net/)からダウンロード。
- **ImportExportTools NG**: Thunderbird 内で「MBOX 形式」をインポートするためのアドオン。

### 取り込み手順
1. Thunderbird を起動し、ローカルフォルダーを右クリックします。
2. 「**ImportExportTools NG**」 > 「**mbox ファイルをインポート**」を選択します。
3. 復号結果のディレクトリ内にある mbox ファイルを選択して取り込みます。

![Thunderbird インポート](image-4.png)
*図：ImportExportTools NG を使用したインポート操作。*

---

## 付録：主要フラグ一覧

| フラグ | 説明 |
| :--- | :--- |
| `-help` | すべてのフラグのヘルプを表示 |
| `-config <file>` | 設定ファイル（JSON）を指定 |
| `-input <path>` | 復号するファイルまたはディレクトリ |
| `-output <path>` | 復号結果の保存先ディレクトリ |
| `-overwrite` | 既存のファイルを上書きする |
| `-credential <file>`| サービスアカウントの JSON キー（Gmail S/MIME 復号用） |
| `-action info` | ファイルの暗号化状態などを確認するモード |
