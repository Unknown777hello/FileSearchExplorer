タイトル：【FileSearchExplorer解説 #5】1行ずつ解説：83-114行目
## はじめに

こんにちはUnknown777です。

今回は、FileSearchExplorerのコードを1行ずつ解説します。

現在、アンケートを実施しています。ぜひご回答ください。

→ https://forms.cloud.microsoft/Pages/ResponsePage.aspx?id=DQSIkWdsW0yxEjajBLZtrQAAAAAAAAAAAANAAqvV7klURUg5VzlVNUhGTFFWVjlRNlo4RDFJOVZOWi4u

コードは以下で公開しています。ReleasesにPython版も置いています。

→ https://github.com/Unknown777hello/FileSearchExplorer

## 本編

- [`QUICK_COMMAND_MAX_RESULTS = 30`]：クイック検索の小窓に一度に表示する検索結果の最大件数を定義し、変数に代入します。
- [`SPLASH_MIN_DISPLAY_MS = 1200`]：アプリ起動時に表示するスプラッシュスクリーンの最低表示時間をミリ秒単位で設定し、変数に代入します。この場合は1.2秒です。
- [`ENCRYPTED_DB_PATH = INDEX_DB_PATH + ".enc"`]：検索インデックスのデータベースファイルのパスに「.enc」を結合し、暗号化されたデータベースファイルのフルパスを変数に代入します。
- [`def _default_key_dir():`]：暗号鍵（`index.key`）を保存するディレクトリのパスを返す関数を定義します。
- [`if os.name == "nt":`]：現在実行されているOSがWindowsかどうかを判定する条件分岐です。
- [インデント内の `base = os.environ.get("LOCALAPPDATA") or os.path.join(os.path.expanduser("~"), "AppData", "Local")`]：Windowsの環境変数`LOCALAPPDATA`を取得します。取得できない場合は、ホームディレクトリ内の`AppData\Local`のパスを組み立て、変数`base`に代入します。
- [インデント内の `return os.path.join(base, "FileSearchExplorerKeys")`]：基準となるパスに`FileSearchExplorerKeys`というフォルダ名を結合し、Windows環境での暗号鍵の保存先パスとして返します。
- [`base = os.environ.get("XDG_CONFIG_HOME") or os.path.join(os.path.expanduser("~"), ".config")`]：Windows以外の環境で、環境変数`XDG_CONFIG_HOME`を取得します。取得できない場合は、ホームディレクトリ内の`.config`のパスを組み立て、変数`base`に代入します。
- [`return os.path.join(base, "file_search_explorer_keys")`]：基準となるパスに`file_search_explorer_keys`を結合し、Windows以外の環境での暗号鍵の保存先パスとして返します。
- [`KEY_DIR = _default_key_dir()`]：関数`_default_key_dir()`を実行し、OSに応じた暗号鍵用ディレクトリのパスを取得して、変数`KEY_DIR`に代入します。
- [`ENCRYPTION_KEY_PATH = os.path.join(KEY_DIR, "index.key")`]：暗号鍵用ディレクトリのパスとファイル名`index.key`を結合し、暗号鍵ファイルのフルパスを作成します。
- [`LEGACY_ENCRYPTION_KEY_PATH = os.path.join(OLD_APP_DATA_DIR, "index.key")`]：旧バージョンの保存先ディレクトリとファイル名`index.key`を結合し、移行前の暗号鍵ファイルのパスを作成します。
- [`SETTINGS_PATH = os.path.join(SETTINGS_DIR, "settings.json")`]：設定ファイル用ディレクトリのパスとファイル名`settings.json`を結合し、設定ファイルのフルパスを作成します。
- [`APP_LOG_PATH = os.path.join(APP_DATA_DIR, "app_errors.log")`]：アプリケーションデータ用ディレクトリのパスとファイル名`app_errors.log`を結合し、エラーログを保存するファイルのフルパスを作成します。
