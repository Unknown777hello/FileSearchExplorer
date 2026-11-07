タイトル：【FileSearchExplorer解説 #7】1行ずつ解説：152-1009行目## はじめに
こんにちはUnknown777です。 
今回は、FileSearchExplorerのコードを1行ずつ解説します。
また、前回のように1ブログあたり100行で解説を進めると合計39本程度のブログを書かなければいけなくなってしまうため、ペースを上げて1ブログあたり1000行程度で進めていきたいと思います。

現在、アンケートを実施しています。ぜひご回答ください。 
→https://forms.cloud.microsoft/Pages/ResponsePage.aspx?id=DQSIkWdsW0yxEjajBLZtrQAAAAAAAAAAAANAAqvV7klURUg5VzlVNUhGTFFWVjlRNlo4RDFJOVZOWi4u
コードは以下で公開しています。ReleasesにPython版も置いています。 
→https://github.com/Unknown777hello/FileSearchExplorer
## 注意事項

本記事は、FileSearchExplorerのソースコードを理解しやすくすることを目的とした解説記事です。

記事の作成にはGoogle Geminiなどの生成AIを活用していますが、生成された文章をそのまま掲載するのではなく、自分でも文章の編集・修正を行っています。

ただし、記事の分量が多いため、説明の誤りや不正確な表現が残っている可能性があります。できる限り正確な解説を心がけていますが、すべての記述の正確性を保証するものではありません。

より正確に理解したい場合は、実際のソースコードや公式ドキュメントも併せてご確認ください。

本記事が、プログラムの仕組みを学ぶきっかけになれば幸いです。
## 本編
#### 第1章：設定ファイルの安全なアトミック書き込み（バックアップ処理）

directory = os.path.dirname(path) or "."：保存先ファイルのパスからディレクトリのパスを取り出し、パスが空の場合はカレントディレクトリ（.）を代入します。
if keep_backup and os.path.exists(path):：バックアップを保持する設定であり、かつ保存先のファイルがすでに存在するかどうかを判定する条件分岐です。
(インデント)try:：既存ファイルのバックアップ作成中に発生する可能性のあるOSエラーを捕捉するための例外処理ブロックを開始します。
(インデント)backup_path = path + ".bak"：既存ファイルのバックアップ用として、元のファイル名に .bak を追加したフルパスを作成します。
(インデント)tmp_bak_fd, tmp_bak_path = tempfile.mkstemp(prefix=".tmpbak_", dir=directory)：安全にバックアップ用の一時ファイルを同じディレクトリ内に作成し、ファイルディスクリプタとファイルパスを取得します。
(インデント)os.close(tmp_bak_fd)：あとでファイル操作を行うため、一時的に開いたファイルディスクリプタをいったん閉じます。
(インデント)with open(path, "rb") as src, open(tmp_bak_path, "wb") as dst:：元ファイルを読み込みモード（バイナリ）、バックアップ用一時ファイルを書き込みモード（バイナリ）で同時に開きます。
(インデント)dst.write(src.read())：元のファイルの中身をすべて読み込み、そのままバックアップ用の一時ファイルへ書き込みます。
(インデント)os.replace(tmp_bak_path, backup_path)：バックアップ用の一時ファイルを正式な .bak ファイルへ置き換えます。
(インデント)if restrict_permissions:：アクセス権限の制限を行う設定になっているかどうかを判定する条件分岐です。
(インデント)restrict_file_permissions(backup_path)：作成したバックアップファイルのアクセス権限を制限するための処理を実行します。
(インデント)except OSError as exc:：バックアップ作成に関連してOSエラーが発生した場合に、その例外を捕捉します。
(インデント)log_error("設定バックアップ作成", f"{path}: {exc}")：発生したエラーの内容をアプリケーションのエラーログに記録します。
tmp_path = None：書き込み用の一時ファイルパスを管理する変数を、初期状態として None に設定します。
try:：本体のJSON書き込みおよびファイル置き換え処理で発生するエラーを捕捉するための例外処理ブロックを開始します。
tmp_fd, tmp_path = tempfile.mkstemp(prefix=".tmp", dir=directory)：データを書き込むための新しい一時ファイルを同じディレクトリ内に作成し、ディスクリプタとパスを取得します。
with os.fdopen(tmp_fd, "w", encoding="utf-8") as f:：作成した一時ファイルを開き、UTF-8エンコーディングのテキスト書き込み用ファイルオブジェクトとして扱います。
json.dump(data, f, ensure_ascii=False, indent=2)：Pythonのデータをインデント付きの整形されたJSON形式にして一時ファイルに書き込みます。
(インデント)f.flush()：OSのバッファにたまっている書き込みデータをファイルへフラッシュします。
(インデント)os.fsync(f.fileno())：ファイル記述子に対してOSの同期処理を要求します。
os.replace(tmp_path, path)：書き込み完了した一時ファイルを、本来のファイルパスへ上書き置き換えします。
tmp_path = None：一時ファイルの処理が完了したため、管理用変数を None に戻します。
_fsync_directory(directory)：ファイルが格納されているディレクトリ自体の変更内容の同期を試みます。
if restrict_permissions:：保存したファイルのアクセス権限を制限する設定が有効か判定します。
(インデント)_restrict_file_permissions(path)：保存された本来の設定ファイルのアクセス権限を制限する処理を実行します。
finally:：例外の発生有無に関わらず、処理の最後に必ず実行されるクリーンアップブロックを開始します。
(インデント)if tmp_path is not None and os.path.exists(tmp_path):：何らかのエラーによって一時ファイルが残ったままになっていないかを判定します。
(インデント)(インデント)try:：残存した不要な一時ファイルを削除する際のエラー処理ブロックを開始します。
(インデント)(インデント)(インデント)os.remove(tmp_path)：不要になった一時ファイルをディスクから削除します。
(インデント)(インデント)except OSError:：削除時に発生する可能性のあるOSエラーを捕捉します。
(インデント)(インデント)(インデント)pass：削除エラーが発生した場合でも処理を継続します。
def _lazy_import_crypto():：外部モジュール（cryptography）のインポートを実際に必要になる直前まで遅延させるための関数を定義します。
(インデント)if 'crypto' not in _LAZY_MODULE_CACHE:：遅延読み込み用キャッシュに暗号化モジュールがまだ含まれていないかを判定します。
(インデント)(インデント)try:：モジュールのインポート時に発生する例外を捕捉するブロックを開始します。
(インデント)(インデント)(インデント)from cryptography.fernet import Fernet, InvalidToken：cryptographyパッケージから必要なクラスや例外クラスをインポートします。
(インデント)(インデント)(インデント)_LAZY_MODULE_CACHE['crypto'] = (Fernet, InvalidToken)：インポートしたモジュールの機能をキャッシュに保存します。
(インデント)(インデント)except ImportError:：モジュールが未導入である場合のインポートエラーを捕捉します。
(インデント)(インデント)(インデント)_LAZY_MODULE_CACHE['crypto'] = None：モジュールが存在しないことを示すため、キャッシュに None を格納します。
(インデント)return _LAZY_MODULE_CACHE['crypto']：キャッシュに保存された暗号化モジュールの参照を返却します。
def crypto_available():：暗号化機能が利用可能かどうかを判定する関数を定義します。
(インデント)try:：モジュール存在確認時に発生する可能性のあるエラーを捕捉するブロックを開始します。
(インデント)(インデント)import importlib.util：実際のインポートを行わずにモジュールの有無を調べるための標準ライブラリをインポートします。
(インデント)(インデント)return importlib.util.find_spec("cryptography.fernet") is not None：cryptography.fernetが存在するかどうかを確認し、その真偽値を返します。
(インデント)except (ImportError, ValueError, ModuleNotFoundError):：存在確認中に発生した各種モジュール関連のエラーを捕捉します。
(インデント)(インデント)return False：エラーが発生した場合は「利用不可（False）」を返します。
def _lazy_import_pil():：画像処理用ライブラリ（Pillow）の読み込みを遅延させるための関数を定義します。
(インデント)if 'pil' not in _LAZY_MODULE_CACHE:：Pillow関連のモジュールがまだキャッシュされていないかを判定します。
(インデント)(インデント)try:：Pillowの各モジュールをインポートする際の例外処理ブロックを開始します。
(インデント)(インデント)(インデント)from PIL import Image, ImageTk, UnidentifiedImageError：画像処理、Tkinter用変換、および不正画像エラーの各クラスをインポートします。
(インデント)(インデント)(インデント)_LAZY_MODULE_CACHE['pil'] = (Image, ImageTk, UnidentifiedImageError)：インポートしたPillowの構成要素をキャッシュに格納します。
(インデント)(インデント)except ImportError:：Pillowが未導入である場合のインポートエラーを捕捉します。
(インデント)(インデント)(インデント)_LAZY_MODULE_CACHE['pil'] = None：未導入であることを示すためキャッシュに None を設定します。
(インデント)return _LAZY_MODULE_CACHE['pil']：キャッシュからPillowの参照を返却します。
def pil_available():：Pillowが利用可能かどうかを判定する関数を定義します。
(インデント)return _lazy_import_pil() is not None：遅延インポートの結果が None でなければ True を返します。
def _lazy_import_plyer():：デスクトップ通知用ライブラリ（plyer）の読み込みを遅延させる関数を定義します。
(インデント)if 'plyer' not in _LAZY_MODULE_CACHE:：plyerがまだキャッシュに存在しないかを判定します。
(インデント)(インデント)try:：plyerの通知モジュールをインポートするブロックを開始します。
(インデント)(インデント)(インデント)from plyer import notification as plyer_notification：plyerから通知機能モジュールを別名でインポートします。
(インデント)(インデント)(インデント)_LAZY_MODULE_CACHE['plyer'] = plyer_notification：インポートした通知モジュールをキャッシュに保存します。
(インデント)(インデント)except ImportError:：未導入時のインポートエラーを捕捉します。
(インデント)(インデント)(インデント)_LAZY_MODULE_CACHE['plyer'] = None：未導入としてキャッシュに None を設定します。
(インデント)return _LAZY_MODULE_CACHE['plyer']：キャッシュから通知モジュールの参照を返します。
def plyer_available():：plyerによる通知機能が利用可能かを判定する関数を定義します。
(インデント)return _lazy_import_plyer() is not None：通知モジュールが読み込み済みであれば True を返します。
def _lazy_import_keyboard():：キーボードフック用モジュールの読み込みを遅延させる関数を定義します。
(インデント)if 'keyboard' not in _LAZY_MODULE_CACHE:：keyboardモジュールがキャッシュに未登録かを判定します。
(インデント)(インデント)try:：keyboardモジュールのインポートブロックを開始します。
(インデント)(インデント)(インデント)import keyboard as keyboard_module：キーボード操作を検知するモジュールをインポートします。
(インデント)(インデント)(インデント)_LAZY_MODULE_CACHE['keyboard'] = keyboard_module：インポートしたモジュールをキャッシュに保存します。
(インデント)(インデント)except ImportError:：未導入時の例外を捕捉します。
(インデント)(インデント)(インデント)_LAZY_MODULE_CACHE['keyboard'] = None：未導入として None をキャッシュします。
(インデント)return _LAZY_MODULE_CACHE['keyboard']：キャッシュからモジュール参照を返します。
def keyboard_hook_available():：キーボードフック機能が利用可能かを判定する関数を定義します。
(インデント)return _lazy_import_keyboard() is not None：モジュールが利用可能なら True を返します。
def _lazy_import_pystray():：システムトレイ用モジュールの読み込みを遅延させる関数を定義します。
(インデント)if 'pystray' not in _LAZY_MODULE_CACHE:：pystrayがキャッシュに未登録かを判定します。
(インデント)(インデント)try:：pystrayのインポートブロックを開始します。
(インデント)(インデント)(インデント)import pystray as pystray_module：システムトレイ用モジュールをインポートします。
(インデント)(インデント)(インデント)_LAZY_MODULE_CACHE['pystray'] = pystray_module：キャッシュにモジュールを保存します。
(インデント)(インデント)except ImportError:：未導入エラーを捕捉します。
(インデント)(インデント)(インデント)_LAZY_MODULE_CACHE['pystray'] = None：未導入として None をキャッシュします。
(インデント)return _LAZY_MODULE_CACHE['pystray']：キャッシュからトレイモジュールの参照を返します。
def pystray_available():：システムトレイ機能が利用可能かを判定する関数を定義します。
(インデント)return _lazy_import_pystray() is not None：トレイモジュールが利用可能なら True を返します。
def tray_feature_available():：システムトレイと画像処理（Pillow）の両方が揃っているかを判定する関数を定義します。
(インデント)return pystray_available() and pil_available()：トレイ機能とPillowの両方が利用可能な場合のみ True を返します。
def load_session_state():：セッション状態をファイルから読み込むための関数を定義します。
(インデント)try:：ファイル読み込みおよびJSON解析時のエラーを捕捉するブロックを開始します。
(インデント)(インデント)with open(SESSION_STATE_PATH, "r", encoding="utf-8") as f:：セッション状態ファイルを読み込み専用のUTF-8エンコーディングで開きます。
(インデント)(インデント)(インデント)data = json.load(f)：ファイル内のJSONデータをPythonのオブジェクトに変換して読み込みます。
(インデント)(インデント)if isinstance(data, dict):：読み込んだデータが辞書型であるかを検証します。
(インデント)(インデント)(インデント)return data：辞書型であればそのままデータを返却します。
(インデント)except (OSError, ValueError) as exc:：ファイル不存在やJSON破損などのエラーを捕捉します。
(インデント)(インデント)if os.path.exists(SESSION_STATE_PATH):：セッションファイルが実際に存在しているかを判定します。
(インデント)(インデント)(インデント)log_error("セッション状態読込", f"{exc}\n{traceback.format_exc()}")：ファイルが存在するのに読み込めなかった場合、エラー内容と詳細なスタックトレースをログに記録します。
(インデント)return {}：読み込みに失敗した場合は空の辞書を返します。
def save_session_state(state):：現在のセッション状態をファイルに保存するための関数を定義します。
(インデント)try:：保存処理中のエラーを捕捉するブロックを開始します。
(インデント)(インデント)os.makedirs(APP_DATA_DIR, exist_ok=True)：アプリ用データディレクトリが存在しない場合は新しく作成します。
(インデント)(インデント)_atomic_write_json(SESSION_STATE_PATH, state, restrict_permissions=True)：セッション状態をアトミック書き込みでJSONファイルとして保存します。
(インデント)except OSError as exc:：保存時エラーを捕捉します。
(インデント)(インデント)log_error("セッション状態保存", f"{exc}\n{traceback.format_exc()}")：保存エラーの内容とスタックトレースをログに記録します。

#### 第2章：プレビュー拡張子の定義と標準設定データの保持

TEXT_PREVIEW_EXTENSIONS = {：テキストとしてプレビュー表示を許可するファイル拡張子の集合の定義を開始します。
(インデント)'htm', 'ini', 'c', 'html', 'json', 'cpp', 'go', 'php', 'css', 'log', 'txt',：プレビュー対象となる各種コード・ドキュメント・設定ファイルの拡張子を指定しています。
(インデント)'rb', 'csv', 'yaml', 'xml', 'bat', 'sh', 'ts', 'rs', 'md', 'java', 'yml', 'py', 'h', 'js'：同じくテキストプレビューに対応するプログラム言語やデータの拡張子を指定し、集合を閉じます。
IMAGE_PREVIEW_EXTENSIONS = {：画像としてプレビュー表示を許可するファイル拡張子の集合の定義を開始します。
(インデント)'flc', 'ppm', 'pixar', 'heif', 'eps', 'jpeg', 'msp', 'mpo', 'tiff', 'jp2', 'jpx',：対応する多様な画像フォーマットの拡張子を指定しています。
(インデント)'j2k', 'png', 'wal', 'qoi', 'dds', 'xpm', 'pcx', 'im', 'jpe', 'pcd', 'jfif', 'ico',：同じく各種画像・アイコン形式の拡張子を指定しています。
(インデント)'sgi', 'xbm', 'icns', 'avif', 'bmp', 'dib', 'psd', 'pnm', 'jpg', 'pgm', 'tif', 'fli',：さらに多くの静止画やアニメーション画像の拡張子を指定しています。
(インデント)'tga', 'jpf', 'webp', 'cur', 'gif', 'heic', 'pbm'：残りの画像拡張子を指定し、集合を閉じます。
DEFAULT_SETTINGS = {：アプリケーション全体で使用されるデフォルトの設定値をまとめた辞書の定義を開始します。
(インデント)'preview_enabled': False,：プレビュー機能の有効・無効の既定値を無効（False）に設定します。
(インデント)'keyword_history': [],：検索キーワードの履歴を保持するリストの初期値を空に設定します。
(インデント)'bookmarks': [],：ブックマーク一覧を保持するリストの初期値を空に設定します。
(インデント)'theme': 'light',：アプリケーションのカラーテーマの既定値をライトテーマ（'light'）に設定します。
(インデント)'last_folder': '',：最後に検索したフォルダパスの初期値を空文字列に設定します。
(インデント)'recursive': True,：サブフォルダを含めて再帰的に検索する設定の既定値を有効（True）に設定します。
(インデント)'ignore_case': True,：大文字と小文字を区別せずに検索する設定の既定値を有効（True）に設定します。
(インデント)'ignore_width': True,：全角と半角を区別せずに検索する設定の既定値を有効（True）に設定します。
(インデント)'include_system': False,：システムファイルを検索対象に含めるかどうかの既定値を無効（False）に設定します。
(インデント)'search_target': 'name',：検索対象のターゲット（ファイル名）の既定値を指定します。
(インデント)'search_folders_too': True,：検索結果にフォルダ自体も含めるかどうかの既定値を有効（True）に設定します。
(インデント)'exact_match': False,：完全一致検索を行うかどうかの既定値を無効（False）に設定します。
(インデント)'match_mode': 'partial',：一致方式の既定値を部分一致（'partial'）に設定します。
(インデント)'use_index': False,：インデックスを利用した高速検索を行うかどうかの既定値を無効（False）に設定します。
(インデント)'include_hidden': False,：隠しファイルを検索対象に含めるかどうかの既定値を無効（False）に設定します。
(インデント)'content_search': False,：ファイル内容の検索を行うかどうかの既定値を無効（False）に設定します。
(インデント)'window_geometry': '1000x650',：ウィンドウの初期サイズおよび位置の既定値を指定します。
(インデント)'recent_files': [],：最近開いたファイルの履歴を保持するリストの初期値を空に設定します。
(インデント)'recent_search_folders': [],：最近検索したフォルダの履歴を保持するリストの初期値を空に設定します。
(インデント)'search_presets': [],：検索プリセットのリストの初期値を空に設定します。
(インデント)'notify_sound_enabled': True,：通知音を有効にするかどうかの既定値を有効（True）に設定します。
(インデント)'notify_sound_type': 'builtin',：通知音の種類（組み込み音）の既定値を指定します。
(インデント)'notify_sound_builtin': 'SystemAsterisk',：組み込み通知音の具体的なサウンド名の既定値を指定します。
(インデント)'notify_sound_file': '',：カスタム通知音のファイルパスの初期値を空に設定します。
(インデント)'notify_desktop_enabled': False,：デスクトップ通知の有効・無効の既定値を無効（False）に設定します。
(インデント)'notify_inapp_enabled': True,：アプリ内通知の有効・無効の既定値を有効（True）に設定します。
(インデント)'search_time_history': [],：検索に要した時間の履歴を保持するリストの初期値を空に設定します。
(インデント)'result_columns_visible': ['type', 'dir', 'size', 'mtime', 'match'],：検索結果一覧で表示する列の初期リストを指定します。
(インデント)'result_column_widths': {},：結果一覧の各列の幅を保持する辞書の初期値を空に設定します。
(インデント)'result_sort_column': None,：結果一覧のソート対象列の初期値を未指定（None）に設定します。
(インデント)'result_sort_reverse': False,：ソート順を逆順にするかどうかの既定値を無効（False）に設定します。
(インデント)'live_reindex_enabled': False,：ライブ再インデックス機能の既定値を無効（False）に設定します。
(インデント)'keyword_suggest_enabled': True,：キーワード候補サジェスト機能の既定値を有効（True）に設定します。
(インデント)'ui_font_family_cache': '',：UIフォント名のキャッシュの初期値を空文字列に設定します。
(インデント)'ui_font_detected': False,：UIフォントが検出済みかどうかのフラグの既定値を無効（False）に設定します。
(インデント)'custom_mode_checked_features': [],：「難しいモード」でチェックされた機能キーのリストの初期値を空に設定します。
(インデント)'quick_command_enabled': True,：クイックコマンド機能の既定値を有効（True）に設定します。
(インデント)'quick_command_folder': '',：クイックコマンド小窓で最後に使ったフォルダの初期値を空に設定します。
(インデント)'minimize_to_tray_enabled': False,：ウィンドウを最小化・閉じた際にシステムトレイへ格納する機能の既定値を無効に設定します。
(インデント)'content_search_extensions': [],：内容検索の対象拡張子リストの初期値を空に設定します。
(インデント)'parallel_worker_count': 0,：並列スレッド数の既定値を自動算出（0）に設定します。
(インデント)'low_load_mode': False,：低負荷モードの有効・無効の既定値を無効に設定します。
(インデント)'max_stored_results_override': 0,：検索結果の保持件数上限の上書き値の既定値を未指定（0）に設定します。
(インデント)'warn_removable_drive': True,：リムーバブルドライブ検索時の確認ダイアログ表示の既定値を有効に設定します。
(インデント)'folder_sets': [],：複数対象フォルダセットのリストの初期値を空に設定します。
(インデント)'search_history': [],：検索履歴のリストの初期値を空に設定します。
(インデント)'index_root_options': {},：ルートごとのインデックスオプションを保持する辞書の初期値を空に設定します。
(インデント)'index_auto_optimize_last': '',：自動最適化を最後に実行した日時の初期値を空文字列に設定します。
}：デフォルト設定を格納する辞書の定義を終了します。

#### 第3章：「難しいモード」の定義とグループ化・検索定数の設定

第3章：「難しいモード」の定義とグループ化・検索定数の設定
CUSTOM_FEATURE_DEFS = [：「難しいモード」で個別に有効化できる各機能の定義リストの開始を宣言します。
(インデント)('ext', "拡張子検索"),：拡張子を指定して検索する機能のキーと表示ラベルを定義します。
(インデント)('exclude_ext', "除外拡張子検索"),：特定の拡張子を除外して検索する機能のキーと表示ラベルを定義します。
(インデント)('size', "ファイルサイズ検索"),：ファイルサイズを条件に指定して検索する機能のキーと表示ラベルを定義します。
(インデント)('content', "ファイル内容検索"),：ファイルの中身をテキスト検索する機能のキーと表示ラベルを定義します。
(インデント)('mtime', "更新日時検索"),：ファイルの更新日時を条件に指定する機能のキーと表示ラベルを定義します。
(インデント)('ctime', "作成日時検索"),：ファイルの作成日時を条件に指定する機能のキーと表示ラベルを定義します。
(インデント)('exclude_folders', "除外フォルダ検索"),：特定のフォルダを検索から除外する機能のキーと表示ラベルを定義します。
(インデント)('match_mode', "一致方法"),：検索キーワードの一致方式を選択する機能のキーと表示ラベルを定義します。
(インデント)('multi_folder', "複数フォルダ検索"),：複数のフォルダを同時に検索対象にする機能のキーと表示ラベルを定義します。
(インデント)('bookmarks', "ブックマーク"),：ブックマーク機能のキーと表示ラベルを定義します。
(インデント)('presets', "プリセット"),：検索プリセット機能のキーと表示ラベルを定義します。
]：カスタム機能の定義リストを終了します。
CUSTOM_FEATURE_KEYS = {key for key, _label in CUSTOM_FEATURE_DEFS}：定義されたカスタム機能のキーのみを効率よく抽出してセットを作成します。
GROUP_MODE_LABELS = [：検索結果をグループ化するためのプルダウン表示用ラベルのリストの開始を宣言します。
(インデント)('none', "グループ化なし"), ('type', "種類別"), ('ext', "拡張子別"),：グループ化しない設定および種類別・拡張子別のグループ化ラベルを定義します。
(インデント)('folder', "フォルダ別"), ('mtime', "更新日別"), ('size', "サイズ帯別"),：フォルダ別・更新日別・サイズ帯別のグループ化ラベルを定義し、リストを終了します。
GROUP_MAX_ENTRIES = 10000：グループ化処理を行う際の最大エントリ数の制限値を定義します。
MTIME_GROUP_ORDER = ["今日", "昨日", "今週(7日以内)", "今月(30日以内)", "それ以前", "更新日不明"]：更新日別グループの並び順を規定するリストを定義します。
SIZE_GROUP_ORDER = ["フォルダ", "空(0B)", "1MB未満", "1MB〜100MB", "100MB〜1GB", "1GB以上", "サイズ不明"]：サイズ帯別グループの並び順を規定するリストを定義します。
MAX_SEARCH_HISTORY = 30：保持する検索履歴の最大件数を定義します。
RECENT_CHANGE_CHOICES = [("24時間以内", 24), ("3日以内", 72), ("7日以内", 168), ("30日以内", 720)]：最近の変更フィルタで使用する時間枠の選択肢を定義します。
RESULT_COLUMN_TOGGLABLE = ('type', 'dir', 'size', 'mtime', 'match')：結果一覧で表示・非表示を切り替え可能な列名のタプルを定義します。
MAX_SEARCH_TIME_HISTORY = 200：保持する検索時間履歴の最大件数を定義します。
HASH_ALGORITHMS = ('md5', 'sha1', 'sha256')：利用可能なハッシュアルゴリズムの種類を定義します。
TREEVIEW_ROW_HEIGHT = 30：検索結果一覧の各行の高さをピクセル単位で定義します。
WINSOUND_BUILTIN_ALIASES = [：Windowsの組み込み通知音のエイリアスと日本語表示名のリストの開始を宣言します。
(インデント)('SystemAsterisk', "通知音（アスタリスク）"),：アスタリスク通知音の識別子と表示名を定義します。
(インデント)('SystemExclamation', "警告音"),：警告音の識別子と表示名を定義します。
(インデント)('SystemHand', "エラー音"),：エラー音の識別子と表示名を定義します。
(インデント)('SystemQuestion', "質問音"),：質問音の識別子と表示名を定義します。
(インデント)('SystemDefault', "既定のシステム音"),：既定のシステム音の識別子と表示名を定義し、リストを終了します。

#### 第4章：起動モード（Lite / Normal / Max）とテーマカラーの定義

HOME_SCREEN_MODES = [：起動時のモード選択画面に表示する3つのモードの定義リストの開始を宣言します。
(インデント){'key': 'lite', 'label': "Lite版", 'description': "機能を絞り、検索速度を最優先します",：Lite版の識別キー、ラベル、および説明文を定義します。
(インデント)(インデント)'selectable': True, 'coming_soon_message': None},：Lite版が選択可能であることを指定し、未実装メッセージを無効にして辞書を閉じます。
(インデント){'key': 'normal', 'label': "Normal版", 'description': "機能と検索速度のバランスを重視した標準モードです",：Normal版の識別キー、ラベル、および説明文を定義します。
(インデント)(インデント)'selectable': True, 'coming_soon_message': None},：Normal版が選択可能であることを指定し、未実装メッセージを無効にして辞書を閉じます。
(インデント){'key': 'max', 'label': "Max版", 'description': "利用可能な機能をすべて使用できます",：Max版の識別キー、ラベル、および説明文を定義します。
(インデント)(インデント)'selectable': True, 'coming_soon_message': None},：Max版が選択可能であることを指定し、未実装メッセージを無効にして辞書を閉じ、リストを終了します。
MODE_FEATURES = {：各モードごとの機能の有効・無効を管理する設定マッピングの定義を開始します。
(インデント)'lite': {：Lite版における各機能の有効・無効フラグの辞書定義を開始します。
(インデント)(インデント)'advanced_panel': False,：高度な検索パネルを無効に設定します。
(インデント)(インデント)'match_mode_extra': False,：高度な一致モードを無効に設定します。
(インデント)(インデント)'content_search': False,：ファイル内容検索を無効に設定します。
(インデント)(インデント)'preview': False,：プレビュー機能を無効に設定します。
(インデント)(インデント)'hash_calc': False,：ハッシュ計算機能を無効に設定します。
(インデント)(インデント)'duplicate_finder': False,：重複ファイル検索機能を無効に設定します。
(インデント)(インデント)'multi_folder_search': False,：複数フォルダ検索機能を無効に設定します。
(インデント)(インデント)'bookmarks': False,：ブックマーク機能を無効に設定します。
(インデント)(インデント)'presets': False,：検索プリセット機能を無効に設定します。
(インデント)(インデント)'search_time_history': False,：検索時間履歴機能を無効に設定します。
(インデント)(インデント)'notification_settings': False,：通知設定機能を無効に設定します。
(インデント)(インデント)'extra_columns': False,：結果一覧の追加表示列を無効に設定します。
(インデント)(インデント)'permission_error_panel': False,：アクセス権限エラーパネルを無効に設定します。
(インデント)(インデント)'result_tools': False,：結果の操作ツールを無効に設定します。
(インデント)(インデント)'quick_folders': False,：検索範囲のクイック選択を無効に設定し、辞書を閉じます。
(インデント)},：Lite版の設定ブロックを終了します。
(インデント)'normal': {：Normal版における各機能の有効・無効フラグの辞書定義を開始します。
(インデント)(インデント)'advanced_panel': True,：高度な検索パネルを有効に設定します。
(インデント)(インデント)'match_mode_extra': False,：高度な一致モードを無効に設定します。
(インデント)(インデント)'content_search': True,：ファイル内容検索を有効に設定します。
(インデント)(インデント)'preview': True,：プレビュー機能を有効に設定します。
(インデント)(インデント)'hash_calc': False,：ハッシュ計算機能を無効に設定します。
(インデント)(インデント)'duplicate_finder': False,：重複ファイル検索機能を無効に設定します。
(インデント)(インデント)'multi_folder_search': True,：複数フォルダ検索機能を有効に設定します。
(インデント)(インデント)'bookmarks': True,：ブックマーク機能を有効に設定します。
(インデント)(インデント)'presets': False,：検索プリセット機能を無効に設定します。
(インデント)(インデント)'search_time_history': False,：検索時間履歴機能を無効に設定します。
(インデント)(インデント)'notification_settings': True,：通知設定機能を有効に設定します。
(インデント)(インデント)'extra_columns': True,：追加表示列を有効に設定します。
(インデント)(インデント)'permission_error_panel': True,：アクセス権限エラーパネルを有効に設定します。
(インデント)(インデント)'result_tools': True,：結果の操作ツールを有効に設定します。
(インデント)(インデント)'quick_folders': True,：検索範囲のクイック選択を有効に設定し、辞書を閉じます。
(インデント)},：Normal版の設定ブロックを終了します。
(インデント)'max': {：Max版における各機能の有効・無効フラグの辞書定義を開始します。
(インデント)(インデント)'advanced_panel': True,：高度な検索パネルを有効に設定します。
(インデント)(インデント)'match_mode_extra': True,：高度な一致モードを有効に設定します。
(インデント)(インデント)'content_search': True,：ファイル内容検索を有効に設定します。
(インデント)(インデント)'preview': True,：プレビュー機能を有効に設定します。
(インデント)(インデント)'hash_calc': True,：ハッシュ計算機能を有効に設定します。
(インデント)(インデント)'duplicate_finder': True,：重複ファイル検索機能を有効に設定します。
(インデント)(インデント)'multi_folder_search': True,：複数フォルダ検索機能を有効に設定します。
(インデント)(インデント)'bookmarks': True,：ブックマーク機能を有効に設定します。
(インデント)(インデント)'presets': True,：検索プリセット機能を有効に設定します。
(インデント)(インデント)'search_time_history': True,：検索時間履歴機能を有効に設定します。
(インデント)(インデント)'notification_settings': True,：通知設定機能を有効に設定します。
(インデント)(インデント)'extra_columns': True,：追加表示列を有効に設定します。
(インデント)(インデント)'permission_error_panel': True,：アクセス権限エラーパネルを有効に設定します。
(インデント)(インデント)'result_tools': True,：結果の操作ツールを有効に設定します。
(インデント)(インデント)'quick_folders': True,：検索範囲のクイック選択を有効に設定し、辞書を閉じます。
(インデント)},：Max版の設定ブロックを終了します。
}：全モードの機能設定マッピングを終了します。
DEFAULT_APP_MODE = 'max'：デフォルトのアプリケーション起動モードをMax版に設定します。
BRAND_COLOR = '#2954A6'：アプリケーションのブランドカラーとなるメインの青色コードを定義します。
BRAND_COLOR_ACTIVE = '#1D3E80'：ボタンが押された際などのアクティブ状態のブランドカラーを定義します。
BRAND_COLOR_LIGHT_TINT = '#E3ECFA'：薄い色合いのアクセント用ティントカラーを定義します。
BRAND_COLOR_DARK_TINT = '#3B6FC4'：やや明るめのダーク向けブランドカラーを定義します。
THEME_COLORS = {：ライトテーマとダークテーマの各UIカラーをまとめた辞書の定義を開始します。
(インデント)'light': {：ライトテーマにおける各種カラー設定の定義を開始します。
(インデント)(インデント)'bg': '#f5f6f8', 'fg': '#1b1d21', 'field_bg': '#ffffff',：背景色、文字色、入力フィールドの背景色を指定します。
(インデント)(インデント)'select_bg': BRAND_COLOR, 'muted_fg': '#6b7078', 'accent_tint': BRAND_COLOR_LIGHT_TINT,：選択時の背景色、控えめな文字色、アクセント用ティントを指定します。
(インデント)(インデント)'border': '#dcdfe5', 'row_alt': '#f8f9fb', 'hover': '#eceef2',：境界線の色、リストの交互の行の背景色、マウスホバー時の色を指定し、辞書を閉じます。
(インデント)},：ライトテーマの定義ブロックを終了します。
(インデント)'dark': {：ダークテーマにおける各種カラー設定の定義を開始します。
(インデント)(インデント)'bg': '#1b1c1f', 'fg': '#e6e7ea', 'field_bg': '#25272b',：背景色、文字色、入力フィールドの背景色を指定します。
(インデント)(インデント)'select_bg': BRAND_COLOR_DARK_TINT, 'muted_fg': '#9aa0a8', 'accent_tint': '#28345A',：選択時の背景色、控えめな文字色、アクセント用ティントを指定します。
(インデント)(インデント)'border': '#3a3d44', 'row_alt': '#2b2d32', 'hover': '#33363c',：境界線の色、リストの交互の行の背景色、マウスホバー時の色を指定し、辞書を閉じます。
(インデント)},：ダークテーマの定義ブロックを終了します。
}：テーマカラー辞書の定義を終了します。

#### 第5章：UIフォント検出関数と最小限の設定読み込み関数

_UI_FONT_FAMILY = None：検出されたUIフォントのファミリー名をキャッシュするためのグローバル変数を初期化します。
def get_ui_font_family(cached_value=None, cached_detected=False):：UIフォントを取得・検出する関数を定義します。
(インデント)global _UI_FONT_FAMILY：関数内でグローバル変数 _UI_FONT_FAMILY を参照・更新することを宣言します。
(インデント)if _UI_FONT_FAMILY is not None:：すでにプロセス内でUIフォントが特定・キャッシュされているかを判定します。
(インデント)(インデント)return _UI_FONT_FAMILY：キャッシュ済みのフォントファミリー名が存在する場合は即座にそれを返却します。
(インデント)if cached_detected:：設定ファイル等ですでにフォント判定が完了しているフラグが立っているかを判定します。
(インデント)(インデント)_UI_FONT_FAMILY = cached_value or ""：キャッシュされたフォント名をグローバル変数に設定します。
(インデント)(インデント)return _UI_FONT_FAMILY：設定ファイルから読み込んだフォント名をそのまま返却します。
(インデント)try:：Tkinterのフォント列挙時に発生する可能性のある例外を捕捉するブロックを開始します。
(インデント)(インデント)import tkinter.font as tkfont：システムで利用可能なフォント一覧を取得するため、Tkinterのフォントモジュールをインポートします。
(インデント)(インデント)available = set(tkfont.families())：利用可能なすべてのフォントファミリ名をセットとして取得します。
(インデント)(インデント)for candidate in ("Yu Gothic UI", "Meiryo UI", "Yu Gothic", "Meiryo"):：優先度の高い日本語UIフォントの候補を順番にチェックするループを開始します。
(インデント)(インデント)(インデント)if candidate in available:：候補のフォントがシステムにインストールされているかを判定します。
(インデント)(インデント)(インデント)(インデント)_UI_FONT_FAMILY = candidate：見つかった候補のフォント名をUIフォントとして採用します。
(インデント)(インデント)(インデント)(インデント)break：フォントが見つかったためループを終了します。
(インデント)(インデント)else:：候補のフォントが一切見つからなかった場合の処理ブロックを開始します。
(インデント)(インデント)(インデント)_UI_FONT_FAMILY = ""：フォントが見つからない場合は空文字列を設定します。
(インデント)except Exception as exc:：フォント列挙中に予期せぬエラーが発生した場合の例外を捕捉します。
(インデント)(インデント)log_error("UIフォント検出", f"{exc}\n{traceback.format_exc()}")：フォント検出エラーの内容と詳細なスタックトレースをログに記録します。
(インデント)(インデント)_UI_FONT_FAMILY = ""：エラー発生時は空文字列を代入して処理を継続します。
(インデント)return _UI_FONT_FAMILY：決定したUIフォントのファミリー名を返却します。
def load_settings_minimal():：起動直後の表示に最低限必要な設定のみを読み込む関数を定義します。
(インデント)global _SETTINGS_RAW_CACHE：生データのキャッシュ用グローバル変数を参照することを宣言します。
(インデント)try:：設定ファイルの読み込みおよびJSON解析時のエラーを捕捉するブロックを開始します。
(インデント)(インデント)with open(SETTINGS_PATH, "r", encoding="utf-8") as f:：設定ファイルを読み込み専用のUTF-8エンコーディングで開きます。
(インデント)(インデント)(インデント)data = json.load(f)：ファイルからJSONデータを読み込んでPythonオブジェクトに変換します。
(インデント)(インデント)if not isinstance(data, dict):：読み込んだデータが辞書型であるかを検証します。
(インデント)(インデント)(インデント)data = {}：辞書型でない場合は安全のため空の辞書に置き換えます。
(インデント)(インデント)else:：正常に辞書型として読み込めた場合の処理を開始します。
(インデント)(インデント)(インデント)_SETTINGS_RAW_CACHE = data：本読み込みで再利用するため、読み込んだ生データをキャッシュに保存します。
(インデント)except (OSError, ValueError):：ファイル不在やJSON破損などのエラーを捕捉します。
(インデント)(インデント)data = {}：エラー発生時は空の辞書を代入して続行します。
(インデント)theme = data.get('theme')：設定データからテーマ設定の値を取得します。
(インデント)if theme not in ('light', 'dark'):：取得したテーマが許容される値であるかを検証します。
(インデント)(インデント)theme = DEFAULT_SETTINGS['theme']：不正な値の場合はデフォルトのテーマに置き換えます。
(インデント)geometry = data.get('window_geometry')：設定データからウィンドウジオメトリを取得します。
(インデント)if not (isinstance(geometry, str) and _WINDOW_GEOMETRY_RE.match(geometry)):：ジオメトリの文字列形式が正しい正規表現に一致するかを検証します。
(インデント)(インデント)geometry = DEFAULT_SETTINGS['window_geometry']：不正な形式の場合はデフォルトのジオメトリに置き換えます。
(インデント)font_cache = data.get('ui_font_family_cache')：設定データからUIフォント名のキャッシュを取得します。
(インデント)if not isinstance(font_cache, str):：取得したフォントキャッシュが文字列型であるかを検証します。
(インデント)(インデント)font_cache = ''：文字列型でない場合は空文字列に置き換えます。
(インデント)font_detected = bool(data.get('ui_font_detected', False))：フォントが検出済みかどうかのフラグをブール値として取得します。
(インデント)return {：最小限の抽出設定をまとめた辞書の返却を開始します。
(インデント)(インデント)'theme': theme,：テーマ設定を辞書に含めます。
(インデント)(インデント)'window_geometry': geometry,：ウィンドウジオメトリを辞書に含めます。
(インデント)(インデント)'ui_font_family_cache': font_cache,：フォントキャッシュを辞書に含めます。
(インデント)(インデント)'ui_font_detected': font_detected,：フォント検出済みフラグを辞書に含め、返却します。
(インデント)}：最小限設定の辞書定義を終了します。

#### 第6章：設定値の移行処理、検証、およびJSON読み込みヘルパー

_SETTINGS_RAW_CACHE = None：設定ファイルの生データを一時的に保持するためのキャッシュ用グローバル変数を初期化します。
_SETTINGS_ENUM_FIELDS = {：特定のキーに対して許容される選択肢の定義を開始します。
(インデント)'theme': {'light', 'dark'},：themeキーで許可されるテーマ名のセットを定義します。
(インデント)'match_mode': {'partial', 'exact', 'regex', 'fuzzy'},：match_modeキーで許可される一致方式のセットを定義します。
(インデント)'notify_sound_type': {'builtin', 'file'},：notify_sound_typeキーで許可される通知音タイプのセットを定義します。
(インデント)'result_sort_column': {'name', 'dir', 'size', 'mtime'},：result_sort_columnキーで許可されるソート列のセットを定義します。
}：許可される選択肢の定義辞書を終了します。
_WINDOW_GEOMETRY_RE = re.compile(r'^\d+x\d+([+-]\d+[+-]\d+)?$')：ウィンドウのジオメトリ文字列の書式を検証するための正規表現パターンをコンパイルして保持します。
def _migrate_settings(data):：旧バージョンの設定データを新しい仕様に合わせて移行する関数を定義します。
(インデント)migrated = dict(data)：元データを上書きしないよう、辞書のコピーを作成します。
(インデント)if 'match_mode' not in migrated and migrated.get('exact_match'):：match_modeキーがなく、かつ旧設定のexact_matchが有効になっていたかを判定します。
(インデント)(インデント)migrated['match_mode'] = 'exact'：旧設定の完全一致フラグから新しいmatch_modeへ値を移行します。
(インデント)return migrated：移行処理が完了した設定データの辞書を返却します。
def _validate_settings(merged):：マージされた設定値の中に不正な型や値が含まれていないかを検証し、必要に応じて既定値に修正する関数を定義します。
(インデント)validated = dict(merged)：検証作業用の辞書コピーを作成します。
(インデント)invalid_keys = []：不正値が検出されたキーを記録するためのリストを初期化します。
(インデント)for key, default_value in DEFAULT_SETTINGS.items():：デフォルト設定のすべてのキーと標準値を順番に走査するループを開始します。
(インデント)(インデント)value = validated.get(key, default_value)：検証対象の設定値を取得します。
(インデント)(インデント)ok = True：値が正常であるかを示すフラグを初期状態（True）に設定します。
(インデント)(インデント)if isinstance(default_value, bool):：デフォルト値がブール型であるかを判定します。
(インデント)(インデント)(インデント)ok = isinstance(value, bool)：現在の値もブール型であるかを検証します。
(インデント)(インデント)elif isinstance(default_value, list):：デフォルト値がリスト型であるかを判定します。
(インデント)(インデント)(インデント)ok = isinstance(value, list)：現在の値もリスト型であるかを検証します。
(インデント)(インデント)elif isinstance(default_value, dict):：デフォルト値が辞書型であるかを判定します。
(インデント)(インデント)(インデント)ok = isinstance(value, dict)：現在の値も辞書型であるかを検証します。
(インデント)(インデント)elif key == 'result_sort_column':：検証中のキーが結果ソート列であるかを判定します。
(インデント)(インデント)(インデント)ok = value is None or (isinstance(value, str) and value in _SETTINGS_ENUM_FIELDS[key])：値がNone、または許可されたソート列の文字列であるかを検証します。
(インデント)(インデント)elif key in _SETTINGS_ENUM_FIELDS:：検証中のキーが許可された選択肢の制限対象であるかを判定します。
(インデント)(インデント)(インデント)ok = isinstance(value, str) and value in _SETTINGS_ENUM_FIELDS[key]：値が文字列であり、かつ許容される選択肢に含まれているかを検証します。
(インデント)(インデント)elif key == 'window_geometry':：検証中のキーがウィンドウジオメトリであるかを判定します。
(インデント)(インデント)(インデント)ok = isinstance(value, str) and bool(_WINDOW_GEOMETRY_RE.match(value))：値が正しいジオメトリ形式の文字列であるかを正規表現で検証します。
(インデント)(インデント)elif isinstance(default_value, str):：デフォルト値が文字列型であるかを判定します。
(インデント)(インデント)(インデント)ok = isinstance(value, str)：現在の値も文字列型であるかを検証します。
(インデント)(インデント)if not ok:：値が不正であったかを判定します。
(インデント)(インデント)(インデント)invalid_keys.append(key)：不正と判定されたキー名を記録用リストに追加します。
(インデント)(インデント)(インデント)validated[key] = default_value：不正な値をデフォルト値に差し戻します。
(インデント)if isinstance(validated.get('result_columns_visible'), list):：表示列設定のリストの中に不正な要素が混ざっていないかを確認します。
(インデント)(インデント)filtered = [c for c in validated['result_columns_visible'] if c in RESULT_COLUMN_TOGGLABLE]：切り替え可能な列名リストに含まれるものだけを抽出し直します。
(インデント)(インデント)if filtered != validated['result_columns_visible']:：フィルタリング前後で内容に差分があるかを判定します。
(インデント)(インデント)(インデント)invalid_keys.append('result_columns_visible(要素)')：不正な要素が含まれていたことを記録リストに追加します。
(インデント)(インデント)validated['result_columns_visible'] = filtered：フィルタリング済みの正常なリストに置き換えます。
(インデント)if isinstance(validated.get('custom_mode_checked_features'), list):：カスタムモードのチェック済み機能リストの中に不正な要素が混ざっていないかを確認します。
(インデント)(インデント)filtered_features = [c for c in validated['custom_mode_checked_features'] if c in CUSTOM_FEATURE_KEYS]：既定の機能キーに含まれるものだけを抽出し直します。
(インデント)(インデント)if filtered_features != validated['custom_mode_checked_features']:：フィルタリング前後で差分があるかを判定します。
(インデント)(インデント)(インデント)invalid_keys.append('custom_mode_checked_features(要素)')：不正な要素が含まれていたことを記録リストに追加します。
(インデント)(インデント)validated['custom_mode_checked_features'] = filtered_features：フィルタリング済みの正常なリストに置き換えます。
(インデント)if invalid_keys:：不正な値や要素が1つ以上検出されていたかを判定します。
(インデント)(インデント)log_error("設定検証", f"不正な設定値を既定値に差し戻しました: {', '.join(invalid_keys)}")：不正値が修正された旨をエラーログに記録します。
(インデント)return validated：検証および修正が完了した設定データの辞書を返却します。
def _read_json_dict_file(path):：指定されたパスのJSONファイルを読み込み、辞書型であることを確認して返すヘルパー関数を定義します。
(インデント)with open(path, "r", encoding="utf-8") as f:：対象のJSONファイルを読み込み専用のUTF-8エンコーディングで開きます。
(インデント)(インデント)data = json.load(f)：ファイルからJSONデータを読み込んでPythonオブジェクトに変換します。
(インデント)if not isinstance(data, dict):：読み込まれたトップレベルのデータが辞書型であるかを検証します。
(インデント)(インデント)raise ValueError(f"JSONのトップレベルがオブジェクトではありません: {type(data)}")：辞書型でなかった場合は値エラーを発生させます。
(インデント)return data：正常に読み込まれた辞書データを返却します。

#### 第7章：本格的な設定読み込み、保存、およびログ出力処理

def load_settings():：設定ファイルを読み込み、バックアップからの復元や検証・移行を行って返す関数を定義します。
(インデント)global _SETTINGS_RAW_CACHE：生データキャッシュ用のグローバル変数を参照することを宣言します。
(インデント)data = None：読み込む設定データを初期化します。
(インデント)primary_error = None：初回の読み込み時に発生したエラーを格納する変数を初期化します。
(インデント)if _SETTINGS_RAW_CACHE is not None:：最小限読み込みでキャッシュされた生データが存在するかを判定します。
(インデント)(インデント)data = _SETTINGS_RAW_CACHE：キャッシュが存在する場合はそれを採用します。
(インデント)(インデント)_SETTINGS_RAW_CACHE = None：一度使用したキャッシュはクリアします。
(インデント)elif os.path.exists(SETTINGS_PATH):：本体の設定ファイルが存在するかを判定します。
(インデント)(インデント)try:：設定ファイルの読み込み時のエラーを捕捉するブロックを開始します。
(インデント)(インデント)(インデント)data = read_json_dict_file(SETTINGS_PATH)：ヘルパー関数を使って設定ファイルを読み込みます。
(インデント)(インデント)except (OSError, ValueError) as exc:：読み込みやJSON解析時のエラーを捕捉します。
(インデント)(インデント)(インデント)primary_error = exc：発生したエラーを記録します。
(インデント)(インデント)(インデント)log_error("設定読込", f"{exc}\n{traceback.format_exc()}")：エラーの内容とトレースバックをログに記録します。
(インデント)if data is None and primary_error is not None:：本体の読み込みに失敗した場合のバックアップ復元処理ブロックを開始します。
(インデント)(インデント)backup_path = SETTINGS_PATH + ".bak"：バックアップファイルのパスを生成します。
(インデント)(インデント)if os.path.exists(backup_path):：バックアップファイルが存在するかを判定します。
(インデント)(インデント)(インデント)try:：バックアップからの読み込み時のエラーを捕捉するブロックを開始します。
(インデント)(インデント)(インデント)(インデント)data = read_json_dict_file(backup_path)：バックアップファイルからデータを読み込みます。
(インデント)(インデント)(インデント)(インデント)log_error("設定読込", "settings.jsonの破損を検出したため、バックアップ(.bak)から復元しました")：復元が行われたことをログに記録します。
(インデント)(インデント)(インデント)except (OSError, ValueError) as backup_exc:：バックアップ読込時のエラーを捕捉します。
(インデント)(インデント)(インデント)(インデント)log_error("設定読込(バックアップ)", f"{backup_exc}\n{traceback.format_exc()}")：バックアップ読込エラーをログに記録します。
(インデント)(インデント)if data is None:：バックアップからも復元できなかった場合の隔離処理ブロックを開始します。
(インデント)(インデント)(インデント)try:：壊れたファイルの退避処理時のエラーを捕捉するブロックを開始します。
(インデント)(インデント)(インデント)(インデント)timestamp = datetime.now().strftime("%Y%m%d%H%M%S")：退避ファイル名用のタイムスタンプ文字列を生成します。
(インデント)(インデント)(インデント)(インデント)quarantine_path = f"{SETTINGS_PATH}.corrupt{timestamp}"：隔離先のファイルパスを生成します。
(インデント)(インデント)(インデント)(インデント)os.replace(SETTINGS_PATH, quarantine_path)：破損した本体ファイルを別名ファイルへ移動・退避させます。
(インデント)(インデント)(インデント)(インデント)log_error("設定読込", f"復旧できない破損設定ファイルを退避しました: {quarantine_path}")：退避完了をログに記録します。
(インデント)(インデント)(インデント)except OSError as quarantine_exc:：退避処理中のエラーを捕捉します。
(インデント)(インデント)(インデント)(インデント)log_error("設定読込(退避)", str(quarantine_exc))：退避エラーをログに記録します。
(インデント)if data is None:：すべての復元試行が失敗した場合の処理を開始します。
(インデント)(インデント)data = {}：空の辞書を代入して初期化します。
(インデント)data = _migrate_settings(data)：読み込んだデータに対して旧バージョンの移行処理を実行します。
(インデント)merged = dict(DEFAULT_SETTINGS)：デフォルト設定のコピーを作成します。
(インデント)merged.update(data)：デフォルト設定に読み込んだデータを上書きマージします。
(インデント)return _validate_settings(merged)：マージされた設定を検証・修正して返却します。
def save_settings(settings):：設定データをファイルに保存するための関数を定義します。
(インデント)try:：設定保存時のエラーを捕捉するブロックを開始します。
(インデント)(インデント)os.makedirs(SETTINGS_DIR, exist_ok=True)：設定保存用ディレクトリが存在しない場合は作成します。
(インデント)(インデント)_atomic_write_json(SETTINGS_PATH, settings, keep_backup=True, restrict_permissions=True)：バックアップ作成と権限制限を伴うアトミック書き込みで設定を保存します。
(インデント)except OSError as exc:：保存時のOSエラーを捕捉します。
(インデント)(インデント)log_error("設定保存", f"{exc}\n{traceback.format_exc()}")：保存エラーの内容とトレースバックをログに記録します。
def _mask_sensitive_log_text(text):：ログに書き込むテキストからユーザー名などの機密情報をマスクする関数を定義します。
(インデント)if not text:：テキストが空であるかを判定します。
(インデント)(インデント)return text：空の場合はそのままテキストを返却します。
(インデント)try:：ホームディレクトリの取得時のエラーを捕捉するブロックを開始します。
(インデント)(インデント)home = os.path.expanduser("~")：ユーザーのホームディレクトリのパスを取得します。
(インデント)except Exception:：取得失敗時の例外を捕捉します。
(インデント)(インデント)return text：失敗した場合はそのままテキストを返却します。
(インデント)if not home or home == "~":：取得したホームディレクトリが無効であるかを判定します。
(インデント)(インデント)return text：無効な場合はそのままテキストを返却します。
(インデント)masked = text：マスク処理用の変数に元のテキストを代入します。
(インデント)for candidate in {home, home.replace("\", "/"), home.replace("/", "\")}:：区切り文字の違いを考慮したホームディレクトリの表記候補をループで処理します。
(インデント)(インデント)if not candidate:：候補が空であるかを判定します。
(インデント)(インデント)(インデント)continue：空の場合はスキップします。
(インデント)(インデント)if os.name == "nt":：実行環境がWindowsであるかを判定します。
(インデント)(インデント)(インデント)pattern = re.escape(candidate)：Windows用にパスの正規表現パターンをエスケープして作成します。
(インデント)(インデント)(インデント)masked = re.sub(pattern, "~", masked, flags=re.IGNORECASE)：大文字小文字を区別せずにホームディレクトリ部分を ~ に置換します。
(インデント)(インデント)else:：Windows以外の環境の場合の処理を開始します。
(インデント)(インデント)(インデント)masked = masked.replace(candidate, "~")：ホームディレクトリ部分を ~ に置換します。
(インデント)return masked：機密情報がマスクされたテキストを返却します。
def log_error(category, message):：アプリケーションのエラーログをファイルに記録する関数を定義します。
(インデント)try:：ログ出力時のエラーを捕捉するブロックを開始します。
(インデント)(インデント)os.makedirs(APP_DATA_DIR, exist_ok=True)：アプリデータディレクトリが存在しない場合は作成します。
(インデント)(インデント)timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")：現在日時のタイムスタンプ文字列を生成します。
(インデント)(インデント)safe_message = _mask_sensitive_log_text(str(message))：エラーメッセージに対して機密情報のマスク処理を適用します。
(インデント)(インデント)with open(APP_LOG_PATH, "a", encoding="utf-8") as f:：ログファイルを追記モードのUTF-8エンコーディングで開きます。
(インデント)(インデント)(インデント)f.write(f"[{timestamp}] [{category}] {safe_message}\n")：タイムスタンプ、カテゴリ、マスク済みメッセージをログファイルに書き込みます。
(インデント)except OSError:：ログ書き込み中のOSエラーを捕捉します。
(インデント)(インデント)return：エラー発生時は処理を中断します。

#### 第8章：クラッシュダイアログの表示と未処理例外ハンドラの仕組み

_CRASH_DIALOG_STATE = {'showing': False}：クラッシュダイアログが重複して表示されるのを防ぐための状態管理用グローバル変数を初期化します。
def _show_crash_dialog(title, message):：予期しない例外発生時にメインウィンドウを隠し、独立のエラーウィンドウを表示する関数を定義します。
(インデント)if _CRASH_DIALOG_STATE['showing']:：すでに別のエラーウィンドウが表示中であるかを判定します。
(インデント)(インデント)return：表示中の場合は多重起動を防ぐため処理を中断します。
(インデント)_CRASH_DIALOG_STATE['showing'] = True：ダイアログ表示中フラグを有効（True）に設定します。
(インデント)try:：ダイアログの構築および表示に関連するエラーを捕捉するブロックを開始します。
(インデント)(インデント)root = tk._default_root：Tkinterのデフォルトのルートウィンドウを取得します。
(インデント)(インデント)owns_root = False：独自にルートウィンドウを作成したかどうかのフラグを初期化します。
(インデント)(インデント)if root is None:：ルートウィンドウが存在しないかを判定します。
(インデント)(インデント)(インデント)root = tk.Tk()：新しくTkのルートインスタンスを作成します。
(インデント)(インデント)(インデント)root.withdraw()：作成したルートウィンドウを一時的に非表示にします。
(インデント)(インデント)(インデント)owns_root = True：独自にルートを作成したフラグを有効にします。
(インデント)(インデント)else:：すでにルートウィンドウが存在する場合の処理を開始します。
(インデント)(インデント)(インデント)try:：メインウィンドウ非表示化時のエラーを捕捉するブロックを開始します。
(インデント)(インデント)(インデント)(インデント)root.withdraw()：メインウィンドウを非表示にします。
(インデント)(インデント)(インデント)except tk.TclError:：Tclエラーを捕捉します。
(インデント)(インデント)(インデント)(インデント)pass：エラーが発生しても処理を続行します。
(インデント)(インデント)(インデント)for child in list(root.winfo_children()):：ルートウィンドウに紐づくすべてのサブウィンドウを走査するループを開始します。
(インデント)(インデント)(インデント)(インデント)if isinstance(child, tk.Toplevel):：子要素が最上位の補助ウィンドウであるかを判定します。
(インデント)(インデント)(インデント)(インデント)(インデント)try:：補助ウィンドウ非表示化時のエラーを捕捉するブロックを開始します。
(インデント)(インデント)(インデント)(インデント)(インデント)(インデント)child.withdraw()：該当する補助ウィンドウも強制的に非表示にします。
(インデント)(インデント)(インデント)(インデント)(インデント)except tk.TclError:：Tclエラーを捕捉します。
(インデント)(インデント)(インデント)(インデント)(インデント)pass：エラーが発生しても処理を継続します。
(インデント)(インデント)win = tk.Toplevel(root)：エラー表示用の新しい最上位ウィンドウを作成します。
(インデント)(インデント)win.title(title)：エラーウィンドウのタイトルを設定します。
(インデント)(インデント)win.configure(bg="#b71c1c")：エラーの深刻さを表す赤系の背景色をウィンドウに適用します。
(インデント)(インデント)win.resizable(False, False)：エラーウィンドウのサイズ変更を不可に固定します。
(インデント)(インデント)try:：最前面固定属性の設定時のエラーを捕捉するブロックを開始します。
(インデント)(インデント)(インデント)win.attributes("-topmost", True)：エラーウィンドウを常に最前面に表示するよう設定します。
(インデント)(インデント)except tk.TclError:：対応していない環境でのTclエラーを捕捉します。
(インデント)(インデント)(インデント)pass：エラーが発生しても処理を継続します。
(インデント)(インデント)frame = tk.Frame(win, bg="#b71c1c", padx=20, pady=16)：余白を持たせたコンテンツ配置用のフレームを作成します。
(インデント)(インデント)frame.pack(fill="both", expand=True)：フレームをウィンドウいっぱいに配置します。
(インデント)(インデント)tk.Label(frame, text="⚠ 予期しないエラーが発生しました", bg="#b71c1c", fg="white", font=("", 12, "bold")).pack(anchor="w")：警告メッセージの見出しラベルを配置します。
(インデント)(インデント)tk.Label(frame, text="アプリの動作が不安定な可能性があります。\n内容はログファイルに記録されました。", bg="#b71c1c", fg="white", justify="left").pack(anchor="w", pady=(8, 8))：注意喚起とログ記録を知らせる説明ラベルを配置します。
(インデント)(インデント)text_box = tk.Text(frame, height=6, width=54, bg="#ffebee", fg="#b71c1c", wrap="word")：エラー詳細を表示するためのテキストボックスウィジェットを作成します。
(インデント)(インデント)text_box.insert("1.0", message)：テキストボックスにエラーメッセージを挿入します。
(インデント)(インデント)text_box.configure(state="disabled")：テキストボックスを読み取り専用に設定します。
(インデント)(インデント)text_box.pack(fill="both", expand=True, pady=(0, 12))：テキストボックスを配置します。
(インデント)(インデント)btn_row = tk.Frame(frame, bg="#b71c1c")：操作ボタンを配置するための横並びフレームを作成します。
(インデント)(インデント)btn_row.pack(fill="x")：ボタン用フレームを配置します。
(インデント)(インデント)def _close_app():：アプリを終了させるための内部関数を定義します。
(インデント)(インデント)(インデント)_CRASH_DIALOG_STATE['showing'] = False：ダイアログ表示中フラグを解除します。
(インデント)(インデント)(インデント)try:：ウィンドウ破棄時のエラーを捕捉するブロックを開始します。
(インデント)(インデント)(インデント)(インデント)win.destroy()：エラーウィンドウを破棄します。
(インデント)(インデント)(インデント)except tk.TclError:：Tclエラーを捕捉します。
(インデント)(インデント)(インデント)(インデント)pass：エラーが発生しても処理を継続します。
(インデント)(インデント)(インデント)os._exit(1)：アプリケーションプロセスを終了します。
(インデント)(インデント)def _copy_details():：エラー詳細をクリップボードにコピーするための内部関数を定義します。
(インデント)(インデント)(インデント)try:：クリップボード操作時のエラーを捕捉するブロックを開始します。
(インデント)(インデント)(インデント)(インデント)win.clipboard_clear()：クリップボードの既存内容をクリアします。
(インデント)(インデント)(インデント)(インデント)win.clipboard_append(message)：エラーメッセージ全体をクリップボードに追加します。
(インデント)(インデント)(インデント)except tk.TclError:：Tclエラーを捕捉します。
(インデント)(インデント)(インデント)(インデント)pass：エラーが発生しても処理を続行します。
(インデント)(インデント)tk.Button(btn_row, text="詳細をコピー", command=_copy_details, bg="#7a1010", fg="white", activebackground="#5a0b0b", activeforeground="white", relief="flat", padx=10, pady=4).pack(side="left")：「詳細をコピー」ボタンを配置します。
(インデント)(インデント)tk.Button(btn_row, text="アプリを終了する", command=_close_app, bg="#d32f2f", fg="white", activebackground="#8e0000", activeforeground="white", relief="flat", padx=10, pady=4).pack(side="right")：「アプリを終了する」ボタンを配置します。
(インデント)(インデント)win.update_idletasks()：ウィンドウの描画レイアウトを強制的に更新します。
(インデント)(インデント)w, h = 480, win.winfo_reqheight()：ウィンドウの幅と高さを取得します。
(インデント)(インデント)sw, sh = win.winfo_screenwidth(), win.winfo_screenheight()：ディスプレイ全体の幅と高さを取得します。
(インデント)(インデント)win.geometry(f"{w}x{h}+{max(0, (sw - w) // 2)}+{max(0, (sh - h) // 2)}")：ウィンドウを画面中央に配置するようジオメトリを設定します。
(インデント)(インデント)win.protocol("WM_DELETE_WINDOW", _close_app)：ウィンドウの閉じるボタンが押されたときの動作に終了関数を割り当てます。
(インデント)(インデント)win.grab_set()：モーダルダイアログとして設定します。
(インデント)(インデント)if owns_root:：独自にルートを作成していたかを判定します。
(インデント)(インデント)(インデント)root.mainloop()：イベントループを開始します。
(インデント)except Exception:：ダイアログ表示処理自体で致命的なエラーが発生した場合の例外を捕捉します。
(インデント)(インデント)_CRASH_DIALOG_STATE['showing'] = False：フラグをリセットします。
(インデント)(インデント)try:：標準エラー出力へのフォールバック書き込み時のエラーを捕捉するブロックを開始します。
(インデント)(インデント)(インデント)print(message, file=sys.stderr)：標準エラー出力にエラーメッセージを出力します。
(インデント)(インデント)except Exception:：出力時エラーを捕捉します。
(インデント)(インデント)(インデント)pass：例外が発生しても処理を継続します。
def _handle_uncaught_exception(exc_type, exc_value, exc_tb):：未処理例外を捕捉する関数を定義します。
(インデント)if issubclass(exc_type, KeyboardInterrupt):：発生した例外がキーボード割り込みであるかを判定します。
(インデント)(インデント)sys.excepthook(exc_type, exc_value, exc_tb)：キーボード割り込みの場合は標準の処理に委譲します。
(インデント)(インデント)return：処理を終了します。
(インデント)message = "".join(traceback.format_exception(exc_type, exc_value, exc_tb))：例外のトレースバック情報を文字列として結合・生成します。
(インデント)log_error("未処理例外", message)：未処理例外の内容をエラーログに記録します。
(インデント)_show_crash_dialog("File Search Explorer - エラー", message[-2000:])：クラッシュダイアログを呼び出します。

#### 第9章：スレッド例外ハンドラとコードの総量について

※ ここでは、バックグラウンドスレッドで発生した例外をroot.after() を使い、メインスレッドでエラーダイアログを表示するよう実行を予約する仕組みです。 （_handle_thread_exception）の解説とあわせて、ここまで解説してきた初期化・設定管理・例外処理等のコード全体の役割を整理します。
def _handle_thread_exception(args):：バックグラウンドスレッド内で発生した未処理例外を捕捉する関数を定義します。
(インデント)message = "".join(traceback.format_exception(args.exc_type, args.exc_value, args.exc_traceback))：スレッド例外のトレースバック情報を生成します。
(インデント)log_error("未処理例外(スレッド)", message)：スレッド側での未処理例外の内容をエラーログに記録します。
(インデント)root = tk._default_root：ルートウィンドウの存在を確認します。
(インデント)if root is not None:：ルートウィンドウが存在しているかを判定します。
(インデント)(インデント)try:：メインスレッド側へのエラーダイアログ表示の予約時のエラーを捕捉するブロックを開始します。
(インデント)(インデント)(インデント)root.after(0, _show_crash_dialog, "File Search Explorer - エラー", message[-2000:])：メインスレッドのコンテキスト上で安全にエラーダイアログを表示するようイベントループに登録します。
(インデント)(インデント)except Exception:：予約登録時の例外を捕捉します。
(インデント)(インデント)(インデント)pass：例外が発生しても処理を継続します。

#### 第10章：アプリケーションのメインクラスと初期化処理の定義

class FileSearchExplorerApp:：ファイル検索エクスプローラーのメインアプリケーションクラスの定義を開始します。
(インデント)def init(self, root):：アプリケーションのインスタンス初期化メソッドを定義します。
(インデント)(インデント)self.root = root：受け取ったルートウィンドウオブジェクトをインスタンス変数に保存します。
(インデント)(インデント)self.root.title("File Search Explorer")：メインウィンドウのタイトルを設定します。
(インデント)(インデント)self.app_mode = DEFAULT_APP_MODE：起動モードをデフォルトのモードに初期化します。
(インデント)(インデント)self.settings = load_settings()：設定ファイルから各種設定値を読み込みます。
(インデント)(インデント)self.mode_features = MODE_FEATURES.get(self.app_mode, MODE_FEATURES['max'])：現在のモードに応じた機能有効・無効フラグのマッピングを取得します。
(インデント)(インデント)self._custom_checked_vars = {}：「難しいモード」で個別にチェックされた変数を管理する辞書を初期化します。
(インデント)(インデント)self._initialize_ui_theme_and_fonts()：UIのテーマカラーやフォントに関する初期設定を行うメソッドを呼び出します。
(インデント)(インデント)self._build_main_menu()：トップメニューバーを構築するメソッドを呼び出します。
(インデント)(インデント)self._build_main_layout()：メイン画面のレイアウトを構築するメソッドを呼び出します。
(インデント)(インデント)self._bind_shortcuts()：キーボードショートカットのバインドを行うメソッドを呼び出します。
(インデント)(インデント)self._restore_window_geometry()：保存されていたウィンドウのサイズや位置を復元するメソッドを呼び出します。
(インデント)(インデント)self.root.protocol("WM_DELETE_WINDOW", self.on_close)：ウィンドウの閉じるボタンが押されたときの終了イベントに関数を割り当てます。

#### 第11章：テーマカラー・フォントの適用とメインメニューの構築

def _initialize_ui_theme_and_fonts(self):：UIのテーマカラーやフォント環境を初期化・設定するメソッドを定義します。
(インデント)theme_name = self.settings.get('theme', 'light')：設定から現在のテーマ名を取得します。
(インデント)self.colors = THEME_COLORS.get(theme_name, THEME_COLORS['light'])：テーマ名に対応するカラーパレットの辞書を取得します。
(インデント)font_family = get_ui_font_family(cached_value=self.settings.get('ui_font_family_cache', ''), cached_detected=self.settings.get('ui_font_detected', False))：UIフォントファミリー名を取得します。
(インデント)if font_family:：取得したフォントファミリー名が空でないかを判定します。
(インデント)(インデント)self.default_font = (font_family, 10)：デフォルトフォントを定義します。
(インデント)(インデント)self.bold_font = (font_family, 10, "bold")：太字用のフォントを定義します。
(インデント)(インデント)self.header_font = (font_family, 11, "bold")：見出し用のフォントを定義します。
(インデント)else:：フォントが見つからなかった場合の処理ブロックを開始します。
(インデント)(インデント)self.default_font = ("", 10)：Tkinterの既定フォントにフォールバックさせます。
(インデント)(インデント)self.bold_font = ("", 10, "bold")：既定フォントの太字を定義します。
(インデント)(インデント)self.header_font = ("", 11, "bold")：既定フォントの見出しを定義します。
(インデント)style = ttk.Style()：ttkのスタイル管理オブジェクトを作成します。
(インデント)try:：ttkテーマの設定時に発生する可能性のある例外を捕捉するブロックを開始します。
(インデント)(インデント)style.theme_use('clam')：モダンな「clam」スタイルを適用します。
(インデント)except Exception:：テーマ適用時のエラーを捕捉します。
(インデント)(インデント)pass：エラーが発生しても処理を継続します。
(インデント)style.configure('.', font=self.default_font, background=self.colors['bg'], foreground=self.colors['fg'])：アプリケーション全体の基本スタイルを設定します。
def _build_main_menu(self):：トップメニューバーを構築するメソッドを定義します。
(インデント)menubar = tk.Menu(self.root)：メニューバーウィジェットを作成します。
(インデント)file_menu = tk.Menu(menubar, tearoff=0)：ファイルメニュー用のプルダウンメニューを作成します。
(インデント)file_menu.add_command(label="終了", command=self.on_close)：ファイルメニューに「終了」項目を追加します。
(インデント)menubar.add_cascade(label="ファイル", menu=file_menu)：メニューバーに「ファイル」カスケードメニューを追加します。
(インデント)self.root.config(menu=menubar)：メニューバーをメインウィンドウに設定します。
