## はじめに

こんにちはUnknown777です。

今回は、FileSearchExplorerのコードを1行ずつ解説します。

ブログの投稿頻度が落ちてしまってすみませんでした。

また、前回の1000行解説ブログが（2026/10/29 20:30現在）1115PVを記録しました。読んでくださった皆さん、本当にありがとうございます。今回のブログも頑張って作りました。

また、今回のブログの「本編」はAIで作っています。

できるだけ自然な文章になるようにUnknown777が書き直しているので、読みやすくなっていると思います。

では、本編に入る前にこのブログの書く方法を記しておきます。

#### ブログ記述方法

AIに1行ずつ詳しく説明してもらう(AIのほうが詳しい)(ググりながらコード書いてるレベルなので)

それを文章作成アプリで移すついでに文章を校正、ググって訂正をする

それをブログサイトなどに張り付けて公開するという手順です。

## 本編

### **第1章：例外フック、初期設定**

- `sys.excepthook = _handle_uncaught_exception`：プロセス全体でキャッチされなかった例外を自作ハンドラへ流す設定。メインスレッドで予期しない例外が発生したとき、ログやダイアログで原因を残せるようにするため。
- `if hasattr(threading, "excepthook")`：Python 3.8以降で存在する`threading.excepthook`があるか確認。古いバージョンでも落ちないようにする互換性チェック。
- `threading.excepthook = _handle_thread_exception`：サブスレッドの未処理例外も同じ仕組みで拾う。GUIアプリではバックグラウンドスレッドの例外を記録できるようにする。

### **第2章：通知やユーティリティなどの小さな関数**

- `def play_notification_sound(settings)`：通知音再生を担当する関数定義。設定辞書を受け取り、ユーザーの設定に合わせる。
- `if not settings.get('notify_sound_enabled', True)`：通知音が無効なら即return。`get`にデフォルト値`True`を指定することで、キーがない場合は有効として扱う。
- `return`：早期リターンでネストを浅く保ち、不要な処理を実行しない。
- `sound_type = settings.get('notify_sound_type', 'builtin')`：ファイル再生か組み込み音かを判断する設定値を取得。デフォルトは組み込み音。
- `try`：音再生はOSや設定に依存して失敗する可能性があるため、例外処理で囲む。
- `if sound_type == 'file'`：ユーザー指定ファイルを再生する分岐。
- `path = settings.get('notify_sound_file', '')`：ファイルパスを取得。設定がない場合は空文字にする。
- `if not path or not os.path.isfile(path)`：パスが空、またはファイルが存在しない場合は再生しない。
- `return`：再生できない場合は早期リターンする。
- `if WINSOUND_AVAILABLE and path.lower().endswith('.wav')`：Windowsの`winsound`が利用でき、パスの末尾が`.wav`の場合に再生する。
- `winsound.PlaySound(path, winsound.SND_FILENAME | winsound.SND_ASYNC)`：ファイル名を指定して非同期再生する。`SND_ASYNC`により、再生完了まで呼び出し元を待たせない。
- `else`：組み込み音の分岐。
- `if WINSOUND_AVAILABLE`：`winsound`が利用できるか確認する。
- `alias = settings.get('notify_sound_builtin', 'SystemAsterisk')`：OSに登録されたサウンドエイリアスを取得。デフォルトは`SystemAsterisk`。
- `winsound.PlaySound(alias, winsound.SND_ALIAS | winsound.SND_ASYNC)`：サウンドエイリアスを指定して非同期再生する。
- `except Exception as exc`：再生中に発生した例外を捕捉する。
- `log_error("通知音", str(exc))`：失敗内容をログに記録する。通知音の失敗でアプリ全体が停止することを避ける。
- `def show_desktop_notification(title, message)`：OSのデスクトップ通知を出す関数。
- `plyer_notification = _lazy_import_plyer()`：`plyer`を必要なタイミングで読み込む。不要な場面でのインポートを避ける。
- `if plyer_notification is None`：ライブラリが利用できない環境では何もしない。
- `return`：通知処理をスキップする。
- `try`：通知処理の例外に備える。
- `plyer_notification.notify(title=title, message=message,`：`plyer`の通知関数を呼び出し、タイトルと本文を渡す。
- `app_name="File Search Explorer", timeout=5)`：アプリ名と通知の表示時間を指定する。`timeout=5`は表示時間の目安だが、実際の表示時間はOSや通知設定にも左右される。
- `except Exception as exc`：通知処理で発生した例外を捕捉する。
- `log_error("デスクトップ通知", str(exc))`：通知に失敗した場合、その内容をログに記録する。

### **第3章：システム検出や計算ロジック**

- `def _detect_system_capability()`：CPU数とメモリ量を検出する関数。後段の動的なUIパラメータ設定に利用する。
- `cpu_count = os.cpu_count() or 2`：論理CPU数を取得する。取得できなかった場合は2を使用する。
- `total_mem_gb = None`：メモリ量の初期値を不明としておく。
- `try`：メモリ情報の取得に失敗する可能性があるため、例外処理で囲む。
- `if os.name == "nt"`：Windows環境かどうかを判定する。
- `import ctypes`：Windows APIを呼び出すために`ctypes`をインポートする。
- `class _MEMORYSTATUSEX(ctypes.Structure)`：Windows APIの`GlobalMemoryStatusEx`で使用する構造体を定義する。
- `_fields_ = [`：構造体のフィールドリストを開始する。Pythonの`ctypes.Structure`では、フィールド名と型を`_fields_`に指定する。
- `('dwLength', ctypes.c_ulong), ('dwMemoryLoad', ctypes.c_ulong),`：構造体のサイズとメモリ使用率を表すフィールドを定義する。
- `('ullTotalPhys', ctypes.c_ulonglong), ('ullAvailPhys', ctypes.c_ulonglong),`：物理メモリの合計と利用可能量を表すフィールドを定義する。
- `('ullTotalPageFile', ctypes.c_ulonglong), ('ullAvailPageFile', ctypes.c_ulonglong),`：ページファイルの合計と利用可能量を表すフィールドを定義する。
- `('ullTotalVirtual', ctypes.c_ulonglong), ('ullAvailVirtual', ctypes.c_ulonglong),`：仮想メモリの合計と利用可能量を表すフィールドを定義する。
- `('sullAvailExtendedVirtual', ctypes.c_ulonglong),`：ソースコードではこのフィールド名が指定されている。Windows APIの正式なフィールド名は`ullAvailExtendedVirtual`であり、ソースコード側に綴りの誤りがある。構造体ではフィールドの並びと型が重要になるが、保守性のため正式な名前に修正することが望ましい。
- `]`：フィールド定義を終了する。
- `stat = _MEMORYSTATUSEX()`：構造体のインスタンスを作成する。
- `stat.dwLength = ctypes.sizeof(_MEMORYSTATUSEX)`：API呼び出し前に構造体のサイズを設定する。
- `ctypes.windll.kernel32.GlobalMemoryStatusEx(ctypes.byref(stat))`：Windows APIを呼び出し、メモリ情報を構造体へ取得する。
- `total_mem_gb = stat.ullTotalPhys / (1024 ** 3)`：物理メモリのバイト数を1024の3乗で割り、GiB相当の値へ変換する。
- `else`：Windows以外の環境での処理。
- `with open("/proc/meminfo") as f`：Linuxのメモリ情報ファイルを開く。`with`を使うことで処理終了時にファイルを閉じる。
- `for line in f`：ファイルを1行ずつ読み込む。
- `if line.startswith("MemTotal:")`：合計メモリを表す`MemTotal:`の行を探す。
- `total_mem_gb = int(line.split()[1]) / (1024 ** 2)`：`split()[1]`で2番目のトークンを取り出し、数値を整数へ変換する。`/proc/meminfo`のメモリ値はkB単位なので、1024の2乗で割ってGiB相当の値に変換する。
- `break`：必要な情報を取得したらループを終了する。
- `except Exception as exc`：メモリ情報の取得中に発生した例外を捕捉する。
- `# 修正1: 取得失敗は致命的ではないため既定値で継続するが、原因はログへ残す`：メモリ取得に失敗しても、アプリ全体を直ちに停止させない方針を示すコメント。
- `log_error("システム能力検出", f"{exc}\\n{traceback.format_exc()}")`：例外内容とスタックトレースをログへ記録する。
- `total_mem_gb = None`：取得失敗時はメモリ量を不明として扱う。
- `return cpu_count, total_mem_gb`：CPU数とメモリ量をタプルで返す。
- `def _compute_dynamic_ui_params()`：検出したシステム性能をもとに、UIのバッチサイズなどを決める関数。
- `cpu_count, mem_gb = _detect_system_capability()`：システム性能を検出する関数を呼び出す。
- `if mem_gb is None`：メモリ量を取得できなかった場合の分岐。
- `# 修正(v0.8.4): 従来はメモリ量の取得に失敗した場合4.0GB相当として`：過去のフォールバック値について説明するコメント。
- `# 扱っていたが、取得失敗は仮想化環境・コンテナ環境・権限制限された`：メモリ情報の取得に失敗する環境についての説明。
- `# 環境などで発生しやすく、そうした環境ほど実際には低スペックである`：取得失敗と実際の性能の関係についてのコメント。ただし、取得失敗だけで低スペックだと断定できるわけではない。
- `# ケースが多い。取得できない=不明である以上、楽観的な値ではなく`：不明な値を楽観的に扱わない方針を示す。
- `# より安全側(=低スペック寄りの挙動になる)の値にフォールバックする`：取得できない場合に低スペック寄りの設定を使用する。
- `# ことで、実メモリが少ない環境でのバッチサイズ過大・再描画過多による`：バッチサイズが大きすぎる場合の問題を説明する。
- `# フリーズを避ける。`：低スペック環境での負荷を抑える目的を示す。
- `mem_gb = 2.0`：メモリ量を取得できない場合、2.0GB相当として扱う。
- `score = cpu_count + mem_gb / 2.0`：CPU数とメモリ量から簡易的なスコアを計算する。実際の性能を完全に測定するものではなく、UI設定を選ぶための目安。
- `# virtual_scroll_debounce_ms: 修正(v0.8.3): 仮想スクロールの再描画デバウンス`：仮想スクロールの再描画間隔を調整する目的を説明するコメント。
- `# 間隔。低スペック環境(スコアが低い)では描画が追いつかず取りこぼす恐れが`：低スペック環境での描画負荷について説明する。
- `# あるため間隔を広めに、高スペック環境では逆に狭めて体感の追従性を上げる。`：環境に応じてデバウンス間隔を変更する方針を示す。ただし、実際の描画の滑らかさは環境や処理内容にも左右される。
- `if score < 6`：最も低いスコアの環境向けの設定を選ぶ。
- `params = {'batch_size': 30, 'flush_interval_ms': 150, 'virtual_window': 200,`：バッチサイズ30件、フラッシュ間隔150ms、仮想ウィンドウ200件を設定する。
- `'virtual_scroll_debounce_ms': 32}`：デバウンス間隔を32msに設定し、再描画頻度を抑える。
- `elif score < 12`：中低スコアの環境向けの分岐。
- `params = {'batch_size': 80, 'flush_interval_ms': 100, 'virtual_window': 400,`：バッチサイズ80件、フラッシュ間隔100ms、仮想ウィンドウ400件を設定する。
- `'virtual_scroll_debounce_ms': 24}`：デバウンス間隔を24msに設定する。
- `elif score < 24`：中高スコアの環境向けの分岐。
- `params = {'batch_size': 150, 'flush_interval_ms': 80, 'virtual_window': 700,`：バッチサイズ150件、フラッシュ間隔80ms、仮想ウィンドウ700件を設定する。
- `'virtual_scroll_debounce_ms': 16}`：デバウンス間隔を16msに設定する。
- `else`：それ以外の環境向けの設定。
- `params = {'batch_size': 300, 'flush_interval_ms': 60, 'virtual_window': 1200,`：バッチサイズ300件、フラッシュ間隔60ms、仮想ウィンドウ1200件を設定する。
- `'virtual_scroll_debounce_ms': 10}`：デバウンス間隔を10msに設定し、高性能環境でのスクロール追従性の向上を意図する。ただし、この値だけで滑らかな動作が保証されるわけではない。
- `params['cpu_count'] = cpu_count`：検出したCPU数を診断用情報として辞書に追加する。
- `params['mem_gb'] = mem_gb`：メモリ量も診断用情報として追加する。
- `return params`：設定辞書を返す。
- `DYNAMIC_UI_PARAMS = _compute_dynamic_ui_params()`：モジュール読み込み時に1回計算し、グローバル変数へ保存する。

### **第4章：NTFS ACL保護の土台とマーカー管理**

`修正(v0.9.7・NTFS ACLによる保護)`：Windowsでは`os.chmod(0o600)`だけでは、Unix系OSと同じように他ユーザーからの読み取りを制限できない場合があります。

そのため、他ユーザーからの読み取りを防ぐには、NTFS ACLなど別の対策も必要になります。

そこでWindowsでは`icacls`（OS標準コマンド）で、対象フォルダの継承を切り、現在のユーザーとSYSTEMにフルコントロールを設定する処理を行います。

フォルダに`OI`と`CI`（配下へ継承）を指定することで、その中に新規作成されるファイルやサブフォルダにも権限が継承されることが期待されます。ただし、実際のACL設定やファイルの作成場所・置換方法によって異なるため、常に同じ保護が適用されるとは限りません。

`os.replace()`でファイルを置き換える場合も、置換先の権限が必ず期待どおりになるとは限らないため、実際のACLを確認することが重要です。

`icacls`呼び出しはリスト形式の引数を使い、`shell=True`は使用しません。これにより、シェルによるコマンド文字列の解釈を避けています。

- `_NTFS_ACL_ATTEMPTED = set()`：どのパスにACL適用を試みたかを記録する集合。重複実行を防ぐために使う。
- `_NTFS_ACL_LOCK = threading.Lock()`：上記集合への並行アクセスを制御するロック。
- `_ACL_MARKER_NAME = ".acl_applied"`：ACL適用済みを示す目印ファイル名。2回目以降の`icacls`実行を省略するための高速化マーカー。
- `def _read_acl_marker(directory)`：マーカーを読み取る関数。
- `try`：ファイル読み込み時の例外に備える。
- `with open(os.path.join(directory, _ACL_MARKER_NAME), "r", encoding="utf-8") as f`：目印ファイルを開く。
- `return f.read().strip()`：ファイル内容を読み取り、前後の空白や改行を取り除いて返す。
- `except OSError`：ファイルが存在しない場合などのOSエラーを捕捉する。
- `return None`：マーカーを読み取れなかった場合は`None`を返す。
- `def _write_acl_marker(directory, account)`：マーカーを書き込む関数。
- `try`：書き込み時の例外に備える。
- `with open(os.path.join(directory, _ACL_MARKER_NAME), "w", encoding="utf-8") as f`：目印ファイルを書き込みモードで開く。
- `f.write(account)`：アカウント名を書き込む。
- `except OSError as exc`：書き込み失敗を捕捉する。
- `log_error("NTFS ACL設定(目印書込)", f"{directory}: {exc}")`：失敗内容をログに記録する。
- `_ACL_FAILED_NAME = ".acl_failed"`：ACL設定の失敗を示すマーカー名。ネットワークフォルダなどで毎回長い待ち時間が発生することを避けるために使う。
- `_ACL_FAILED_RETRY_SEC = 7 * 24 * 3600`：再試行までの期間を7日間に設定する。
- `def _acl_failed_recently(directory)`：最近ACL設定に失敗したかを判定する関数。
- `"""前回のACL設定が失敗していて、まだ再試行する時期でなければTrue`：docstring。再試行を抑える目的を説明する。
- `(ネットワーク上のフォルダ等で、起動のたびに最大15秒待たされるのを避ける)。"""`：docstringの続き。
- `try`：ファイルの更新時刻を確認する処理を保護する。
- `return (time.time() - os.path.getmtime(os.path.join(directory, _ACL_FAILED_NAME)) < _ACL_FAILED_RETRY_SEC)`：失敗マーカーの更新時刻から経過時間を計算し、設定期間内か判定する。
- `except OSError`：マーカーがない場合などのエラーを捕捉する。
- `return False`：最近の失敗が確認できない場合は、再試行を妨げない。
- `def _write_acl_failed_marker(directory)`：失敗マーカーを書き込む関数。
- `try`：書き込み時の例外に備える。
- `with open(os.path.join(directory, _ACL_FAILED_NAME), "w", encoding="utf-8") as f`：失敗マーカーを開く。
- `f.write(datetime.now().strftime("%Y-%m-%d %H:%M:%S"))`：失敗時刻を人間が読める形式で記録する。
- `except OSError`：書き込み失敗を捕捉する。
- `pass`：マーカーを書き込めなかった場合も、ここでは追加処理を行わない。
- `def _clear_acl_failed_marker(directory)`：失敗マーカーを削除する関数。
- `try`：削除時の例外に備える。
- `os.remove(os.path.join(directory, _ACL_FAILED_NAME))`：失敗マーカーを削除する。
- `except OSError`：削除できなかった場合の例外を捕捉する。
- `pass`：削除失敗によってアプリ全体が停止することを避ける。

**補足：ACLマーカーの限界**

`.acl_applied`は処理を省略するための目印であり、現在のACLが正しく設定されていることを保証するものではありません。マーカーが偽装された場合や、外部からACLが変更された場合、マーカーの内容が一致していても実際のアクセス制御が期待どおりでない可能性があります。マーカーだけでACLの真正性を保証できるわけではない点に注意が必要です。

### **第5章：NTFS ACL適用と失敗時のリトライ制御**

- `def _apply_ntfs_owner_only_acl(path, is_dir)`：WindowsでACLを設定する関数。`is_dir`でフォルダかファイルかを区別する。
- `"""Windowsのみ。成功したらTrue、失敗/非対応ならFalse(失敗はログへ記録)。`：docstring。戻り値の意味を説明する。
- `同じパスに対する再試行(icacls多重起動)を避けるため、結果は1プロセス内で`：二重起動を防ぐ目的を説明する。
- `1回だけ試みる。"""`：docstringの終了。
- `if os.name != "nt"`：Windows以外の環境かを判定する。
- `return False`：Windows以外では非対応として`False`を返す。
- `key = os.path.normcase(os.path.abspath(path))`：パスを絶対パスに変換し、大文字小文字を正規化する。
- `with _NTFS_ACL_LOCK`：共有集合へのアクセスをロックする。
- `if key in _NTFS_ACL_ATTEMPTED`：すでに処理を試みたパスか確認する。
- `return key in _NTFS_ACL_ATTEMPTED_OK`：成功済みの集合に入っていれば`True`を返す。
- `_NTFS_ACL_ATTEMPTED.add(key)`：初回の処理対象として記録する。
- `ok = False`：成功フラグを初期化する。
- `try`：ACL設定処理を例外処理で囲む。
- `username = os.environ.get("USERNAME") or ""`：環境変数からユーザー名を取得する。
- `domain = os.environ.get("USERDOMAIN") or ""`：環境変数からドメイン名を取得する。
- `if not username or "\x00" in path`：ユーザー名が取得できない、またはパスにヌル文字が含まれているかを確認する。この条件は攻撃の有無を判定するものではなく、ユーザー名の有無とパス内のヌル文字を確認するもの。
- `raise ValueError("ユーザー名を特定できないためACL設定を中止しました")`：条件に該当する場合、例外を発生させてACL設定を中断する。
- `account = f"{domain}\\{username}" if domain else username`：ドメインがある場合は`DOMAIN\USER`形式のアカウント文字列を作成する。ドメインがない場合はユーザー名だけを使用する。
- `# 修正(v0.9.7・起動高速化): icaclsは起動のたびに同期実行すると起動が遅くなる。`：起動時の待ち時間を抑えるためのコメント。
- `# 設定済みのフォルダには目印ファイルを置き、同じアカウントで設定済みなら`：マーカーによる処理省略の説明。
- `# 2回目以降はicaclsを呼ばずにスキップする。`：2回目以降の処理を省略する方針を示す。
- `if is_dir and _read_acl_marker(path) == account`：フォルダであり、マーカーの内容が現在のアカウントと一致するか確認する。
- `with _NTFS_ACL_LOCK`：成功集合へのアクセスをロックする。
- `_NTFS_ACL_ATTEMPTED_OK.add(key)`：成功済みとして集合へ追加する。
- `return True`：マーカーの内容が一致する場合、実際の`icacls`呼び出しを省略して成功扱いにする。ただし、これはマーカーの内容を信頼した判断であり、現在のACL状態を再検証しているわけではない。
- `if is_dir and _acl_failed_recently(path)`：フォルダのACL設定に最近失敗しているか確認する。
- `return False`：再試行期間内なら処理を省略する。
- `perm = "(OI)(CI)F" if is_dir else "F"`：フォルダの場合は子ファイル・子フォルダへの継承を指定したフルコントロール、ファイルの場合は対象ファイルへのフルコントロールを指定する。
- `# 先にgrantを並べ、その後で継承を切る(アカウント名の解決に失敗した場合に`：権限付与と継承設定の順序に関するコメント。
- `# 「継承だけ切れて誰もアクセスできない」状態になるのを避けるため)`：継承だけが先に切られるリスクを抑える意図を説明する。
- `cmd = ["icacls", os.path.abspath(path),`：`icacls`コマンドをリスト形式で構築する。
- `"/grant:r", f"{account}:{perm}",`：現在のユーザーに指定した権限を設定する。
- `"/grant:r", f"*S-1-5-18:{perm}",`：SYSTEMアカウントにも指定した権限を設定する。`S-1-5-18`はSYSTEMのセキュリティ識別子。
- `"/inheritance:r"]`：親から継承したアクセス許可を削除する指定。
- `creationflags = getattr(subprocess, "CREATE_NO_WINDOW", 0)`：Windowsでコンソールウィンドウを表示しないためのフラグを取得する。
- `result = subprocess.run(cmd, capture_output=True, text=True, timeout=15,`：コマンドを実行し、出力を取得する。15秒のタイムアウトを設定する。
- `creationflags=creationflags)`：取得したフラグを適用する。
- `if result.returncode != 0`：コマンドの終了コードから失敗を判定する。
- `raise OSError((result.stderr or result.stdout or "").strip()`：エラー出力を例外に含める。
- `or f"icaclsがエラー終了しました(コード: {result.returncode})")`：出力がない場合は終了コードを含むメッセージを使用する。
- `ok = True`：ここまで処理が進んだ場合は成功フラグを設定する。
- `if is_dir`：フォルダの場合の後処理。
- `_write_acl_marker(path, account)`：成功マーカーを書き込む。
- `_clear_acl_failed_marker(path)`：失敗マーカーを削除する。
- `except Exception as exc`：処理中に発生した例外を捕捉する。
- `log_error("NTFS ACL設定", f"{path}: {exc}（1週間は再試行しません）" if is_dir else f"{path}: {exc}")`：失敗内容をログに記録する。
- `if is_dir`：フォルダの場合の失敗処理。
- `_write_acl_failed_marker(path)`：失敗マーカーを書き込む。
- `if ok`：成功時の後処理。
- `with _NTFS_ACL_LOCK`：成功集合へのアクセスをロックする。
- `_NTFS_ACL_ATTEMPTED_OK.add(key)`：成功済みのパスとして記録する。
- `return ok`：成功したかどうかを返す。
- `_NTFS_ACL_ATTEMPTED_OK = set()`：成功したパスを保持する集合。
- `def _restrict_directory_permissions(directory)`：フォルダ権限を制限するための関数。
- `"""修正(v0.9.7): フォルダを所有者のみアクセス可能にする。`：docstring。フォルダ権限を制限する目的を説明する。
- `Windows: NTFS ACL(配下へ継承)。その他: chmod 0o700。"""`：WindowsではNTFS ACL、その他の環境では`chmod`を使用する方針。
- `try`：権限設定の例外に備える。
- `if os.name == "nt"`：Windows環境か確認する。
- `_apply_ntfs_owner_only_acl(directory, is_dir=True)`：フォルダ向けのACL設定関数を呼び出す。
- `else`：Windows以外の処理。
- `os.chmod(directory, 448) # 0o700`：Unix系では`0o700`相当の権限を指定する。所有者に読み取り・書き込み・実行権限を設定する意図だが、実際の適用結果はOSやファイルシステムなどにも左右される。
- `except OSError as exc`：権限設定失敗を捕捉する。
- `log_error("フォルダ権限設定", f"{directory}: {exc}")`：失敗内容をログに記録する。

### **第6章：権限制限ラッパーとファイル・フォルダ保護**

- `def _restrict_file_permissions(path)`：ファイル用の権限制限関数。鍵や平文DBなど、機密情報を含む可能性のあるファイルに使う。
- `"""修正(v0.8.0・インデックスDBセキュリティ): 暗号鍵や復号後の平文DBなど、`：docstring。対象ファイルの機密性について説明する。
- `所有者以外からは読めないようにしたいファイルに対して、所有者のみ読み書き可能な`：所有者のみのアクセスを意図する。
- `権限を設定する。`：権限設定の目的を説明する。
- `修正(v0.9.7): Windowsではchmodがほぼ無効なため、親フォルダへNTFS ACL(配下へ継承)を`：Windowsでは親フォルダ側のACLによる保護も試みる。
- `設定し、それが成功していれば個別ファイルへの設定は不要(継承で保護される)。`：親フォルダのACL設定が成功していれば、子ファイルにも権限が継承されることを期待する。
- `親フォルダへの設定に失敗した場合のみ、そのファイル単体へACLを試みる。`：親フォルダへの設定に失敗した場合の代替処理。
- `chmod(0o600)は多重防御としてプラットフォームを問わず従来どおり試みる。`：従来の権限設定も実行する。
- `失敗しても機能停止するほどの問題ではないため例外は握り潰すが、権限系の`：権限設定に失敗してもアプリを停止させない方針。
- `問題は原因調査の価値があるためログには残す。"""`：失敗の原因を後から調べられるようにする。
- `try`：権限設定処理を開始する。
- `os.chmod(path, 384)`：`0o600`相当の権限を指定する。Unix系では所有者の読み取り・書き込み権限に相当するが、Windowsではこの指定だけでNTFS ACLによるアクセス制御が同じように設定されるとは限らない。
- `except OSError as exc`：権限設定時のOSエラーを捕捉する。
- `log_error("ファイル権限設定", f"{path}: {exc}")`：失敗内容をログに記録する。
- `if os.name == "nt"`：Windows環境かどうかを判定する。
- `parent = os.path.dirname(os.path.abspath(path))`：親フォルダの絶対パスを取得する。
- `if not _apply_ntfs_owner_only_acl(parent, is_dir=True)`：親フォルダへのACL設定が成功しなかったかを確認する。
- `_apply_ntfs_owner_only_acl(path, is_dir=False)`：親フォルダへの設定が失敗した場合、ファイル単体へのACL設定を試みる。

### **第7章：暗号鍵の生成・保存・移行**

- `def _load_or_create_encryption_key()`：暗号鍵の読み込み、または新規作成を行う関数。インデックスDB暗号化に必要な鍵を扱う。
- `crypto = _lazy_import_crypto()`：`cryptography`を必要なタイミングで読み込む。
- `if crypto is None`：暗号ライブラリが利用できない場合の分岐。
- `raise RuntimeError("cryptography ライブラリが利用できません")`：ライブラリがない場合は例外を発生させ、鍵を読み込む処理を中断する。
- `Fernet, _InvalidToken = crypto`：Fernetクラスと例外クラスを取得する。
- `# 修正(v0.9.7): 鍵はDB等のあるAPP_DATA_DIRとは別のKEY_DIRに置く。`：鍵とDBを別の場所に保存する方針を説明する。
- `os.makedirs(KEY_DIR, exist_ok=True)`：鍵用ディレクトリを作成する。すでに存在していてもエラーにしない。
- `_restrict_directory_permissions(KEY_DIR)`：鍵用ディレクトリの権限制限を試みる。OSや環境によって結果が異なるため、この呼び出しだけで保護が必ず完了するわけではない。
- `if os.path.exists(ENCRYPTION_KEY_PATH)`：鍵ファイルがすでに存在するかを確認する。
- `# 修正(v0.8.0): 従来は鍵ファイルを新規作成した場合のみ chmod していたため、`：既存鍵ファイルの権限に関する修正の説明。
- `# 既に存在する鍵ファイル(例えば旧バージョンで作成されたもの、または`：過去のバージョンで作られた鍵も対象にする。
- `# 何らかの理由で権限が緩められたもの)の権限が緩いまま放置される`：権限が緩い状態で残る可能性について説明する。
- `# 可能性があった。読み込みのたびに毎回、権限を再確認・修正する。`：読み込みのたびに権限制限を試みる方針。
- `_restrict_file_permissions(ENCRYPTION_KEY_PATH)`：鍵ファイルの権限制限を試みる。
- `with open(ENCRYPTION_KEY_PATH, "rb") as f`：鍵ファイルをバイナリモードで開く。
- `return f.read()`：鍵データを読み込んで返す。
- `if os.path.exists(LEGACY_ENCRYPTION_KEY_PATH)`：旧保存場所に鍵があるか確認する。
- `return _migrate_legacy_encryption_key()`：旧鍵がある場合は移行関数へ処理を委譲する。
- `key = Fernet.generate_key()`：新しい暗号鍵を生成する。
- `_write_key_file_atomic(key)`：鍵を一時ファイル経由で書き込む。
- `return key`：生成した鍵を返す。
- `def _write_key_file_atomic(key)`：鍵をアトミックに書き込む関数。
- `"""修正(v0.9.7): 鍵を一時ファイル経由で書き込み、fsync後にos.replace()で`：一時ファイルを使用する目的を説明する。
- `確定する(書き込み途中の不完全な鍵ファイルを残さないため)。"""`：不完全な鍵ファイルが保存先に残るリスクを抑える。
- `tmp_path = ENCRYPTION_KEY_PATH + ".tmp"`：一時ファイルのパスを作成する。
- `try`：書き込み中の例外に備える。
- `with open(tmp_path, "wb") as f`：一時ファイルをバイナリ書き込みモードで開く。
- `f.write(key)`：鍵データを書き込む。
- `f.flush()`：Python側のバッファをフラッシュする。
- `os.fsync(f.fileno())`：OSに対してファイル内容の同期を要求する。
- `os.replace(tmp_path, ENCRYPTION_KEY_PATH)`：一時ファイルを保存先へ置き換える。通常は置換途中の状態が見えることを避けられるが、あらゆる障害時のデータ保全まで保証するわけではない。
- `finally`：後始末を実行する。
- `if os.path.exists(tmp_path)`：一時ファイルが残っているか確認する。
- `try`：削除時の例外に備える。
- `os.remove(tmp_path)`：一時ファイルを削除する。
- `except OSError`：削除失敗を捕捉する。
- `pass`：削除失敗時には追加処理を行わない。
- `_restrict_file_permissions(ENCRYPTION_KEY_PATH)`：置き換え後の鍵ファイルに対して権限制限を試みる。
- `def _migrate_legacy_encryption_key()`：旧保存場所の鍵を新しい保存場所へ移行する関数。
- `"""修正(v0.9.7): 旧バージョンがAPP_DATA_DIR直下に作成した鍵を、新しい保存場所へ移行する。`：移行の背景を説明する。
- `鍵を失うと暗号化DBが二度と復号できなくなるため、次の順序で安全側に進める。`：鍵を失わないようにする方針を説明する。
- `1. 旧鍵を読み込む`：旧鍵を読み込む。
- `2. 新しい場所へ書き込み、読み戻して内容が完全一致することを確認する`：移行先へ書き込んだ後、内容を読み戻して検証する。
- `3. 確認できた場合のみ旧鍵を削除する(削除に失敗しても新しい鍵は有効なので継続)`：検証後に旧鍵の削除を試みる。
- `新しい場所への書き込み・検証のいずれかに失敗した場合は、旧鍵をそのまま返す`：移行に失敗した場合は旧鍵を維持する。
- `（旧鍵は削除しない）ことで、移行に失敗した場合に旧鍵を使い続け、鍵を失うリスクを抑える方針です。ただし、これによって鍵の喪失を絶対に防げるわけではありません。"""`：鍵喪失のリスクを抑える方針を説明する。
- `with open(LEGACY_ENCRYPTION_KEY_PATH, "rb") as f`：旧鍵をバイナリモードで開く。
- `key = f.read()`：旧鍵を読み込む。
- `try`：新しい場所への書き込みと検証を開始する。
- `_write_key_file_atomic(key)`：新しい場所へ鍵を書き込む。
- `with open(ENCRYPTION_KEY_PATH, "rb") as f`：移行先の鍵を読み込む。
- `if f.read() != key`：移行先の内容が元の鍵と一致するか確認する。
- `raise OSError("移行先の鍵ファイルの内容が一致しません")`：内容が一致しない場合は例外を発生させる。
- `except OSError as exc`：書き込みや検証に失敗した場合の処理。
- `log_error("暗号鍵移行", f"新しい場所への移行に失敗したため旧鍵を継続使用します: {exc}")`：失敗内容をログに記録する。
- `_restrict_file_permissions(LEGACY_ENCRYPTION_KEY_PATH)`：旧鍵の権限制限を試みる。
- `return key`：旧鍵を返し、鍵を失うリスクを抑える。
- `try`：旧鍵の削除処理を開始する。
- `os.remove(LEGACY_ENCRYPTION_KEY_PATH)`：旧鍵ファイルの削除を試みる。
- `except OSError as exc`：削除失敗を捕捉する。
- `log_error("暗号鍵移行", f"旧鍵ファイルの削除に失敗しました(新しい鍵は有効です): {exc}")`：削除失敗をログに記録する。
- `log_error("暗号鍵移行", "暗号鍵をDBとは別のフォルダへ移行しました")`：移行処理の記録を残す。
- `return key`：移行した鍵を返す。

### **第8章：インデックスDBの暗号化・復号とWAL管理**

- `def _cleanup_wal_files()`：SQLiteのWAL関連ファイルを掃除する関数。
- `for suffix in ('-wal', '-shm')`：`wal`と`shm`の2種類を対象にする。
- `path = INDEX_DB_PATH + suffix`：対象ファイルのパスを作成する。
- `if not os.path.exists(path)`：ファイルが存在するか確認する。
- `continue`：存在しない場合は次のファイルへ進む。
- `try`：削除時の例外に備える。
- `os.remove(path)`：対象ファイルを削除する。
- `except OSError`：削除時のOSエラーを捕捉する。
- `continue`：削除できない場合も次のファイルへ進む。
- `def decrypt_index_db_if_needed()`：必要に応じて暗号化DBを復号する関数。インデックス検索を実際に使う直前に実行することで、通常起動時の処理を減らす。
- `# 暗号化DBの復号は起動時ではなく、インデックス検索を実際に使う直前`：遅延復号の方針を説明する。
- `# (IndexManager 初期化の直前)にのみ呼び出す。これにより通常起動では`：呼び出しタイミングの説明。
- `# cryptography の読み込みも復号処理も発生しない。`：通常起動時の処理を減らす意図。
- `# 修正(v0.8.0): 戻り値は「復号を試みて失敗したか」を表す`：戻り値の意味を説明する。
- `# (True=問題なし/復号不要、False=復号を試みて失敗した)。`：通常は`True`を問題なし、復号失敗時は`False`として扱う方針。
- `# 呼び出し元(ensure_index_backend)はFalseの場合、異常系として`：呼び出し元の処理について説明する。
- `# 次回起動を待たずその場でintegrity_checkを強制する。`：異常系で整合性チェックを実行する方針。
- `crypto = _lazy_import_crypto()`：暗号ライブラリを遅延インポートする。
- `if crypto is None`：暗号ライブラリが利用できない場合の分岐。
- `return True`：**この戻り値だけでは、暗号化DBが存在しないことを確認した意味にはなりません。暗号化DBが存在する場合でも`True`が返る実装では、呼び出し元が復号不要と誤認する可能性があります。暗号化DBが存在し、平文DBが存在しない状態で`cryptography`が利用できない場合、後続処理で空のDBが作られる可能性があるため注意が必要です。**
- `_Fernet, InvalidToken = crypto`：Fernetクラスと復号失敗時の例外クラスを取得する。
- `Fernet = _Fernet`：Fernetクラスを扱いやすい名前へ割り当てる。
- `if not os.path.exists(ENCRYPTED_DB_PATH)`：暗号化ファイルが存在するか確認する。
- `return True`：暗号化ファイルが存在しない場合は復号不要として返す。
- `if os.path.exists(INDEX_DB_PATH)`：暗号化DBと平文DBの両方が存在する状態か確認する。
- `# 修正(v0.8.0・平文DBの残留防止): 暗号化ファイルと平文ファイルの両方が`：平文DBが残留する場合の説明。
- `# 既に存在する状態は、前回セッションが正常終了せず`：前回セッションの終了状態について説明する。
- `# (クラッシュ・強制終了・電源断など)、暗号化前の平文DBが削除されない`：異常終了時に平文DBが残る可能性を説明する。
- `# まま残ってしまったケースを意味する。この状態を放置すると、`：残留した場合のリスクについて説明する。
- `# ディスク上に検索対象のファイル名・パス情報を含む平文DBが`：平文DBに機密性のある情報が含まれる可能性を説明する。
- `# 無期限に残り続けてしまう。`：平文DBが残留し続ける可能性について説明する。
- `# 暗号化ファイル(ENCRYPTED_DB_PATH)を正として扱い、残っていた平文`：暗号化ファイルを正として扱う方針。
- `# DBは古い可能性があるため破棄してから復号し直す(平文DBの方を正として`：平文DBを正とすると最新の変更を失う可能性があることを説明する。
- `# 使うと、直前に暗号化されたはずの最新の変更を取りこぼす恐れがあるため)。`：暗号化ファイルを優先する理由を説明する。
- `log_error(`：ログ記録を開始する。
- `"インデックスDB",`：ログのカテゴリを指定する。
- `"前回セッションの残留とみられる平文DBを検出したため、"`：残留DBを検出したことを記録する。
- `"暗号化ファイルを正として破棄し復号し直します")`：暗号化ファイルを正として処理することを記録する。
- `try`：平文DBの削除処理を開始する。
- `os.remove(INDEX_DB_PATH)`：残留した平文DBを削除する。
- `except OSError as exc`：削除失敗を捕捉する。
- `log_error("インデックスDB(残留平文DB削除)", str(exc))`：削除失敗をログに記録する。
- `_cleanup_wal_files()`：WAL関連ファイルを掃除する。
- `tmp_path = INDEX_DB_PATH + ".tmp"`：復号用の一時ファイルパスを作成する。
- `try`：復号処理を開始する。
- `key = _load_or_create_encryption_key()`：暗号鍵を読み込む、または必要に応じて作成する。
- `fernet = Fernet(key)`：Fernetインスタンスを作成する。
- `with open(ENCRYPTED_DB_PATH, "rb") as f`：暗号化DBをバイナリモードで開く。
- `decrypted = fernet.decrypt(f.read())`：暗号化データを読み込み、復号する。
- `with open(tmp_path, "wb") as f`：一時ファイルをバイナリ書き込みモードで開く。
- `f.write(decrypted)`：復号したDBデータを書き込む。
- `f.flush()`：Python側のバッファをフラッシュする。
- `os.fsync(f.fileno())`：OSにファイル内容の同期を要求する。
- `os.replace(tmp_path, INDEX_DB_PATH)`：一時ファイルを平文DBの保存先へ置き換える。
- `# 修正(v0.8.0): 復号された平文DBはファイル名・ディレクトリ検索の対象パス等、`：復号後のDBにも機密性のある情報が含まれる可能性を説明する。
- `# 機密性のある情報を含みうるため、暗号鍵ファイルと同様に所有者のみ`：権限制限の必要性を説明する。
- `# 読み書き可能な権限に制限しておく。`：平文DBの権限制限を試みる。
- `_restrict_file_permissions(INDEX_DB_PATH)`：平文DBの権限制限を試みる。
- `# 修正(v0.8.0・WAL/SHMの扱い確認): 復号直後、まだDBへ一切アクセスしていない`：WAL関連ファイルの掃除タイミングを説明する。
- `# このタイミングで前回セッションのWAL/SHMファイルの残骸を掃除しておく。`：復号直後に掃除する方針。
- `# encrypt_index_db()側では暗号化直前(PRAGMA wal_checkpoint実行後)に`：暗号化前にもWAL関連の処理を行うことを説明する。
- `# 掃除しているため通常は残らないはずだが、前回が異常終了だった場合は`：異常終了時に残骸が残る可能性を説明する。
- `# チェックポイントされないままのWAL/SHMが残っている可能性がある。`：WAL関連ファイルが残る可能性を説明する。
- `# 復号直後の(まだ誰もconnectしていない)DBファイルに対して、古いWAL/SHMを`：古いWAL関連ファイルを持ち越す問題について説明する。
- `# 持ち越すと、journal_mode=WAL再開時に不整合の原因になりうるため、`：不整合の可能性を説明する。
- `# ここで明示的に取り除いておく(WAL/SHMが無くても実害は無い操作)。`：復号直後に掃除する方針。
- `_cleanup_wal_files()`：WAL関連ファイルを掃除する。
- `return True`：復号成功として返す。
- `except (InvalidToken, OSError, ValueError) as exc`：鍵不一致、ファイル破損、OSエラーなど、指定した例外を捕捉する。
- `# 修正1: 復号失敗(鍵不一致・破損ファイル等)は原因調査に必須なのでログへ残す`：復号失敗を記録する方針。
- `log_error("インデックスDB復号", f"{exc}\\n{traceback.format_exc()}")`：例外内容とスタックトレースをログへ記録する。
- `if os.path.exists(tmp_path)`：一時ファイルが残っているか確認する。
- `try`：一時ファイル削除の例外に備える。
- `os.remove(tmp_path)`：一時ファイルを削除する。
- `except OSError as cleanup_exc`：削除失敗を捕捉する。
- `log_error("インデックスDB復号(一時ファイル削除)", str(cleanup_exc))`：削除失敗をログに記録する。
- `# 修正(v0.8.0・復号失敗時のDB破壊防止): 従来はログを残すのみで、`：復号失敗時の処理方針を説明する。
- `# 復号に失敗した暗号化ファイルをそのままにしていた。この状態では`：従来の問題点について説明する。
- `# ENCRYPTED_DB_PATH が存在するにもかかわらず INDEX_DB_PATH が`：暗号化DBがある状態を説明する。
- `# 用意できないため、後続の IndexManager() 初期化で「空の新規DB」として`：復号に失敗した場合に空DBとして扱われる可能性を説明する。
- `# 扱われてしまい、ユーザーからは実質的にインデックスの中身が消えたのと`：ユーザーからはデータ消失に見える可能性を説明する。
- `# 同じ結果に見える(実際には暗号化ファイルは残っているが、鍵不一致や`：暗号化ファイルが残っていても復号できない可能性があることを説明する。
- `# ファイル破損で二度と復号できない可能性が高い)。`：ファイル破損時のリスクを説明する。
- `# そのため、復号に失敗した暗号化ファイルは上書き・混同を避けるため`：暗号化ファイルを退避する目的を説明する。
- `# 別名(.corrupt_タイムスタンプ)で退避し、次回同じ壊れたファイルに対して`：退避ファイル名の形式を説明する。
- `# 復号を再試行し続けないようにする。退避しておくことで、原因調査や`：再試行を避け、原因調査を可能にする。
- `# 手動でのサルベージ(鍵の再入手など)の余地も残す。`：復旧の可能性を残す。
- `if os.path.exists(ENCRYPTED_DB_PATH)`：暗号化ファイルがまだ存在するか確認する。
- `timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")`：退避ファイル名に使用するタイムスタンプを作成する。
- `quarantine_path = f"{ENCRYPTED_DB_PATH}.corrupt_{timestamp}"`：退避先のパスを作成する。
- `try`：退避処理を開始する。
- `os.replace(ENCRYPTED_DB_PATH, quarantine_path)`：暗号化ファイルを別名へ移動する。
- `log_error(`：退避のログを記録する。
- `"インデックスDB復号",`：ログのカテゴリを指定する。
- `f"復号に失敗した暗号化DBを退避しました: {quarantine_path}"`：退避先をログに記録する。
- `"（次回起動時は新規インデックスとして扱われます）")`：次回起動時の扱いをログに記録する。
- `except OSError as quarantine_exc`：退避処理の失敗を捕捉する。
- `log_error("インデックスDB復号(退避)", str(quarantine_exc))`：退避失敗をログに記録する。
- `return False`：復号失敗を呼び出し元へ伝える。
- `def encrypt_index_db()`：終了時に平文DBを暗号化して保存する関数。
- `crypto = _lazy_import_crypto()`：暗号ライブラリを遅延インポートする。
- `if crypto is None`：暗号ライブラリが利用できない場合の分岐。
- `return`：暗号ライブラリが利用できない場合は暗号化処理を行わずに終了する。この場合、平文DBが残る可能性があるため、実際の運用ではその状態を確認する必要がある。
- `Fernet, _InvalidToken = crypto`：Fernetクラスと例外クラスを取得する。
- `if not os.path.exists(INDEX_DB_PATH)`：平文DBが存在するか確認する。
- `return`：平文DBが存在しなければ暗号化不要として終了する。
- `tmp_path = ENCRYPTED_DB_PATH + ".tmp"`：暗号化用の一時ファイルパスを作成する。
- `try`：暗号化処理を開始する。
- `conn = sqlite3.connect(INDEX_DB_PATH, timeout=30)`：DBに接続し、ロック解除を最大30秒待つ設定を行う。
- `conn.execute("PRAGMA wal_checkpoint(TRUNCATE)")`：WALのチェックポイントを実行し、WALの内容をDB本体へ反映してWALを切り詰める。
- `conn.close()`：DB接続を閉じる。
- `_cleanup_wal_files()`：残っているWAL関連ファイルを掃除する。
- `key = _load_or_create_encryption_key()`：暗号鍵を読み込む、または作成する。
- `fernet = Fernet(key)`：Fernetインスタンスを作成する。
- `with open(INDEX_DB_PATH, "rb") as f`：平文DBをバイナリモードで開く。
- `encrypted = fernet.encrypt(f.read())`：DBの内容を読み込み、暗号化する。
- `with open(tmp_path, "wb") as f`：一時ファイルをバイナリ書き込みモードで開く。
- `f.write(encrypted)`：暗号化したデータを書き込む。
- `f.flush()`：Python側のバッファをフラッシュする。
- `os.fsync(f.fileno())`：OSにファイル内容の同期を要求する。
- `os.replace(tmp_path, ENCRYPTED_DB_PATH)`：一時ファイルを暗号化DBの保存先へ置き換える。
- `os.remove(INDEX_DB_PATH)`：平文DBファイルを削除する。ただし、通常のファイル削除は、記憶媒体上のデータを復元不可能な形で完全消去することを保証するものではない。
- `except OSError as exc`：OSエラーを捕捉する。ただし、SQLite関連の例外など、`OSError`以外の例外はこの節では捕捉されない可能性がある。
- `# 修正1: 終了処理中の暗号化失敗はデータ消失につながりうるため必ずログへ残す`：暗号化失敗を記録する方針。
- `log_error("インデックスDB暗号化", f"{exc}\\n{traceback.format_exc()}")`：例外内容とスタックトレースをログへ記録する。
- `if os.path.exists(tmp_path)`：一時ファイルが残っているか確認する。
- `try`：一時ファイルの削除処理を開始する。
- `os.remove(tmp_path)`：一時ファイルを削除する。
- `except OSError as cleanup_exc`：削除失敗を捕捉する。
- `log_error("インデックスDB暗号化(一時ファイル削除)", str(cleanup_exc))`：削除失敗をログに記録する。

**暗号化処理についての注意点**

現在の説明対象コードでは、`encrypt_index_db()`が`OSError`を中心に例外を捕捉しているため、SQLite関連の例外などが発生した場合、一時ファイルの削除処理を通らずに終了する可能性があります。その場合、一時ファイルや平文DBが残るおそれがあります。コード側では例外処理の範囲を見直し、後始末を`finally`で行うなどの対策を検討する必要があります。

また、`decrypt_index_db_if_needed()`では、`cryptography`が利用できない場合に暗号化DBの存在を確認せず`True`を返す経路があります。暗号化DBが存在する状態で復号できないまま後続処理が進むと、空の新規DBが作られたように見える可能性があります。暗号化DBが存在する場合は、復号できないことを呼び出し元へ伝えるなど、コード側の修正も検討すべきです。

### **第9章：ファイルユーティリティとハッシュ計算**

- `def human_size(num_bytes)`：バイト数を人間が読みやすい単位へ変換する関数。
- `if num_bytes is None`：値が`None`か確認する。
- `return ""`：`None`の場合は空文字を返す。
- `step = 1024.0`：単位変換に使用する基準値を設定する。
- `for unit in ('B', 'KB', 'MB', 'GB', 'TB')`：単位を小さい順に確認する。
- `if num_bytes < step`：現在の単位で表示できるか判定する。
- `if unit == 'B'`：バイト単位か確認する。
- `return f"{num_bytes:.0f} {unit}"`：バイト数を小数点なしで表示する。
- `return f"{num_bytes:.1f} {unit}"`：KB以上は小数点以下1桁で表示する。
- `num_bytes /= step`：次の単位へ変換するため1024で割る。
- `return f"{num_bytes:.1f} PB"`：TBを超えた値はPB単位で表示する。
- `def compute_file_hashes(path, algorithms=HASH_ALGORITHMS, stop_event=None, chunk_size=1048576)`：ファイルの複数ハッシュを計算する関数。重複検出などに利用する。
- `hashers = {name: hashlib.new(name) for name in algorithms}`：指定されたアルゴリズムごとにハッシュオブジェクトを作成する。
- `with open(path, "rb") as f`：ファイルをバイナリモードで開く。
- `while True`：ファイルをチャンク単位で読み込むループを開始する。
- `if stop_event is not None and stop_event.is_set()`：キャンセル用のイベントが設定されていて、中断要求が出ているか確認する。
- `return None`：中断された場合は`None`を返す。
- `chunk = f.read(chunk_size)`：指定したサイズでファイルを読み込む。デフォルトは1,048,576バイト（1MiB）。
- `if not chunk`：読み込むデータがなくなったか確認する。
- `break`：ファイルの終端に到達したらループを終了する。
- `for h in hashers.values()`：各ハッシュオブジェクトを順番に処理する。
- `h.update(chunk)`：読み込んだチャンクをハッシュ計算へ反映する。複数のハッシュを計算する場合でも、ファイルを何度も最初から読み直さずに処理できる。
- `return {name: h.hexdigest() for name, h in hashers.items()}`：各ハッシュの計算結果を16進文字列に変換し、辞書として返す。

## おわりに

今回は、FileSearchExplorerの1010〜1490行目を解説しました。

今回の範囲では、例外処理、通知、システム性能の検出、NTFS ACLによるアクセス制御、暗号鍵の管理、インデックスDBの暗号化・復号、ファイルのハッシュ計算などを扱いました。

特に、暗号化DBが存在する場合の復号失敗時の扱いや、ACLマーカーによる処理省略については、実装上の注意点があります。権限設定や暗号化処理は、コードを書くだけで完全な安全性が保証されるわけではありません。

今後も、コードの動作だけでなく、失敗時の挙動や安全性についても確認しながら解説を続けていきたいと思います。

最後まで読んでいただき、ありがとうございました！
