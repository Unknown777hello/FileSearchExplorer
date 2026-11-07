タイトル：【FileSearchExplorer解説 #9】1行ずつ解説：1491-3456行目
# はじめに
こんにちはUnknown777です。
今回は、FileSearchExplorerのコードを1行ずつ解説します。 今回の範囲は1493〜3456行目です。前回の1010〜1490行目の解説も読んでくださった皆さん、本当にありがとうございます。今回のブログも頑張って作りました。
ひさしぶりの解説回ですね。
今回は、シリーズ最大の2000行解説になっています。
また、今回のブログの「本編」はAIで作っています。
できるだけ自然な文章になるようにUnknown777が書き直しているので、読みやすくなっていると思います。
では、本編に入る前にこのブログの書く方法を記しておきます。
### ブログ記述方法
AIに1行ずつ詳しく説明してもらう(AIのほうが詳しい)(ググりながらコード書いてるレベルなので)
それを文章作成アプリで移すついでに文章を校正、ググって訂正をする
それをブログサイトなどに張り付けて公開するという手順です。
# 本編
## 第1章：テキスト判定とプレビュー
    • def _detect_text_encoding(sample_bytes):：先頭バイト列から文字コードを判定する関数。戻り値はエンコーディング名かNone。
    • """修正5: テキストの文字コードをBOM等から判定する。：UTF-8 BOM付き・UTF-16(LE/BE)対応を追加した修正の説明。従来はUTF-8/CP932のみだった。
    • UTF-16のテキストはASCII範囲の文字ではバイト列に大量の：UTF-16だとASCII文字でも2バイトに1つNULが入る。
    • \\x00 (NUL) を含むため、_looks_binary() の単純なNULバイト判定を先に通すと：ここが今回の範囲で一番大事な順序の話。NUL判定を先にやるとUTF-16が全部バイナリ扱いになる。
    • 誤ってバイナリ扱いされてしまう。そのため呼び出し側では、この関数による：だから呼び出し側では必ずこの判定を_looks_binary()より先に行うこと、という呼び出し順の契約。
    • 判定を _looks_binary() より先に行うこと。：この順序を守らないと修正5の意味がなくなる。
    • 戻り値: 判定できたエンコーディング名の文字列、判定できなければNone。"""：docstringの締め。
    • if sample_bytes.startswith(b'\xef\xbb\xbf'):：UTF-8 BOMの3バイトをチェック。
    • return 'utf-8-sig'：BOM付きUTF-8はutf-8-sigで読むとBOMが除去される。
    • if sample_bytes.startswith(b'\xff\xfe'):：UTF-16 LEのBOM。
    • return 'utf-16-le'：LEとして返す。
    • if sample_bytes.startswith(b'\xfe\xff'):：UTF-16 BEのBOM。
    • return 'utf-16-be'：BEとして返す。
    • # BOMが無いUTF-16の簡易ヒューリスティック: ASCII主体のテキストであれば：BOMなしの救済策。あくまで簡易ヒューリスティックで、確実な判定ではない。
    • # 偶数/奇数いずれかのバイト位置にNULが偏って高頻度で出現する。：ASCII主体のUTF-16なら、片側のバイト位置だけNULが偏るという性質を使う。
    • if len(sample_bytes) >= 8:：短すぎるサンプルでは統計が信用できないので、8バイト未満はヒューリスティックをしない。
    • even = sample_bytes[0::2]：偶数位置のバイトだけ抜き出す。
    • odd = sample_bytes[1::2]：奇数位置のバイトだけ抜き出す。
    • even_null_ratio = even.count(0) / len(even) if even else 0：偶数側のNUL比率。
    • odd_null_ratio = odd.count(0) / len(odd) if odd else 0：奇数側のNUL比率。
    • if odd_null_ratio > 0.4 and even_null_ratio < 0.1:：奇数側にNULが40%超、偶数側は10%未満ならLEらしい。
    • return 'utf-16-le'：LEとして返す。
    • if even_null_ratio > 0.4 and odd_null_ratio < 0.1:：逆パターンならBEらしい。
    • return 'utf-16-be'：BEとして返す。
    • return None：ここまでで分からなければNone。呼び出し側は後でUTF-8/CP932を試す。
    • def _looks_binary(sample_bytes):：バイナリらしいかを簡易判定する関数。
    • """先頭バイト列を見て、テキストではなくバイナリらしいかを簡易判定する。：docstring。あくまで簡易判定。
    • NULバイトが含まれる場合や、制御文字の比率が高い場合はバイナリとみなす。"""：判定基準の説明。
    • if not sample_bytes:：空データはテキスト扱い。
    • return False：空はバイナリではない。
    • if b"\x00" in sample_bytes:：NULが1つでもあればバイナリ。
    • return True：ここで即バイナリ判定。だからUTF-16はこの前に_detect_text_encoding()を通す必要がある。
    • text_chars = bytes(range(0x20, 0x7f)) + b"\n\r\t\b\f"：テキストとみなすバイトの集合。印字可能ASCIIと改行系の制御文字。
    • non_text = sum(1 for b in sample_bytes if b not in text_chars and b < 0x80)：0x80未満でテキスト集合に入っていないバイトを数える。0x80以上はここではカウントしないため、日本語のUTF-8がそのままバイナリ扱いされにくい作り。
    • return non_text / len(sample_bytes) > 0.30：30%超が非テキストならバイナリ。
    • def read_text_preview(path, max_chars=3000):：テキストプレビューを最大3000文字読む関数。
    • try:：ファイル読み込みの例外に備える。
    • with open(path, "rb") as f:：まずバイナリで開く。判定用サンプルを読むため。
    • sample = f.read(4096)：先頭4096バイトだけ読む。全文を読まないことで大きいファイルでも軽い。
    • except OSError:：開けない場合はプレビューなし。
    • return None：OSError時はNone。
    • # 修正5: UTF-16はNULバイトを含むため、バイナリ判定(_looks_binary)より先に：ここが修正5の呼び出し側の実装。
    • # BOM/ヒューリスティックによる文字コード判定を行う：順序のコメント。
    • detected_enc = _detect_text_encoding(sample)：まずBOM/ヒューリスティックで判定。
    • if detected_enc:：判定できた場合。
    • try:：読み込みの例外に備える。
    • with open(path, "r", encoding=detected_enc, errors="replace") as f:：判定したエンコーディングで読む。errors="replace"なので読めない文字は置換文字になる。
    • return f.read(max_chars)：最大文字数まで返す。
    • except OSError:：開き直しに失敗した場合。
    • return None：プレビューなし。
    • if _looks_binary(sample):：エンコーディングが分からなかった場合に、ここで初めてバイナリ判定。
    • return None：バイナリならプレビューなし。
    • for enc in ('utf-8', 'cp932'):：日本語Windowsで多い2つを順に試す。
    • try:：デコードの例外に備える。
    • with open(path, "r", encoding=enc, errors="strict") as f:：strictで厳密にデコードする。ここで失敗したら次のエンコーディングへ回すため。
    • return f.read(max_chars)：読めたら即返す。
    • except UnicodeDecodeError:：デコード失敗は次のエンコーディングへ。
    • continue：次のエンコーディングを試す。
    • except OSError:：開けない場合。
    • return None：プレビューなし。
    • try:：最後のフォールバック。
    • with open(path, "r", encoding="utf-8", errors="replace") as f:：どうしても読めない場合はUTF-8のreplaceで無理やり読む。
    • return f.read(max_chars)：置換しながら返す。
    • except OSError:：それでも開けない場合。
    • return None：プレビューなし。
    • def _load_pil_thumbnail_data(path, max_size=220):：Pillowでサムネイル用の画像データを読み込む関数。Tkinterを呼ばないのがポイント。
    • """修正(UIフリーズ対策): Pillowによるデコード・リサイズはTkinterを一切：UIフリーズ対策の修正。
    • 呼び出さないため、バックグラウンドスレッドから安全に実行できる。：Pillowの処理はスレッドセーフ寄りなので、バックグラウンドでやれる。
    • 大きな画像のデコードはCPU/IOを消費しUIスレッドを固まらせうるため、：大きい画像をUIスレッドで開くと固まる。
    • この関数を別スレッドで呼び出し、結果(PIL Imageオブジェクト)をメイン：別スレッドでデコードだけ済ませる。
    • スレッド側で ImageTk.PhotoImage() に変換する(Tk呼び出しは必ずメイン：Tk呼び出しだけは必ずメインスレッドで、という役割分担。
    • スレッドで行うこと)。"""：docstringの締め。
    • pil = _lazy_import_pil()：Pillowを遅延インポートする。
    • if pil is None:：Pillowがない場合。
    • return None：サムネイルなし。
    • Image, _ImageTk, UnidentifiedImageError = pil：使うものだけ取り出す。
    • try:：画像読み込みの例外に備える。
    • img = Image.open(path)：ヘッダだけ読む段階。ここではまだ本文はデコードされていない。
    • # 修正6: Image.open()自体はヘッダのみ読み、本文の破損は遅延評価される：ここが修正6の核心。
    • # ことがあるため、load()でこの時点でデコードを強制して壊れた画像を検出する：load()しないと壊れた画像が後で初めて発覚する。
    • img.load()：ここでデコードを強制する。
    • img.thumbnail((max_size, max_size))：アスペクト比を保って最大サイズ内に収める。
    • return img：PIL Imageを返す。まだPhotoImageにはしない。
    • except UnidentifiedImageError as exc:：画像として認識できない場合。
    • log_error("プレビュー(Pillow)", f"{path}: {exc}")：ログに残す。
    • raise ValueError(：呼び出し側に分かりやすいエラーとして投げ直す。
    • "画像として認識できませんでした（ファイルが壊れているか、非対応の形式の可能性があります）"：ユーザー向けメッセージ。
    • ) from exc：元の例外を残して再送出する。
    • except OSError as exc:：切り詰められたJPEGなど、破損データはOSErrorになることがある。
    • # 破損した画像データ(切り詰められたJPEG等)はOSErrorとして送出されうる：コメント。
    • log_error("プレビュー(Pillow)", f"{path}: {exc}")：ログに残す。
    • raise ValueError(f"画像の読み込みに失敗しました（ファイルが壊れている可能性があります）: {exc}") from exc：同様に分かりやすいエラーへ変換する。
    • def load_preview_thumbnail(path, max_size=220):：メインスレッド専用のサムネイル作成関数。
    • """メインスレッド専用。ImageTk.PhotoImage()/tk.PhotoImage()はいずれも：この関数はメインスレッドからしか呼んではいけない。
    • Tk呼び出しのため、必ずメインスレッドから呼び出すこと。"""：Tk呼び出しの制約。
    • pil = _lazy_import_pil()：Pillowの有無を確認。
    • if pil is not None:：Pillowがある場合。
    • _Image, ImageTk, _UnidentifiedImageError = pil：ImageTkを取り出す。
    • img = _load_pil_thumbnail_data(path, max_size)：先ほどのバックグラウンド対応の関数でデコードする。
    • return ImageTk.PhotoImage(img)：ここで初めてTk用の画像に変換する。ここがメインスレッド限定の理由。
    • ext = os.path.splitext(path)[1].lower()：Pillowがない場合のフォールバック用に拡張子を見る。
    • if ext not in ('.png', '.gif', '.pgm', '.ppm'):：tk.PhotoImageで読めるのはこのあたりだけ。
    • raise ValueError("この画像形式のプレビューには Pillow が必要です\n(pip install Pillow)")：それ以外はPillow必須としてエラーにする。
    • try:：tk.PhotoImageの例外に備える。
    • photo = tk.PhotoImage(file=path)：Tk標準で画像を読む。
    • except tk.TclError as exc:：壊れた画像だとTclErrorになる。
    • # 修正6: Pillow未導入時のtk.PhotoImageも壊れた画像でTclErrorになりうるため：Pillowなしの経路も同じく修正6の対象。
    • # 同様に安全に処理する：コメント。
    • log_error("プレビュー(Tk PhotoImage)", f"{path}: {exc}")：ログに残す。
    • raise ValueError(f"画像の読み込みに失敗しました（ファイルが壊れている可能性があります）: {exc}") from exc：分かりやすいエラーへ変換。
    • factor = max(1, max(photo.width(), photo.height()) // max_size)：縮小率を計算する。1未満にはしない。
    • if factor > 1:：縮小が必要な場合。
    • photo = photo.subsample(factor, factor)：間引いて縮小する。Pillowほど滑らかではないが、フォールバックとしては十分。
    • return photo：PhotoImageを返す。
## 第2章：進捗表示・実行確認・パスを開く
    • def format_progress(current_size, total_size, count):：進捗文言を作る関数。
    • current_text = human_size(current_size)：バイト数を人間向けの単位へ変換。
    • return f"{current_text} 処理済み（{count} 件）"：現在の処理量と件数を表示する。なお、total_sizeは引数として受け取っているが、この関数内では使用されていない。
    • def needs_execution_confirm(path):：実行前に確認ダイアログが必要な拡張子かを判定する。
    • if not path or os.path.isdir(path):：パスが空かフォルダなら確認不要。
    • return False：フォルダは実行対象ではない。
    • ext = os.path.splitext(path)[1].lower().lstrip(".")：拡張子を小文字化し、ドットを外す。
    • return ext in EXECUTION_CONFIRM_EXTENSIONS：実行確認対象の集合に入っているかで返す。
    • def _validate_path_for_external_command(path):：外部コマンドへ渡す直前のパス検証。セキュリティの多重防御。
    • """修正(セキュリティ・多重防御・v0.7.8): open_path()がexplorer/open/xdg-open・：対象はopen_path()が呼ぶ外部コマンド。
    • os.startfile()を呼び出す直前の入力検証。：呼ぶ直前でチェックする。
    • 監査の結果、これらの呼び出しはすべてリスト形式の引数で渡されており：shell=Trueは使っていないという監査結果。
    • (shell=Trueは本アプリ全体で未使用)、OS側でシェル解釈が行われないため：だからシェル経由のコマンドインジェクション経路はそもそも存在しない、という前提。
    • シェル経由のコマンドインジェクション経路はそもそも存在しない。：ここが主防御。
    • ここでは念のための多重防御として、文字列型でない値やNULバイトを含む：この関数はあくまで念のための二重防御。
    • 異常な入力(OS API層で予期しないエラーやパス切り詰めを起こしうる)のみを：弾くのは異常入力だけ。
    • 弾く。正常なファイル/フォルダパスの挙動には一切影響しない。"""：正常パスには影響しない、という設計。
    • if not isinstance(path, str) or not path:：文字列でないか空なら不正。
    • return False：不正として返す。
    • if "\x00" in path:：NULバイトが入っているとOS APIで切り詰めなどが起きうる。
    • return False：不正として返す。
    • return True：それ以外はOK。
    • def open_path(path, reveal_only=False):：ファイル/フォルダをOSの標準の方法で開く関数。reveal_only=Trueなら親フォルダで選択状態にする。
    • if not _validate_path_for_external_command(path):：まずパス検証。
    • log_error("開く操作", "不正なパスが渡されたため処理を中止しました")：不正パスはログに残す。
    • messagebox.showerror("エラー", "無効なパスのため開けませんでした。")：ユーザーにも伝える。
    • return：開く処理を中止する。
    • # 修正(v0.7.9・権限不足/存在しないファイルへの処理改善): 検索結果が：検索結果表示後にファイルが移動・削除されるレースへの対策。
    • # 表示されてからOSに開く操作を依頼するまでの間に、対象が他プロセス等で：表示と実行の間に状態が変わりうる。
    • # 移動・削除されている、あるいはシンボリックリンクの実体が既に無くなって：壊れたリンクも含む。
    • # いる(壊れたリンク)ケースがある。os.path.exists()はリンク先の実体を：exists()はリンク先の実体を見る。
    • # 確認するため、壊れたリンクも「存在しない」として検出できる。事前に：だから壊れたリンクもここで検出できる。
    • # 検出して分かりやすいメッセージを出すことで、OS側の分かりにくいエラーで：OS任せだと分かりにくいエラーになりやすい。
    • # 処理が止まってしまうのを防ぐ(reveal_onlyの場合は親フォルダの存在確認、：reveal_onlyなら親フォルダを確認する。
    • # 通常openの場合は対象自体の存在確認を行う)。：通常は対象自体を確認する。
    • check_target = os.path.dirname(path) if reveal_only else path：確認対象を切り替える。
    • if not os.path.exists(check_target):：存在しない場合。
    • log_error("開く操作", f"{path}: 対象が見つかりません（移動・削除、または壊れたリンクの可能性）")：理由をログに残す。
    • messagebox.showerror(：ユーザーへ表示。
    • "エラー", "対象が見つかりませんでした。\n移動または削除された可能性があります。")：分かりやすい文言。
    • return：ここで中止する。
    • target = os.path.dirname(path) if reveal_only else path：実際に開く対象。reveal_onlyでもxdg-openなどでは親フォルダを開く形になる。
    • try:：OSへの依頼を例外処理で囲む。
    • if sys.platform.startswith("win"):：Windows分岐。
    • if reveal_only:：フォルダで選択表示する場合。
    • subprocess.run(["explorer", "/select,", path])：explorer /select,で選択状態で開く。リスト形式なのでシェル解釈はされない。
    • return：Windowsのreveal_onlyはここで終わり。
    • os.startfile(target)：通常は関連付けアプリで開く。
    • return：Windows分岐の終わり。
    • if sys.platform == "darwin":：macOS分岐。
    • if reveal_only:：Finderで表示する場合。
    • result = subprocess.run(["open", "-R", path], capture_output=True, text=True)：open -RでFinderで表示。
    • else:：通常オープン。
    • result = subprocess.run(["open", target], capture_output=True, text=True)：openで開く。
    • if result.returncode != 0:：失敗した場合。
    • msg = result.stderr.strip()：標準エラーからメッセージを取る。
    • if not msg:：メッセージが空なら。
    • msg = f"open がエラー終了しました（コード: {result.returncode}）"：終了コード入りのメッセージを作る。
    • raise RuntimeError(msg)：例外として後段の共通処理へ渡す。
    • return：macOS分岐の終わり。
    • result = subprocess.run(["xdg-open", target], capture_output=True, text=True)：Linuxはxdg-open。
    • if result.returncode != 0:：失敗した場合。
    • msg = result.stderr.strip()：標準エラーを取る。
    • if not msg:：空なら。
    • msg = f"xdg-open がエラー終了しました（コード: {result.returncode}）"：終了コード入りのメッセージ。
    • raise RuntimeError(msg)：例外として後段へ。
    • return：Linux分岐の終わり。
    • except Exception as exc:：OSへの依頼で起きた例外をまとめて受ける。
    • log_error("開く操作", f"{path}: {exc}")：ログに残す。
    • messagebox.showerror("エラー", f"開けませんでした:\n{exc}")：ユーザーへ伝える。
## 第3章：隠し/システムファイルと保護フォルダ
    • def is_hidden_or_system(full_path, name=None):：隠し属性とシステム属性をまとめて判定する旧来の関数。v0.9.6で個別判定用の関数が追加された後も、ソースコード内には定義が残っている。
    • if not name:：名前が渡されていない場合。
    • name = os.path.basename(full_path)：パスから名前を取る。
    • try:：属性取得の例外に備える。
    • if sys.platform.startswith("win"):：Windows分岐。
    • import ctypes：Windows API呼び出し用。
    • FILE_ATTRIBUTE_HIDDEN = 2：隠し属性の値。
    • FILE_ATTRIBUTE_SYSTEM = 4：システム属性の値。
    • attrs = ctypes.windll.kernel32.GetFileAttributesW(str(full_path))：属性を取得する。
    • if attrs == -1:：取得失敗。
    • return name.startswith(".")：失敗時はドット始まりで代用する。
    • return bool(attrs & (FILE_ATTRIBUTE_HIDDEN | FILE_ATTRIBUTE_SYSTEM))：どちらかのビットがあればTrue。
    • return name.startswith(".")：Windows以外はドット始まりで判定。
    • except Exception as exc:：例外時の処理。
    • # 修正1: 頻繁に呼ばれる関数のため大量ログを避けたいが、原因不明の不具合を：頻繁に呼ばれるのでログが爆発しやすい。
    • # 追跡できるようログには残しつつ、フォールバック動作は維持する：ログは残すが、動作は止めない。
    • log_error("隠しファイル判定", f"{full_path}: {exc}")：ログに残す。
    • return name.startswith(".")：フォールバックでドット判定。
    • def is_hidden(full_path, name=None):：隠し属性だけを判定する関数。v0.9.6で分離された。
    • """修正(v0.9.6・隠し/システムファイルの個別検索対応): 隠し属性のみを判定する。：分離の理由。
    • 従来のis_hidden_or_system()は隠し・システムを1つにまとめていたが、：従来は一括だった。
    • 「隠しファイルのみ含める」「システムファイルのみ含める」を個別に：個別に切り替えたい。
    • 切り替えられるようにするため分離する。"""：docstringの締め。
    • if not name:：名前がなければパスから取る。
    • name = os.path.basename(full_path)：同上。
    • try:：属性取得の例外に備える。
    • if sys.platform.startswith("win"):：Windows分岐。
    • import ctypes：同上。
    • FILE_ATTRIBUTE_HIDDEN = 2：隠し属性。
    • attrs = ctypes.windll.kernel32.GetFileAttributesW(str(full_path))：属性を取得。
    • if attrs == -1:：取得失敗。
    • return name.startswith(".")：ドットで代用。
    • return bool(attrs & FILE_ATTRIBUTE_HIDDEN)：隠しビットだけ見る。
    • return name.startswith(".")：Windows以外はドット判定。
    • except Exception as exc:：例外時。
    • log_error("隠しファイル判定", f"{full_path}: {exc}")：ログに残す。
    • return name.startswith(".")：フォールバック。
    • def is_system_file(full_path):：システム属性だけを判定する関数。
    • """修正(v0.9.6・隠し/システムファイルの個別検索対応): システム属性のみを判定する。：分離のもう片方。
    • Windows以外ではシステム属性の概念が無いため常にFalseを返す。"""：Windows以外は常にFalse。
    • try:：属性取得の例外に備える。
    • if sys.platform.startswith("win"):：Windows分岐。
    • import ctypes：同上。
    • FILE_ATTRIBUTE_SYSTEM = 4：システム属性。
    • attrs = ctypes.windll.kernel32.GetFileAttributesW(str(full_path))：属性を取得。
    • if attrs == -1:：取得失敗。
    • return False：失敗時はシステム扱いしない。ここはis_hidden()と違い、ドットでの代用はしない。
    • return bool(attrs & FILE_ATTRIBUTE_SYSTEM)：システムビットだけ見る。
    • return False：Windows以外は常にFalse。
    • except Exception as exc:：例外時。
    • log_error("システムファイル判定", f"{full_path}: {exc}")：ログに残す。
    • return False：例外時はシステム扱いしない。
    • def get_protected_system_dirs():：保護対象のシステムフォルダ一覧を作る関数。
    • candidates = []：候補リスト。
    • for var in ("WINDIR", "SYSTEMROOT", "PROGRAMFILES", "PROGRAMFILES(X86)", "PROGRAMDATA"):：環境変数から候補を集める。
    • val = os.environ.get(var)：環境変数を取得。
    • if val:：値がある場合。
    • candidates.append(val)：候補へ追加。
    • candidates.extend([：環境変数がなくても分かるよう、代表的なパスも追加する。
    • r"C:\Windows",：Windows本体。
    • r"C:\Program Files",：プログラム本体。
    • r"C:\Program Files (x86)",：32ビット用。
    • r"C:\ProgramData",：共有データ。
    • ])：候補の追加終わり。
    • normalized = set()：正規化済みの集合。
    • for c in candidates:：候補ごとに正規化する。
    • try:：正規化の例外に備える。
    • normalized.add(os.path.normcase(os.path.normpath(c)))：大文字小文字と区切りを正規化して集合へ。
    • except Exception as exc:：正規化失敗時。
    • # 修正1: 個々の候補の正規化失敗は致命的ではないためスキップして継続するが、：1つの候補の失敗で全体を止めない。
    • # 原因調査ができるようログには残す：ログは残す。
    • log_error("システム保護フォルダ判定", f"{c}: {exc}")：ログに残す。
    • continue：次の候補へ。
    • return normalized：正規化済みの集合を返す。
    • _EXTENDED_PATH_PREFIXES = ("\\\\?\\unc\\", "\\\\?\\")：Windowsの長いパス表記のプレフィックス。\\?\UNC\と\\?\の2つ。
    • def _strip_extended_path_prefix(norm_path):：比較用に長いパス表記を通常表記へ揃える関数。
    • """修正(v0.7.9・パス正規化の強化): Windowsの長いパス表記：長いパス表記の問題。
    • (\\\\?\\C:\\... や \\\\?\\UNC\\server\\share\\...)は通常のパスと：同じ実体でも表記が違う。
    • 実体は同じだが、os.path.normpath()ではこのプレフィックスが除去されない。：normpath()だけでは揃わない。
    • このため、同じフォルダを指していても表記の違いだけで：表記違いだけで判定をすり抜ける恐れがあった。
    • is_path_within()による配下判定・保護フォルダ判定をすり抜けてしまう：保護判定のすり抜けは危険。
    • 恐れがあった。ここで比較用に正規化済み(normcase済み)の文字列から：比較前にここで揃える。
    • プレフィックスを取り除き、通常表記と同一視できるようにする。：通常表記と同一視する。
    • UNCプレフィックス(\\\\?\\UNC\\)は除去後に先頭へ \\\\ を補って：UNCは除去後に\\を補う。
    • 通常のUNCパス表記(\\\\server\\share\\...)に揃える。"""：通常のUNC表記へ揃える。
    • for prefix in _EXTENDED_PATH_PREFIXES:：2つのプレフィックスを順に試す。UNCの方を先に試すのがポイント。逆だと\\?\だけ取れてUNCの判別が崩れる。
    • if norm_path.startswith(prefix):：プレフィックスに一致した場合。
    • rest = norm_path[len(prefix):]：プレフィックス以降を取る。
    • if prefix.endswith("unc\\"):：UNCプレフィックスだった場合。
    • return "\\\\" + rest：先頭に\\を補って通常のUNC表記へ。
    • return rest：通常の長いパス表記ならプレフィックスを外しただけを返す。
    • return norm_path：どちらにも当てはまらなければそのまま返す。
    • def is_path_within(path_norm, base_norm):：正規化済みパスがbase自身または配下かを判定する。
    • """正規化済みパスがbase_norm自身またはその配下かを判定する。：docstring。
    • base_normがドライブ直下(C:\\)やルート(/)など、normpathが末尾の：ドライブ直下は末尾に区切りが残る特殊ケース。
    • 区切り文字を残す特殊なパスの場合に単純な "base_norm + os.sep" 連結だと：単純連結だと区切りが二重になる。
    • 区切り文字が二重になり配下判定が常にFalseになってしまうため、：二重区切りだと配下判定が常に失敗する。
    • 末尾の区切り文字の有無を吸収してから判定する。"""：末尾の有無を吸収して判定する。
    • path_norm = _strip_extended_path_prefix(path_norm)：長いパス表記を揃える。
    • base_norm = _strip_extended_path_prefix(base_norm)：基準側も揃える。
    • if path_norm == base_norm:：同一パスなら配下扱い。
    • return True：同一ならTrue。
    • prefix = base_norm if base_norm.endswith(os.sep) else base_norm + os.sep：末尾に区切りがあればそのまま、なければ1つ補う。
    • return path_norm.startswith(prefix)：プレフィックス一致で配下判定。
    • def is_protected_system_dir(path):：システム保護フォルダ配下かを判定する。判定できないときは安全側に倒す。
    • """システム保護フォルダ(またはその配下)かどうかを判定する。：docstring。
    • 修正(v0.9.7・判定不能時は安全側=保護対象扱い): 従来はパスの正規化に失敗した場合：従来は失敗時にFalseを返していた。
    • Falseを返していたため、判定できない異常時に削除・移動・名前変更などの処理が：判定できないのに処理が続くと危険。
    • そのまま続行されてしまう恐れがあった。判定できない(例外・不正な入力)場合は：判定不能は危険側。
    • 「保護対象である」とみなしてTrueを返し、呼び出し側が操作を拒否するようにする：フェイルクローズ。
    • (フェイルクローズ)。"""：安全側に倒す方針。
    • # 修正(v0.7.9・シンボリックリンク対策): 従来はabspath()のみで正規化していたため、：シンボリックリンク対策。
    • # 実体はC:\Windows配下にあるフォルダへのシンボリックリンク/ジャンクションを：別名経由のすり抜け。
    • # 別名で経由された場合、保護対象のパス文字列と一致せず判定をすり抜けてしまう：文字列一致だけでは防げない。
    • # 恐れがあった。realpath()でシンボリックリンク/ジャンクションを解決した上で：realpath()で実体を解決する。
    • # 判定することで、別名経由でもシステムフォルダへの操作を確実に検出する。：別名経由でも検出する。
    • if not isinstance(path, str) or not path or "\x00" in path:：不正な入力は即保護対象。
    • log_error("システム保護フォルダ判定", "不正なパス入力のため保護対象として扱います")：ログに残す。
    • return True：不正入力は保護対象扱い。
    • try:：判定処理を例外処理で囲む。
    • candidates = {os.path.abspath(path), os.path.realpath(path)}：abspathとrealpathの両方を候補にする。片方だけではすり抜けるケースがあるため。
    • norm_candidates = {os.path.normcase(os.path.normpath(c)) for c in candidates}：両方を正規化する。
    • protected_dirs = get_protected_system_dirs()：保護対象一覧を取得。
    • if not protected_dirs:：一覧が空なら判定不能。
    • raise ValueError("保護対象フォルダの一覧を取得できませんでした")：例外として後段のフェイルクローズへ渡す。
    • for norm in norm_candidates:：候補ごとに。
    • for protected in protected_dirs:：保護対象ごとに。
    • if is_path_within(norm, protected):：配下判定。
    • return True：1つでも当てはまれば保護対象。
    • return False：どれにも当てはまらなければ保護対象ではない。
    • except Exception as exc:：判定中の例外。
    • # 判定できない場合は安全側(保護対象=True)に倒し、操作を拒否させる。：フェイルクローズのコメント。
    • log_error("システム保護フォルダ判定", f"{path}: {exc}（判定不能のため保護対象として扱います）")：ログに残す。
    • return True：判定不能は保護対象扱い。
## 第4章：危険操作の確認ダイアログ
    • def play_warning_sound():：警告音を鳴らす関数。
    • if not WINSOUND_AVAILABLE:：winsoundが使えない環境なら何もしない。
    • return：早期リターン。
    • try:：再生の例外に備える。
    • winsound.MessageBeep(winsound.MB_ICONHAND)：手アイコンの警告音を鳴らす。
    • except Exception as exc:：再生失敗時。
    • log_error("警告音", str(exc))：ログに残す。音が鳴らなくても処理は止めない。
    • def show_danger_dialog(parent, title, headline, message, listing=None, confirm_text="実行する",：危険操作の確認ダイアログ。赤い厳重警戒ウィンドウ。
    • double_confirm=False, second_message=None):：double_confirm=Trueなら2段階確認になる。
    • """修正(v0.9.6・ファイル操作の危険操作確認): ゴミ箱移動・完全削除のような：対象はゴミ箱移動・完全削除などの危険操作。
    • 危険な操作の前に表示する、赤い厳重警戒ウインドウ。double_confirm=Trueの：赤いウィンドウで目立たせる。
    • 場合、1回目の「実行する」後にもう一段階、念押しの確認ダイアログを：2段階確認の説明。
    • 表示する(完全削除用。2回OKを押さないと実行できない)。：2回OKしないと実行できない。
    • 戻り値: 実行してよいか(bool)"""：戻り値は実行可否。
    • play_warning_sound()：表示前に警告音を鳴らす。
    • dialog = tk.Toplevel(parent)：親の上にダイアログを作る。
    • dialog.title(title)：タイトルを設定。
    • dialog.transient(parent)：親に紐付ける。
    • dialog.grab_set()：モーダル化する。他のウィンドウを触れなくする。
    • dialog.resizable(False, False)：サイズ変更不可。
    • dialog.configure(bg="#b71c1c")：背景を濃い赤にする。
    • result = {"ok": False}：結果を辞書で保持する。クロージャで書き換えるため。
    • frame = tk.Frame(dialog, bg="#b71c1c", padx=18, pady=16)：内側フレームも赤で統一。
    • frame.pack(fill="both", expand=True)：フレームを広げる。
    • tk.Label(frame, text=headline, bg="#b71c1c", fg="white", font=("", 13, "bold")).pack(anchor="w")：見出しを白の太字で表示。
    • tk.Label(frame, text=message, bg="#b71c1c", fg="white", justify="left",：メッセージを表示。
    • wraplength=520, font=("", 10)).pack(anchor="w", pady=(8, 8))：520pxで折り返す。
    • if listing:：対象ファイルの一覧がある場合。
    • list_box = tk.Text(frame, height=min(8, max(3, len(listing))), width=68,：一覧表示用のテキストボックス。高さは3〜8行に制限。
    • bg="#ffebee", fg="#b71c1c", relief="solid", borderwidth=1, wrap="none")：薄い赤背景に濃い赤文字。
    • list_box.insert("1.0", "\n".join(listing[:15]) + ("\n..." if len(listing) > 15 else ""))：最大15件まで表示し、それ以上は...で省略。
    • list_box.configure(state="disabled")：編集不可にする。
    • list_box.pack(fill="both", expand=True, pady=(0, 14))：一覧を配置。
    • btn_frame = tk.Frame(frame, bg="#b71c1c")：ボタン用フレーム。
    • btn_frame.pack(fill="x")：横いっぱいに広げる。
    • def on_yes():：実行ボタンの処理。
    • result["ok"] = True：実行OKを記録。
    • dialog.destroy()：ダイアログを閉じる。
    • def on_no():：キャンセルボタンの処理。
    • dialog.destroy()：ダイアログを閉じる。resultはFalseのまま。
    • ttk.Button(btn_frame, text="キャンセル", command=on_no).pack(side="right")：キャンセルボタンを右に配置。
    • yes_btn = tk.Button(btn_frame, text=confirm_text, command=on_yes,：実行ボタン。
    • bg="#d32f2f", fg="white", activebackground="#8e0000",：赤系で目立たせる。
    • activeforeground="white", font=("", 10, "bold"),：白の太字。
    • relief="flat", padx=10, pady=4)：フラットな見た目。
    • yes_btn.pack(side="right", padx=(0, 8))：キャンセルの左に配置。
    • dialog.protocol("WM_DELETE_WINDOW", on_no)：×ボタンでもキャンセル扱い。
    • dialog.bind("<Escape>", lambda e: on_no())：Escでもキャンセル。
    • dialog.update_idletasks()：サイズ確定のために更新する。
    • try:：中央配置の例外に備える。
    • x = parent.winfo_x() + (parent.winfo_width() - dialog.winfo_reqwidth()) // 2：親の中央のX座標。
    • y = parent.winfo_y() + (parent.winfo_height() - dialog.winfo_reqheight()) // 2：親の中央のY座標。
    • dialog.geometry(f"+{max(x, 0)}+{max(y, 0)}")：負の座標にならないようmaxで調整。
    • except tk.TclError:：座標取得に失敗した場合。
    • pass：中央配置を諦める。
    • dialog.wait_window()：ダイアログが閉じるまで待つ。
    • if not result["ok"]:：1回目でキャンセルされた場合。
    • return False：実行しない。
    • if not double_confirm:：2段階確認でない場合。
    • return True：1回目でOKなら実行する。
    • # 完全削除など特に危険な操作は、1回目のOKの後にもう一段階の：2段階確認のコメント。
    • # 念押し確認を行う(限界警戒色ウインドウ・2回OKを押さないと実行できない)。：2回OKが必要。
    • play_warning_sound()：2回目でも警告音を鳴らす。
    • dialog2 = tk.Toplevel(parent)：2回目のダイアログ。
    • dialog2.title("最終確認")：タイトルは最終確認。
    • dialog2.transient(parent)：親に紐付ける。
    • dialog2.grab_set()：モーダル化。
    • dialog2.resizable(False, False)：サイズ変更不可。
    • dialog2.configure(bg="#7a0000")：1回目よりさらに濃い赤。
    • result2 = {"ok": False}：2回目の結果。
    • frame2 = tk.Frame(dialog2, bg="#7a0000", padx=18, pady=16)：内側フレームも濃い赤。
    • frame2.pack(fill="both", expand=True)：広げる。
    • tk.Label(frame2, text="⚠⚠ 本当に実行しますか？ ⚠⚠", bg="#7a0000", fg="white",：2回目の見出し。
    • font=("", 14, "bold")).pack(anchor="w")：大きめの太字。
    • tk.Label(frame2, text=second_message or "この操作は取り消せません。もう一度確認してください。",：2回目のメッセージ。指定がなければ既定文。
    • bg="#7a0000", fg="white", justify="left", wraplength=480,：480pxで折り返す。
    • font=("", 10, "bold")).pack(anchor="w", pady=(10, 14))：太字で表示。
    • btn_frame2 = tk.Frame(frame2, bg="#7a0000")：ボタン用フレーム。
    • btn_frame2.pack(fill="x")：横いっぱい。
    • def on_yes2():：2回目の実行ボタン。
    • result2["ok"] = True：OKを記録。
    • dialog2.destroy()：閉じる。
    • def on_no2():：2回目のキャンセル。
    • dialog2.destroy()：閉じる。
    • ttk.Button(btn_frame2, text="キャンセル", command=on_no2).pack(side="right")：キャンセルボタン。
    • yes_btn2 = tk.Button(btn_frame2, text="完全に削除する", command=on_yes2,：2回目の実行ボタンは文言が固定で「完全に削除する」。
    • bg="#b71c1c", fg="white", activebackground="#5a0000",：濃い赤系。
    • activeforeground="white", font=("", 10, "bold"),：白の太字。
    • relief="flat", padx=10, pady=4)：フラット。
    • yes_btn2.pack(side="right", padx=(0, 8))：キャンセルの左に配置。
    • dialog2.protocol("WM_DELETE_WINDOW", on_no2)：×でもキャンセル。
    • dialog2.bind("<Escape>", lambda e: on_no2())：Escでもキャンセル。
    • dialog2.update_idletasks()：サイズ確定。
    • try:：中央配置。
    • x = parent.winfo_x() + (parent.winfo_width() - dialog2.winfo_reqwidth()) // 2：中央X。
    • y = parent.winfo_y() + (parent.winfo_height() - dialog2.winfo_reqheight()) // 2：中央Y。
    • dialog2.geometry(f"+{max(x, 0)}+{max(y, 0)}")：負にならないよう調整。
    • except tk.TclError:：失敗時。
    • pass：中央配置を諦める。
    • dialog2.wait_window()：閉じるまで待つ。
    • return result2["ok"]：2回目の結果を返す。
## 第5章：検索条件のパースとあいまい検索
    • def normalize_ext_filter(ext_csv_or_iterable):：拡張子フィルタを正規化する関数。
    • if ext_csv_or_iterable is None:：未指定の場合。
    • return set()：空集合を返す。
    • if isinstance(ext_csv_or_iterable, str):：文字列ならカンマで分割。
    • items = ext_csv_or_iterable.split(",")：カンマ分割。
    • else:：文字列以外はイテラブルとしてそのまま使う。
    • items = ext_csv_or_iterable：そのまま使う。
    • return {e.strip().lower().lstrip(".") for e in items if e and e.strip()}：空白除去・小文字化・ドット除去して集合化。空要素は除外。
    • _BOOL_OP_RE = re.compile(r'\s+(AND|OR|NOT)\s+')：ブール演算子を検出する正規表現。AND/OR/NOTは大文字のみが対象で、小文字のandなどは演算子として扱われない点に注意。
    • FUZZY_MATCH_THRESHOLD = 0.72：あいまい一致の閾値。SequenceMatcherのratio()がこの以上なら一致。
    • def parse_boolean_query(keyword):：キーワードをブール演算子で分割する関数。
    • if not keyword or not keyword.strip():：空なら。
    • return []：空リストを返す。
    • # 修正(v0.9.6・フレーズ検索): 二重引用符で囲んだ部分はフレーズとして扱い、：フレーズ検索の修正。
    • # 中に含まれる AND/OR/NOT は演算子ではなく文字列そのものとして検索する。：フレーズ内の演算子は演算子ではない。
    • phrases = []：フレーズを退避するリスト。
    • def _mask(m):：フレーズをマスクする内部関数。
    • phrases.append(m.group(1))：引用符の中身を保存。
    • return f"\x00{len(phrases) - 1}\x00"：\x00数字\x00というプレースホルダへ置き換える。\x00は通常のキーワードに入りにくいため選ばれている。
    • masked = re.sub(r'"([^"]*)"', _mask, keyword.strip())：二重引用符で囲まれた部分をすべてマスクする。
    • def _restore(text):：マスクを戻す内部関数。
    • return re.sub(r"\x00(\d+)\x00", lambda m: phrases[int(m.group(1))], text).strip()：プレースホルダを元のフレーズへ戻す。
    • parts = _BOOL_OP_RE.split(masked)：演算子で分割する。splitは演算子自体も結果に含む。
    • terms = [(None, _restore(parts[0]))]：最初の項は演算子なし。
    • i = 1：2番目の要素から処理する。
    • while i < len(parts) - 1:：演算子と項のペアが残っている間。
    • terms.append((parts[i], _restore(parts[i + 1])))：演算子と次の項を追加。
    • i += 2：2つ進める。
    • return [(op, term) for op, term in terms if term]：空の項を除いて返す。
    • def strip_phrase_quotes(keyword):：内容検索用の引用符除去。
    • """内容検索用: 全体が二重引用符で囲まれている場合は引用符を外す。"""：全体が引用符の場合だけ外す。
    • if keyword and len(keyword) >= 2 and keyword.startswith('"') and keyword.endswith('"'):：両端が二重引用符か確認。
    • return keyword[1:-1]：中身だけ返す。
    • return keyword：それ以外はそのまま返す。
    • def fuzzy_name_match(name, keyword, ignore_case, threshold=FUZZY_MATCH_THRESHOLD):：あいまい一致の判定。
    • if not keyword:：キーワードが空なら。
    • return True：空はすべて一致。
    • base, _ext = os.path.splitext(name)：拡張子を除いたベース名で比べる。
    • target = base.lower() if ignore_case else base：大文字小文字を無視する場合は小文字化。
    • kw = keyword.lower() if ignore_case else keyword：キーワード側も小文字化。
    • if kw in target:：部分一致なら即True。ここが一番速い経路。
    • return True：部分一致で返す。
    • candidates = [target] + [t for t in re.split(r'[ _\-.\(\)\[\]]+', target) if t]：全体に加え、区切り文字で分割したトークンも候補にする。report_finalならreportとfinalでも比べられる。
    • for token in candidates:：候補ごとに。
    • if difflib.SequenceMatcher(None, kw, token).ratio() >= threshold:：類似度が閾値以上なら一致。
    • return True：一致。
    • if _within_typo_distance(kw, token):：タイプミス距離でもチェックする。
    • return True：タイプミス範囲内なら一致。
    • return False：どれにも当てはまらなければ不一致。
    • def _within_typo_distance(keyword, token):：タイプミス許容範囲内かを判定する。
    • if not keyword or not token:：どちらかが空なら。
    • return False：不一致。
    • max_len = max(len(keyword), len(token))：長い方の長さで許容数を決める。
    • if max_len <= 2:：2文字以下は。
    • allowed = 0：タイプミスを許さない。
    • elif max_len <= 6:：6文字以下は。
    • allowed = 1：1文字まで許す。
    • else:：それより長い場合は。
    • allowed = 2：2文字まで許す。
    • if abs(len(keyword) - len(token)) > allowed:：長さの差が許容を超えたら。
    • return False：不一致。
    • return levenshtein_distance(keyword, token, max_dist=allowed) <= allowed：編集距離が許容内かで返す。
    • def levenshtein_distance(a, b, max_dist=None):：編集距離を計算する関数。
    • if a == b:：同一なら。
    • return 0：距離0。
    • la, lb = len(a), len(b)：長さを取得。
    • if max_dist is not None and abs(la - lb) > max_dist:：長さの差だけで許容を超えたら。
    • return max_dist + 1：許容超えとして返す。全部計算しない早期リターン。
    • prev = list(range(lb + 1))：前の行のDPテーブルを初期化。
    • for i in range(1, la + 1):：aの文字ごと。
    • cur = [i] + [0] * lb：現在の行を初期化。
    • row_min = cur[0]：行の最小値を追跡する。早期打ち切り用。
    • for j in range(1, lb + 1):：bの文字ごと。
    • cost = 0 if a[i - 1] == b[j - 1] else 1：同じ文字ならコスト0、違えば1。
    • cur[j] = min(prev[j] + 1, cur[j - 1] + 1, prev[j - 1] + cost)：削除・挿入・置換の最小値。
    • row_min = min(row_min, cur[j])：行の最小値を更新。
    • if max_dist is not None and row_min > max_dist:：行全体が許容を超えたら。
    • return max_dist + 1：これ以上計算しても許容内に収まらないので打ち切る。
    • prev = cur：次の行へ。
    • return prev[lb]：最後のセルの値が編集距離。
    • # 修正(v0.9.0・検索アルゴリズム最適化): 正規表現モードでは同じキーワードに：正規表現の再コンパイル対策。
    • # 対して1ファイルごとにre.compile()相当のコストが発生していた(re.search()は：re.search()は内部キャッシュがあるが、照合コストは毎回かかる。
    • # 内部でパターンをキャッシュするが、キャッシュキーの照合コスト自体は毎回：その照合コスト自体を避けたい。
    • # 発生する)。ここでは(パターン文字列, ignore_case)単位で明示的にコンパイル：パターンと大文字小文字フラグをキーにする。
    • # 済みオブジェクトをキャッシュし、同一検索内での再コンパイルコストを避ける。：同一検索内での再コンパイルを避ける。
    • # キャッシュは無制限に増え続けないよう、上限件数を超えたら丸ごとクリアする：上限を超えたら丸ごとクリアする単純な方針。
    • # (検索中に大量の異なる正規表現を使うケースは通常想定されないため、：大量の異なる正規表現は想定しない。
    • # 単純なサイズ上限で十分)。：コメントの締め。
    • _REGEX_CACHE = {}：コンパイル済み正規表現のキャッシュ。
    • _REGEX_CACHE_MAX = 256：上限256件。
    • def _get_compiled_regex(pattern, ignore_case):：キャッシュ付きで正規表現をコンパイルする。
    • key = (pattern, ignore_case)：キャッシュのキー。
    • compiled = _REGEX_CACHE.get(key)：キャッシュを確認。
    • if compiled is not None:：キャッシュにあれば。
    • return compiled：そのまま返す。
    • if len(_REGEX_CACHE) >= _REGEX_CACHE_MAX:：上限に達していたら。
    • _REGEX_CACHE.clear()：丸ごとクリアする。
    • flags = re.IGNORECASE if ignore_case else 0：フラグを決める。
    • compiled = re.compile(pattern, flags)：コンパイルする。
    • _REGEX_CACHE[key] = compiled：キャッシュへ保存。
    • return compiled：返す。
    • def _match_single_term(name, term, ignore_case, match_mode, ignore_width=False):：1つの検索語に対する一致判定。
    • if not term:：検索語が空なら。
    • return True：空は一致。
    • if ignore_width:：全角/半角を区別しない場合。
    • # 修正(v0.9.6・全角/半角を区別しない検索): NFKC正規化で全角英数・：NFKCで統一する。
    • # 半角カナ等を統一してから比較する(正規表現モードは対象外とする。：正規表現は対象外。
    • # 正規表現は文字クラスの意味が変わってしまうため)。：正規表現でNFKCすると文字クラスの意味が変わるため。
    • if match_mode != 'regex':：正規表現以外なら。
    • name = unicodedata.normalize('NFKC', name)：ファイル名をNFKC化。
    • term = unicodedata.normalize('NFKC', term)：検索語もNFKC化。
    • if match_mode == 'regex':：正規表現モード。
    • try:：コンパイルの例外に備える。
    • compiled = _get_compiled_regex(term, ignore_case)：キャッシュ付きでコンパイル。
    • except re.error:：不正な正規表現の場合。
    • return False：不一致として扱う。検索全体を止めない。
    • return compiled.search(name) is not None：部分一致するかで返す。
    • if match_mode == 'fuzzy':：あいまいモード。
    • return fuzzy_name_match(name, term, ignore_case)：あいまい判定へ委譲。
    • # 修正(v0.9.0): 完全一致・ワイルドカードより先に、最も呼び出し頻度が：判定順序の最適化。
    • # 高い「部分一致」を判定できるよう、以降の判定順序を軽い処理から並べる。：軽い処理から並べる。
    • target = name.lower() if ignore_case else name：比較用に小文字化。
    • kw = term.lower() if ignore_case else term：検索語も小文字化。
    • if match_mode == 'exact':：完全一致モード。
    • base, _ext = os.path.splitext(name)：拡張子を除いたベース名。
    • base_cmp = base.lower() if ignore_case else base：ベース名も小文字化。
    • return target == kw or base_cmp == kw：拡張子込みかベース名のどちらかが完全一致すればTrue。
    • if any(ch in term for ch in "*?"):：ワイルドカード文字が含まれている場合。
    • return fnmatch.fnmatch(target, kw)：fnmatchで判定。
    • return kw in target：それ以外は部分一致。
    • def match_keyword_name(name, keyword, ignore_case, match_mode='partial', ignore_width=False):：キーワード全体の一致判定。
    • if not keyword:：キーワードが空なら。
    • return True：空は一致。
    • if match_mode == 'regex':：正規表現モードはブール演算子を使わない。
    • return _match_single_term(name, keyword, ignore_case, match_mode, ignore_width)：正規表現としてそのまま判定。
    • terms = parse_boolean_query(keyword)：ブール演算子で分割する。
    • if not terms:：項がなければ。
    • return True：一致扱い。
    • result = True：結果の初期値。
    • for i, (op, term) in enumerate(terms):：項ごとに評価する。
    • term_match = _match_single_term(name, term, ignore_case, match_mode, ignore_width)：その項の一致を判定。
    • if i == 0:：最初の項は。
    • result = term_match：そのまま結果にする。
    • elif op == 'AND':：ANDなら。
    • result = result and term_match：両方一致でTrue。
    • elif op == 'OR':：ORなら。
    • result = result or term_match：どちらか一致でTrue。
    • elif op == 'NOT':：NOTなら。
    • result = result and (not term_match)：今までの結果がTrueで、かつこの項に一致しない場合だけTrue。
    • return result：最終結果を返す。なお、この評価は左から順に行われ、AND/ORの優先順位や括弧はない点に注意。
    • def match_filters(name, ext_filter, exclude_ext=None, full_path=None, include_hidden=True,：拡張子・隠し・システムなどのフィルタ判定。
    • include_system=True, exclude_filename_patterns=None, exclude_temp=False):：引数の続き。
    • if ext_filter:：拡張子フィルタがある場合。
    • _, ext = os.path.splitext(name)：拡張子を取得。
    • if ext.lower().lstrip(".") not in ext_filter:：フィルタに含まれなければ。
    • return False：不一致。
    • if exclude_ext:：除外拡張子がある場合。
    • _, ext = os.path.splitext(name)：拡張子を取得。
    • if ext.lower().lstrip(".") in exclude_ext:：除外対象なら。
    • return False：不一致。
    • # 修正(v0.9.6・一時ファイルを除外/除外ファイル名): 拡張子より前段の：判定順序のコメント。
    • # フィルタと同様、判定コストの低いものから順に評価する。：軽いものから評価する。
    • if exclude_temp and is_temp_file(name):：一時ファイル除外がONで、一時ファイルなら。
    • return False：不一致。
    • if exclude_filename_patterns and match_exclude_filename(name, exclude_filename_patterns):：除外ファイル名パターンに一致したら。
    • return False：不一致。
    • # 修正(v0.9.6・隠し/システムファイルの個別検索対応): 従来は：隠し/システムの分離対応。
    • # is_hidden_or_system()で隠し・システムを一括判定していたが、：従来は一括だった。
    • # include_hidden/include_systemそれぞれ個別にOFFにできるようにする。：個別にOFFにできるようにする。
    • if full_path is not None:：フルパスが分かっている場合。
    • if not include_hidden and is_hidden(full_path, name):：隠しを除外する設定で、隠しなら。
    • return False：不一致。
    • if not include_system and is_system_file(full_path):：システムを除外する設定で、システムなら。
    • return False：不一致。
    • return True：ここまで来たら一致。
    • if not include_hidden and name.startswith("."):：フルパスが分からない場合のフォールバック。ドット始まりで代用する。
    • return False：不一致。
    • return True：それ以外は一致。
    • def parse_exclude_folders(text):：除外フォルダのテキストを集合へ変換する。
    • if not text:：空なら。
    • return set()：空集合。
    • return {p.strip().lower() for p in text.split(",") if p.strip()}：カンマ分割・小文字化して集合化。
    • # 修正(v0.9.6・一時ファイルを除外): よくある一時/ロックファイルのパターン。：一時ファイルのパターン。
    • # fnmatch用のワイルドカードで表現する(大文字小文字はis_temp_file側で無視)。：fnmatch用のワイルドカード。
    • TEMP_FILE_PATTERNS = (：一時ファイルのパターンのタプル。
    • '~$*', '*.tmp', '*.temp', '*.bak', '*.swp', '*.crdownload', '*.part',：代表的な一時ファイル。
    • '*.download', 'thumbs.db', 'desktop.ini', '.ds_store', 'ehthumbs.db',：OSやブラウザの一時ファイルも含む。
    • )：タプルの終わり。
    • # 修正(v0.9.6・Windowsシステム領域を除外): 検索対象から除外したい：システム関連フォルダ名の除外。
    • # Windows/アプリのシステム関連フォルダ名。exclude_foldersと同じ仕組み：既存の除外フォルダの仕組みに相乗りさせる。
    • # (フォルダ名の小文字一致)に相乗りさせることで、既存の走査除外ロジックを：既存ロジックを再利用する。
    • # そのまま再利用できる。：コメントの締め。
    • SYSTEM_EXCLUDE_FOLDER_NAMES = {：システム除外フォルダ名の集合。
    • 'windows', 'program files', 'program files (x86)', 'programdata',：代表的なシステムフォルダ。
    • '$recycle.bin', 'system volume information', 'msocache', 'recovery',：ごみ箱や復旧領域など。
    • '$windows.~bt', '$windows.~ws',：Windowsアップデートの一時フォルダ。
    • }：集合の終わり。
    • def is_temp_file(name):：一時ファイルかを判定する。
    • lname = name.lower()：小文字化して比較する。
    • return any(fnmatch.fnmatch(lname, pat) for pat in TEMP_FILE_PATTERNS)：どれか1つでも一致すれば一時ファイル。
    • def parse_exclude_filenames(text):：除外ファイル名のパターンを集合へ変換する。
    • if not text:：空なら。
    • return set()：空集合。
    • return {p.strip() for p in text.split(",") if p.strip()}：カンマ分割して集合化。ここでは小文字化はしない。比較時に小文字化する。
    • def match_exclude_filename(name, patterns):：除外ファイル名パターンに一致するかを判定する。
    • if not patterns:：パターンがなければ。
    • return False：不一致。
    • lname = name.lower()：小文字化。
    • return any(fnmatch.fnmatch(lname, pat.lower()) for pat in patterns)：パターン側も小文字化して比較。
    • def get_drive_type_label(path):：ドライブ種別を判定する。USBなどのリムーバブルなら'removable'を返す。
    • """修正(v0.9.6・USB/外付けドライブの扱い設定): 指定パスが属するドライブの：USB/外付けドライブの扱い設定。
    • 種別を判定する。リムーバブルドライブ(USBメモリ等)の場合 'removable' を、：リムーバブルなら'removable'。
    • それ以外/判定不能なら None を返す。Windows以外・判定失敗時は常にNone。"""：それ以外はNone。
    • if not sys.platform.startswith("win"):：Windows以外は。
    • return None：常にNone。
    • try:：API呼び出しの例外に備える。
    • import ctypes：Windows API用。
    • drive = os.path.splitdrive(os.path.abspath(path))[0]：ドライブ文字を取り出す。
    • if not drive:：ドライブが取れなければ。
    • return None：判定不能。
    • drive_root = drive + "\\"：ドライブルートを作る。
    • DRIVE_REMOVABLE = 2：リムーバブルの値。
    • dtype = ctypes.windll.kernel32.GetDriveTypeW(drive_root)：ドライブ種別を取得。
    • if dtype == DRIVE_REMOVABLE:：リムーバブルなら。
    • return 'removable'：'removable'を返す。
    • return None：それ以外はNone。
    • except Exception as exc:：例外時。
    • log_error("ドライブ種別判定", f"{path}: {exc}")：ログに残す。
    • return None：判定不能はNone。
    • def dir_is_excluded(dir_path, exclude_folders):：フォルダパスが除外対象かを判定する。
    • if not exclude_folders:：除外指定がなければ。
    • return False：除外ではない。
    • parts = os.path.normpath(dir_path).lower().replace("\\", "/").split("/")：正規化・小文字化・区切り統一して分割する。
    • return any(p in exclude_folders for p in parts if p)：どれか1つの要素が除外集合にあれば除外。
    • EXTENSION_PRESETS = {：拡張子プリセット。UIのプルダウン用。
    • '画像': "jpg,jpeg,png,gif,bmp,webp,svg,tiff,ico,heic",：画像。
    • '動画': "mp4,mov,avi,wmv,mkv,flv,webm,m4v",：動画。
    • '音楽': "mp3,wav,flac,aac,m4a,ogg,wma",：音楽。
    • '文書': "txt,doc,docx,pdf,rtf,odt,md,xlsx,xls,ppt,pptx,csv",：文書。
    • '圧縮ファイル': "zip,rar,7z,tar,gz,bz2,xz",：圧縮ファイル。
    • '実行ファイル': "exe,msi,bat,sh,app,apk",：実行ファイル。
    • 'コード': "py,js,ts,html,css,java,c,cpp,cs,go,rs,php,rb,json,xml,yml",：コード。
    • }：プリセットの終わり。
    • SEARCH_TEMPLATES = {：検索テンプレート。拡張子だけでなく、内容検索やフォルダ検索の有無も含む。
    • '画像': {'ext': EXTENSION_PRESETS['画像'], 'content_search': False, 'folders_too': False},：画像はファイル名のみ。
    • '動画': {'ext': EXTENSION_PRESETS['動画'], 'content_search': False, 'folders_too': False},：動画もファイル名のみ。
    • '音楽': {'ext': EXTENSION_PRESETS['音楽'], 'content_search': False, 'folders_too': False},：音楽もファイル名のみ。
    • '文書': {'ext': EXTENSION_PRESETS['文書'], 'content_search': True, 'folders_too': False},：文書は内容検索もON。
    • '圧縮ファイル': {'ext': EXTENSION_PRESETS['圧縮ファイル'], 'content_search': False, 'folders_too': False},：圧縮ファイルはファイル名のみ。
    • '実行ファイル': {'ext': EXTENSION_PRESETS['実行ファイル'], 'content_search': False, 'folders_too': False},：実行ファイルはファイル名のみ。
    • 'コード': {'ext': EXTENSION_PRESETS['コード'], 'content_search': True, 'folders_too': False},：コードは内容検索もON。
    • # 修正(v0.9.6・検索テンプレート追加): 拡張子ではなくサイズ・更新日時で：サイズ・更新日時で絞るテンプレート。
    • # 絞り込む「FSEらしい」テンプレートを追加する。'size_min_mb'は最小サイズ：size_min_mbは最小サイズ(MB)。
    • # (MB)、'mtime_days'は「直近N日以内に更新」を表し、_on_template_selected()：mtime_daysは直近N日以内。
    • # 側でsize_min_var/mtime_from_varへ変換して適用する。：選択時に変数へ変換して適用する。
    • '大きいファイル': {'ext': '', 'content_search': False, 'folders_too': False, 'size_min_mb': 100},：100MB以上のファイル。
    • '最近変更したファイル': {'ext': '', 'content_search': False, 'folders_too': False, 'mtime_days': 7},：直近7日以内に更新されたファイル。
    • }：テンプレートの終わり。
    • # 修正(v0.9.6・正規表現テンプレート): ファイル名検索でよく使う正規表現。：正規表現テンプレート。
    • REGEX_TEMPLATES = {：正規表現テンプレートの辞書。
    • '日付入り(2024-01-31等)': r'\d{4}[-_.]?\d{2}[-_.]?\d{2}',：日付入り。
    • '連番付き(_001等)': r'[_-]\d{2,}(\.[^.]+)?$',：連番付き。
    • 'バージョン番号(v1.2.3等)': r'[vV]?\d+\.\d+(\.\d+)?',：バージョン番号。
    • 'コピー・重複(コピー/(1)/copy)': r'(コピー|copy|\(\d+\))',：コピー・重複。
    • '画像ファイル': r'\.(jpe?g|png|gif|bmp|webp)$',：画像ファイル。
    • 'IMG_/DSC_で始まる': r'^(IMG|DSC|DSCN|P)_?\d+',：カメラファイル名。
    • '数字のみの名前': r'^\d+(\.[^.]+)?$',：数字のみ。
    • 'スペースを含む': r'\s',：スペースを含む。
    • '全角文字を含む': r'[^\x00-\x7F]',：全角文字を含む。
    • 'ドットで始まる': r'^\.',：ドットで始まる。
    • }：正規表現テンプレートの終わり。
    • EXECUTION_CONFIRM_EXTENSIONS = frozenset({：実行前に確認が必要な拡張子の集合。
    • 'ps1', 'vbe', 'cpl', 'reg', 'exe', 'msc', 'ps1xml', 'apk', 'bat', 'cmd',：スクリプトや実行形式。
    • 'sh', 'vbs', 'scr', 'jse', 'com', 'wsh', 'msi', 'jar', 'wsf', 'js'：スクリプトやインストーラなど。
    • })：集合の終わり。
    • CONTENT_SEARCH_EXTENSIONS = TEXT_PREVIEW_EXTENSIONS：内容検索の対象拡張子は、テキストプレビュー対象と同じ集合を使う。
    • CONTENT_SEARCH_MAX_BYTES = 5242880：内容検索の最大サイズ。5MBを超えるファイルは内容検索しない。
    • CONTENT_SNIPPET_MAX_CHARS = 120：内容検索のスニペットの最大文字数。
    • # 修正(v0.8.7・安定性/速度): minifyされたJS/CSSや改行の無い巨大な1行の：巨大な1行への対策。
    • # ログ・JSON等では、「1行ずつ読み込む」処理が実質ファイル全体(最大：改行なしだと1行がファイル全体になる。
    • # CONTENT_SEARCH_MAX_BYTESまで許容)を1つの巨大な文字列として一度に：巨大な文字列を一度に作るとメモリが跳ねる。
    • # 構築することになり、瞬間的なメモリ増加や処理の一時停止を招く。：メモリ増加や一時停止の原因。
    • # ここでは1回の読み込み単位(実質的な走査チャンク)の文字数に上限を設け、：読み込み単位に上限を設ける。
    • # 改行の無い長い行はこの単位で分割して走査する(キーワードがチャンクの：長い行は分割して走査する。
    • # 境目をまたぐケースを取りこぼさないよう、次のチャンクの先頭にキーワード：境目をまたぐキーワードを取りこぼさないよう持ち越す。
    • # 長さ分だけ直前チャンクの末尾を持ち越して連結してから検索する)。：持ち越し連結の説明。
    • LINE_SCAN_CHUNK_CHARS = 8192：1回の読み込み単位は8192文字。
## 第6章：内容検索
    • def search_file_content(path, keyword, ignore_case, max_bytes=CONTENT_SEARCH_MAX_BYTES, stop_event=None):：ファイル内容を検索する関数。見つかれば"{行番号}行目: {スニペット}"を返し、見つからなければNone。
    • # 修正7: 検索中にファイルが削除・移動されるレース条件に対応する。：検索中の削除・移動への対策。
    • # 以前はos.path.getsize()の例外(FileNotFoundError等)が捕捉されておらず、：以前はgetsize()の例外が捕捉されていなかった。
    • # このファイル1件のせいで検索スレッド全体が異常終了してしまっていた。：1件の例外で検索全体が止まっていた。
    • # 修正(v0.8.4・多重防御): safe_delete_file()では削除対象がシンボリックリンク：シンボリックリンクへの多重防御。
    • # である場合を明示的に除外しているが、内容検索側は無防備にopen()していた。：内容検索側は無防備だった。
    • # シンボリックリンクは実体が検索対象フォルダの外側(監視対象外の場所)を：リンク先が検索対象外の場所を指すことがある。
    • # 指している可能性があり、意図せずそこへアクセスして内容を読んでしまう：意図せず対象外を読んでしまう恐れがあった。
    • # ことになる。壊れたリンクの検出にもなるため、ここでスキップする。：ここでスキップすることで壊れたリンクの検出にもなる。
    • try:：リンク判定の例外に備える。
    • if os.path.islink(path):：シンボリックリンクなら。
    • return None：内容検索しない。
    • except OSError:：判定に失敗した場合。
    • return None：内容検索しない。
    • try:：サイズ判定の例外に備える。
    • if os.path.getsize(path) > max_bytes:：上限を超えたら。
    • return None：内容検索しない。
    • except OSError:：サイズ取得に失敗した場合。
    • return None：内容検索しない。
    • kw = keyword.lower() if ignore_case else keyword：比較用に小文字化。
    • if not kw:：キーワードが空なら。
    • return None：検索しない。
    • try:：サンプル読み込みの例外に備える。
    • with open(path, "rb") as f:：バイナリで開く。
    • sample = f.read(4096)：先頭4096バイトを読む。
    • except OSError:：開けない場合。
    • return None：内容検索しない。
    • def _scan_with_encoding(enc, strict):：指定エンコーディングで走査する内部関数。
    • try:：走査中の例外に備える。
    • with open(path, "r", encoding=enc, errors=("strict" if strict else "replace")) as f:：strictなら厳密に、replaceなら置換しながら読む。
    • lineno = 1：行番号の初期値。
    • carry = ""：チャンク境目の持ち越し用。
    • read_count = 0：読み込み回数のカウンタ。停止チェック用。
    • while True:：チャンク読み込みループ。
    • # 修正(v0.8.7): readline()にサイズ上限を渡すことで、改行の：readline()にサイズ上限を渡す修正。
    • # 無い巨大な1行(minifyされたJS/CSS・改行なしログ等)でも：巨大な1行でも。
    • # 一度にLINE_SCAN_CHUNK_CHARSを超える文字列を構築しない。：一度に巨大な文字列を作らない。
    • # 実際の行がこの上限より長い場合、readline()はその行の：上限より長い行は途中で切れて返る。
    • # 途中までを返し、次回呼び出しで続きが返る(改行を跨がない)。：次回呼び出しで続きが返る。
    • chunk = f.readline(LINE_SCAN_CHUNK_CHARS)：8192文字まで読む。
    • if not chunk:：読み込むものがなくなったら。
    • break：ループを抜ける。
    • read_count += 1：読み込み回数を増やす。
    • # 修正(v0.8.2・停止応答性)を維持: 128回の読み込みおきに：停止応答性の維持。
    • # stop_eventを確認し、巨大ファイル・巨大行の途中でも：巨大ファイルでも止まれるようにする。
    • # 速やかに検索全体を打ち切れるようにする。：速やかに打ち切る。
    • if stop_event is not None and read_count % 128 == 0 and stop_event.is_set():：128回ごとに停止確認。
    • return None：停止ならNoneを返す。
    • ends_line = chunk.endswith("\n")：このチャンクが改行で終わるか確認。
    • # キーワードがチャンクの境目をまたいで途切れないよう、：境目をまたぐキーワードへの対策。
    • # 直前チャンクの末尾(キーワード長-1文字分)を持ち越して連結する。：直前チャンクの末尾を持ち越す。
    • search_text = carry + chunk if carry else chunk：持ち越し分と今回分を連結する。
    • target = search_text.lower() if ignore_case else search_text：比較用に小文字化。
    • idx = target.find(kw)：キーワードを検索する。
    • if idx != -1:：見つかった場合。
    • snippet = search_text.strip()：前後の空白を除いたスニペットを作る。
    • if len(snippet) > CONTENT_SNIPPET_MAX_CHARS:：120文字を超えたら。
    • start = max(0, idx - 40)：キーワードの40文字前から切り出す。
    • snippet = search_text.strip()[start:start + CONTENT_SNIPPET_MAX_CHARS]：120文字分切り出す。
    • snippet = ("…" if start > 0 else "") + snippet + "…"：前後を省略したことを…で示す。
    • return f"{lineno}行目: {snippet}"：行番号付きで返す。
    • if ends_line:：改行で終わっていたら。
    • lineno += 1：行番号を進める。
    • carry = ""：持ち越しをクリアする。
    • else:：改行で終わっていない場合。つまり行の途中。
    • overlap = max(0, len(kw) - 1)：持ち越す文字数はキーワード長マイナス1。
    • carry = search_text[-overlap:] if overlap else ""：末尾だけを持ち越す。
    • return None：見つからなかった場合。
    • except UnicodeDecodeError:：デコードに失敗した場合。
    • return "ERROR"：呼び出し側へデコード失敗を知らせる目印。
    • except OSError:：読み込みに失敗した場合。
    • return None：内容検索しない。
    • # 修正5: UTF-16はNULバイトを含むため、バイナリ判定(_looks_binary)より先に：ここも修正5の順序。
    • # BOM/ヒューリスティックによる文字コード判定を行う：BOM判定を先に行う。
    • detected_enc = _detect_text_encoding(sample)：BOM/ヒューリスティックで判定。
    • if detected_enc:：判定できた場合。
    • result = _scan_with_encoding(detected_enc, strict=False)：replaceで走査する。
    • return None if result == "ERROR" else result：ERRORならNone、それ以外は結果を返す。
    • if _looks_binary(sample):：判定できなかった場合にバイナリ判定。
    • return None：バイナリなら内容検索しない。
    • for enc in ('utf-8', 'cp932'):：UTF-8とCP932を順に試す。
    • result = _scan_with_encoding(enc, strict=True)：strictで走査する。
    • if result == "ERROR":：デコード失敗なら。
    • continue：次のエンコーディングへ。
    • return result：見つかったか、見つからなかったかを返す。Noneなら見つからなかった。
    • return None：どちらでも読めなかった場合。
## 第7章：1エントリ評価と検索ワーカー
    • def _evaluate_entry(name, full_path, is_dir, keyword, ext_filter, exclude_ext, ignore_case,：1エントリが条件に一致するかを判定するモジュールレベル関数。
    • match_mode, include_hidden, content_search, exclude_folders, stop_event=None,：引数の続き。
    • content_search_extensions=None, ignore_width=False, include_system=True,：引数の続き。
    • search_target='name', exclude_filename_patterns=None, exclude_temp=False):：引数の続き。
    • """1エントリ(ファイル/フォルダ)がキーワード・拡張子・内容検索の条件に：docstring。
    • 一致するかどうかを判定する。SearchWorker._evaluate()と：SearchWorkerとParallelSearchWorkerの両方から呼ばれる。
    • ParallelSearchWorker(ThreadPoolExecutorのワーカースレッド)の両方から：並列検索のワーカースレッドからも呼ばれる。
    • 共通して呼び出せるよう、インスタンスに依存しないモジュールレベル関数として：インスタンスに依存しない形に切り出している。
    • 切り出している(並列検索ではスレッドごとにselfの状態を共有する必要がないため、：並列検索ではスレッドごとに状態を共有しなくてよいため。
    • この形の方が扱いやすい)。：この形の方が扱いやすい。
    • 修正(v0.9.0・検索アルゴリズム最適化): 判定順序を「計算コストが低く、：判定順序の最適化。
    • かつ絞り込み効果が高いもの」から並べる。除外フォルダ判定(セット参照)・：除外フォルダ判定や拡張子フィルタは軽い。
    • 拡張子フィルタ(文字列比較)はキーワード一致判定(ループ・正規表現を伴いうる)：キーワード判定は重くなりうる。
    • より軽いため先に行う(従来の順序を維持しつつ、拡張子取得の重複呼び出しを：軽いものから先に行う。
    • 1回にまとめる)。"""：docstringの締め。
    • if is_dir and exclude_folders and name.lower() in exclude_folders:：フォルダで、除外フォルダに含まれていたら。
    • return (False, None)：不一致。スニペットなし。
    • if not match_filters(name, ext_filter, exclude_ext, full_path, include_hidden, include_system,：拡張子・隠し・システムなどのフィルタ判定。
    • exclude_filename_patterns=exclude_filename_patterns, exclude_temp=exclude_temp):：引数の続き。
    • return (False, None)：フィルタに落ちたら不一致。
    • def _content_match():：内容検索の実行部分を切り出した内部関数。
    • # 修正(v0.9.6・検索対象「ファイル名のみ/内容のみ/両方」対応):：検索対象の切り替え対応。
    • # 内容検索の実行部分を切り出し、search_targetの値に応じて：search_targetに応じて呼び分けられるようにする。
    • # 呼び分けられるようにする。：コメント。
    • if is_dir or not keyword or match_mode == 'regex':：フォルダ・キーワードなし・正規表現モードでは内容検索しない。
    • return None：内容検索なし。
    • ext = os.path.splitext(name)[1].lower().lstrip(".")：拡張子を取得。
    • # 修正(v0.9.6・内容検索対象拡張子の設定化): 設定画面で対象拡張子が：内容検索対象拡張子の設定化。
    • # カスタマイズされている場合はそちらを使い、未指定(None)なら従来通り：設定があればそれを使う。
    • # 既定の拡張子集合(CONTENT_SEARCH_EXTENSIONS)にフォールバックする。：未指定なら既定値へ。
    • allowed_exts = content_search_extensions if content_search_extensions is not None \：設定値か既定値かを選ぶ。
    • else CONTENT_SEARCH_EXTENSIONS：既定値へのフォールバック。
    • if ext not in allowed_exts:：対象拡張子でなければ。
    • return None：内容検索しない。
    • return search_file_content(full_path, strip_phrase_quotes(keyword), ignore_case, stop_event=stop_event)：内容検索を実行する。引用符はここで外す。
    • # 修正(v0.9.6・検索対象「ファイル名のみ/内容のみ/両方」対応):：検索対象の切り替え。
    • # search_target='content' の場合はファイル名一致を評価せず、内容一致：contentなら内容のみ。
    • # のみで判定する。'name'(従来のcontent_search無効時と同じ)はファイル名：nameならファイル名のみ。
    • # のみ。'both'は従来どおりファイル名 OR 内容(content_searchがONの場合)。：bothならファイル名OR内容。
    • if search_target == 'content':：内容のみの場合。
    • snippet = _content_match()：内容検索を実行。
    • return (snippet is not None, snippet)：見つかれば一致。
    • if match_keyword_name(name, keyword, ignore_case, match_mode, ignore_width):：ファイル名一致を判定。
    • return (True, None)：ファイル名で一致したらスニペットなしで返す。
    • if search_target == 'both' and content_search:：両方対象で、内容検索がONなら。
    • snippet = _content_match()：内容検索を実行。
    • if snippet is not None:：内容で見つかったら。
    • return (True, snippet)：一致としてスニペット付きで返す。
    • return (False, None)：どれにも当てはまらなければ不一致。
    • class SearchWorker(threading.Thread):：検索を実行するスレッドクラス。
    • BATCH_SIZE = 50：結果をまとめて送る件数。
    • BATCH_INTERVAL = 0.15：バッチを送る間隔(秒)。
    • EXAMINE_REPORT_EVERY = 200：走査件数の報告間隔。
    • # 修正(v0.8.2・メモリ保護): 権限エラー等でアクセスできないフォルダが：アクセスエラーのメモリ保護。
    • # 極端に多い環境(例: ネットワークドライブの大部分に権限が無い等)では、：権限のないフォルダが多い環境では。
    • # access_errorsリストが検索結果本体(MAX_STORED_RESULTS)とは別に：アクセスエラーのリストが無制限に増える恐れがあった。
    • # 無制限に肥大化しうる。上限を設け、超過分は集計のみ(件数カウント)に切り替える。：上限を設け、超過分は件数だけ数える。
    • MAX_ACCESS_ERRORS = 5000：アクセスエラーの保持上限。
    • def __init__(self, root_dir, keyword, content_search, ext_filter, exclude_ext,：コンストラクタ。検索条件をすべて受け取る。
    • recursive, ignore_case, search_folders_too, match_mode, include_hidden,：引数の続き。
    • exclude_folders=None, size_min=None, size_max=None,：引数の続き。
    • mtime_from=None, mtime_to=None, ctime_from=None, ctime_to=None,：引数の続き。
    • on_result_batch=None, on_done=None, on_error=None, on_examined=None,：コールバック群。
    • batch_size=None, batch_interval=None, max_result_cap=None,：バッチ設定と保持上限。
    • content_search_extensions=None, ignore_width=False, include_system=True,：内容検索拡張子・全角半角・システム。
    • search_target='name', exclude_filename_patterns=None, exclude_temp=False,：検索対象・除外ファイル名・一時ファイル。
    • low_load_mode=False, parallel_worker_count=0):：低負荷モード・並列スレッド数。
    • super().__init__(daemon=True)：デーモンスレッドとして初期化。アプリ終了時に道連れで終了する。
    • if batch_size:：バッチサイズの指定があれば。
    • self.BATCH_SIZE = batch_size：上書きする。
    • if batch_interval:：バッチ間隔の指定があれば。
    • self.BATCH_INTERVAL = batch_interval：上書きする。
    • self.root_dir = root_dir：検索ルートを保存。
    • self.keyword = keyword：キーワードを保存。
    • self.content_search = content_search：内容検索の有無を保存。
    • self.ext_filter = normalize_ext_filter(ext_filter)：拡張子フィルタを正規化して保存。
    • self.exclude_ext = normalize_ext_filter(exclude_ext)：除外拡張子も正規化して保存。
    • self.recursive = recursive：再帰検索の有無。
    • self.ignore_case = ignore_case：大文字小文字を無視するか。
    • self.search_folders_too = search_folders_too：フォルダも検索対象にするか。
    • self.match_mode = match_mode：一致モード。
    • self.include_hidden = include_hidden：隠しファイルを含めるか。
    • # 修正(v0.9.6): システムファイルの個別除外・全角半角無視・：v0.9.6の追加項目。
    • # 検索対象(ファイル名/内容/両方)を検索ワーカーへ保持する。：検索対象などを保持する。
    • self.include_system = include_system：システムファイルを含めるか。
    • self.ignore_width = ignore_width：全角半角を無視するか。
    • self.search_target = search_target：検索対象。
    • # 修正(v0.9.6・除外ファイル名/一時ファイルを除外): ワイルドカード：除外ファイル名・一時ファイルの対応。
    • # パターン集合とON/OFFフラグを保持する。：パターン集合とフラグを保持する。
    • self.exclude_filename_patterns = exclude_filename_patterns or set()：除外ファイル名パターン。
    • self.exclude_temp = exclude_temp：一時ファイル除外フラグ。
    • # 修正(v0.9.6・低負荷モード): ONの場合、走査ループに軽いウェイトを：低負荷モードの説明。
    • # 挟んでCPU使用率の急上昇を抑える(_maybe_throttle参照)。：ウェイトでCPUを抑える。
    • self.low_load_mode = low_load_mode：低負荷モードを保存。
    • # 修正(v0.9.6・並列スレッド数の設定): ParallelSearchWorker._worker_count()：並列スレッド数の設定。
    • # から参照する(SearchWorker単体では未使用)。：SearchWorker単体では未使用。
    • self.parallel_worker_count = parallel_worker_count or 0：並列スレッド数を保存。
    • self.exclude_folders = exclude_folders or set()：除外フォルダ。
    • self.size_min = size_min：最小サイズ。
    • self.size_max = size_max：最大サイズ。
    • self.mtime_from = mtime_from：更新日時の下限。
    • self.mtime_to = mtime_to：更新日時の上限。
    • self.ctime_from = ctime_from：作成日時の下限。
    • self.ctime_to = ctime_to：作成日時の上限。
    • self.on_result_batch = on_result_batch：結果バッチのコールバック。
    • self.on_done = on_done：完了時のコールバック。
    • self.on_error = on_error：エラー時のコールバック。
    • self.on_examined = on_examined：走査件数通知のコールバック。
    • # 修正(v0.9.0・大量ファイル環境の走査最適化): UI側の保持上限：大量ファイル環境の最適化。
    • # (SearchTab.MAX_STORED_RESULTS)に相当する値を渡しておくことで、：UI側の保持上限を渡しておく。
    • # 既にその件数を超えて見つかっている場合、サイズ/日付範囲フィルタが：上限を超えたらstat()を省略できる。
    • # 無効な検索に限り以降のos.stat()呼び出しを省略できるようにする：stat()呼び出しを省略する。
    • # (UI側はどのみち上限超過分を保持しないため、正確なサイズ・更新日時を：上限超過分はUIで捨てられるため。
    • # 計算する意味が無い)。Noneの場合は上限なし(従来どおり)として扱う。：Noneなら上限なし。
    • self.max_result_cap = max_result_cap：保持上限を保存。
    • # 修正(v0.9.6・内容検索対象拡張子の設定化): 設定画面でカスタマイズ：内容検索対象拡張子の設定化。
    • # された対象拡張子の集合。Noneなら_evaluate_entry側で既定値：Noneなら既定値へ。
    • # (CONTENT_SEARCH_EXTENSIONS)にフォールバックする。：フォールバックの説明。
    • self.content_search_extensions = content_search_extensions：内容検索対象拡張子を保存。
    • self._stop_event = threading.Event()：停止用のイベント。
    • # 修正(v0.9.6・検索中の一時停止): set()の間は走査を継続し、clear()の：一時停止の仕組み。
    • # 間は走査ループの先頭で待機する(停止とは異なり、後から再開できる)。：停止とは違い、後から再開できる。
    • self._pause_event = threading.Event()：一時停止用のイベント。
    • self._pause_event.set()：初期状態は実行中。
    • self.found_count = 0：見つかった件数。
    • self.examined_count = 0：走査した件数。
    • self.access_errors = []：アクセスエラーの一覧。
    • # 修正(v0.8.2): 実際に発生した総アクセスエラー件数(上限超過分も含む)。：総件数のカウンタ。
    • # UI側の「権限エラー: N件」表示は保持件数ではなくこちらを使う。：UI表示は総件数を使う。
    • self.access_error_total_count = 0：総アクセスエラー件数。
    • self.access_errors_truncated = False：上限で打ち切ったかどうかのフラグ。
    • def stop(self):：停止を依頼する。
    • self._stop_event.set()：停止イベントを立てる。
    • # 停止時は一時停止状態のままだと待機ループから抜けられないため、：一時停止中だと待機ループから抜けられない。
    • # 一時停止も解除して速やかにstop_eventを検知できるようにする。：一時停止も解除する。
    • self._pause_event.set()：一時停止を解除する。
    • def pause(self):：一時停止を依頼する。
    • self._pause_event.clear()：一時停止イベントをクリアする。
    • def resume(self):：再開を依頼する。
    • self._pause_event.set()：一時停止イベントを立てる。
    • def _wait_while_paused(self):：一時停止中はここで待つ。
    • """一時停止中はここでブロックする。停止指示が来たら即座に抜ける。：docstring。
    • 戻り値はstop_eventが立ったかどうか。"""：戻り値は停止したかどうか。
    • while not self._pause_event.is_set():：一時停止中はループする。
    • if self._stop_event.is_set():：停止指示が来たら。
    • return True：停止したとして抜ける。
    • time.sleep(0.1)：0.1秒待って再確認する。
    • return False：一時停止が解除されたら抜ける。
    • def _maybe_throttle(self, examined_count):：低負荷モードのウェイト。
    • """修正(v0.9.6・低負荷モード): CPU使用率を抑えたい場合、一定件数：低負荷モードの説明。
    • ごとに短いスリープを挟んでCPUを他プロセスへ譲る。"""：CPUを他プロセスへ譲る。
    • if self.low_load_mode and examined_count % 300 == 0:：低負荷モードで300件ごとに。
    • time.sleep(0.01)：0.01秒待つ。
    • def _on_walk_error(self, exc):：os.walkのエラー時の処理。
    • path = getattr(exc, 'filename', None) or self.root_dir：エラーが起きたパス。取れなければルート。
    • err_type, message = _classify_access_error(path, exc)：エラーを分類する。
    • self.access_error_total_count += 1：総件数を増やす。
    • if len(self.access_errors) >= self.MAX_ACCESS_ERRORS:：保持上限に達していたら。
    • if not self.access_errors_truncated:：初めて打ち切る場合。
    • self.access_errors_truncated = True：打ち切りフラグを立てる。
    • log_error(：ログに残す。
    • "検索(アクセスエラー)",：カテゴリ。
    • f"アクセスエラーが{self.MAX_ACCESS_ERRORS}件を超えたため、"：上限超過のメッセージ。
    • "以降は一覧への保持を打ち切り件数のみ集計します。")：以降は件数のみ集計する。
    • return：一覧への追加はしない。
    • self.access_errors.append({'path': path, 'type': err_type, 'message': message})：一覧へ追加する。
    • def _evaluate(self, name, full_path, is_dir):：_evaluate_entryへの薄いラッパー。
    • return _evaluate_entry(：モジュールレベル関数へ委譲する。
    • name, full_path, is_dir, self.keyword, self.ext_filter, self.exclude_ext,：引数を渡す。
    • self.ignore_case, self.match_mode, self.include_hidden, self.content_search,：引数の続き。
    • self.exclude_folders, stop_event=self._stop_event,：引数の続き。
    • content_search_extensions=self.content_search_extensions,：引数の続き。
    • ignore_width=self.ignore_width, include_system=self.include_system,：引数の続き。
    • search_target=self.search_target,：引数の続き。
    • exclude_filename_patterns=self.exclude_filename_patterns, exclude_temp=self.exclude_temp)：引数の続き。
    • def _has_stat_range_filter(self):：サイズ/日付範囲フィルタがあるか確認する。
    • return any(v is not None for v in (：どれか1つでも指定されていればTrue。
    • self.size_min, self.size_max, self.mtime_from, self.mtime_to,：サイズと更新日時。
    • self.ctime_from, self.ctime_to))：作成日時。
    • def _should_skip_stat_for_cap(self):：stat()を省略してよいか判定する。
    • """修正(v0.9.0): 保持上限に既に達しており、かつサイズ/日付範囲での：保持上限に達している場合の最適化。
    • 絞り込みが指定されていない場合、stat()を呼ばずにダミーのサイズ・：stat()を呼ばずにダミーで済ませる。
    • 更新日時で結果を積んでよいかどうかを判定する。範囲フィルタが：範囲フィルタがある場合はstat()が必要。
    • 指定されている場合はstat()の結果自体がマッチ判定に必要なため、：範囲フィルタがある場合は省略できない。
    • 上限超過後もスキップしない(正確性を優先する)。"""：正確性を優先する。
    • return (self.max_result_cap is not None：上限が指定されていて。
    • and self.found_count >= self.max_result_cap：上限に達していて。
    • and not self._has_stat_range_filter())：範囲フィルタがない場合。
    • def _within_range(self, stat, is_dir):：stat結果が範囲内か判定する。
    • if not is_dir:：ファイルの場合。
    • if self.size_min is not None and stat.st_size < self.size_min:：最小サイズ未満なら。
    • return False：範囲外。
    • if self.size_max is not None and stat.st_size > self.size_max:：最大サイズ超過なら。
    • return False：範囲外。
    • elif self.size_min is not None or self.size_max is not None:：フォルダで、サイズ指定がある場合。
    • return False：フォルダはサイズ指定があると対象外。フォルダのサイズは意味が違うため。
    • if self.mtime_from is not None and stat.st_mtime < self.mtime_from:：更新日時が下限未満なら。
    • return False：範囲外。
    • if self.mtime_to is not None and stat.st_mtime > self.mtime_to:：更新日時が上限超過なら。
    • return False：範囲外。
    • if self.ctime_from is not None and stat.st_ctime < self.ctime_from:：作成日時が下限未満なら。
    • return False：範囲外。
    • if self.ctime_to is not None and stat.st_ctime > self.ctime_to:：作成日時が上限超過なら。
    • return False：範囲外。
    • return True：すべて範囲内ならTrue。
    • def run(self):：スレッドの本体。
    • try:：全体を例外処理で囲む。
    • if not os.path.isdir(self.root_dir):：ルートがフォルダでなければ。
    • self.on_error(f"フォルダが見つかりません: {self.root_dir}")：エラーを通知する。
    • return：検索を中止する。
    • batch = []：結果バッチ。
    • last_flush = time.time()：最終フラッシュ時刻。
    • last_examine_report = last_flush：最終走査報告時刻。
    • visited_real = {os.path.normcase(os.path.realpath(self.root_dir))}：訪問済みの実体パス。シンボリックリンクの循環対策。
    • for dirpath, dirnames, filenames in os.walk(：os.walkで走査する。
    • self.root_dir, followlinks=False, onerror=self._on_walk_error):：リンクは追わない。エラーはコールバックへ。
    • if self._stop_event.is_set():：停止指示があれば。
    • break：走査を抜ける。
    • if self.exclude_folders:：除外フォルダがある場合。
    • dirnames[:] = [d for d in dirnames if d.lower() not in self.exclude_folders]：dirnamesを書き換えて、そのフォルダへ潜らないようにする。
    • entries = []：このフォルダのエントリ一覧。
    • if self.search_folders_too:：フォルダも検索対象なら。
    • entries.extend((d, True) for d in dirnames)：フォルダもエントリへ追加。
    • entries.extend((f, False) for f in filenames)：ファイルもエントリへ追加。
    • if self.recursive and dirnames:：再帰検索で、サブフォルダがある場合。
    • safe_dirnames = []：安全なサブフォルダ一覧。
    • for d_idx, d in enumerate(dirnames):：サブフォルダごとに。
    • # 修正(v0.8.2・停止応答性): 1フォルダ直下に大量の：停止応答性の修正。
    • # サブフォルダがある場合(数万件規模)、この：数万件規模のサブフォルダがあると。
    • # realpath()解決ループ自体に時間がかかり、停止操作が：realpath()解決だけで時間がかかる。
    • # 反映されるまでの待ち時間が長くなっていた。500件おきに：500件ごとに停止確認する。
    • # stop_eventを確認し、途中でも速やかに打ち切れるようにする。：途中でも速やかに打ち切る。
    • if d_idx % 500 == 0 and self._stop_event.is_set():：500件ごとに停止確認。
    • dirnames[:] = safe_dirnames：ここまで安全だった分だけ残す。
    • break：ループを抜ける。
    • try:：realpath取得の例外に備える。
    • real = os.path.normcase(os.path.realpath(os.path.join(dirpath, d)))：実体パスを解決する。
    • except OSError:：取得に失敗した場合。
    • continue：そのフォルダはスキップする。
    • if real in visited_real:：既に訪問済みなら。
    • continue：循環を避けるためスキップする。
    • visited_real.add(real)：訪問済みへ追加する。
    • safe_dirnames.append(d)：安全なフォルダへ追加する。
    • else:：forが最後まで回った場合。
    • dirnames[:] = safe_dirnames：安全なフォルダだけを残す。
    • for name, is_dir in entries:：エントリごとに評価する。
    • if self._stop_event.is_set():：停止指示があれば。
    • break：抜ける。
    • if self._wait_while_paused():：一時停止中は待つ。停止したらTrue。
    • break：停止したら抜ける。
    • full_path = os.path.join(dirpath, name)：フルパスを作る。
    • self.examined_count += 1：走査件数を増やす。
    • self._maybe_throttle(self.examined_count)：低負荷モードならウェイトを入れる。
    • if self.on_examined and (：走査件数の通知条件。
    • self.examined_count % self.EXAMINE_REPORT_EVERY == 0：200件ごと。
    • or time.time() - last_examine_report >= self.BATCH_INTERVAL):：または0.15秒経過。
    • self.on_examined(self.examined_count)：走査件数を通知する。
    • last_examine_report = time.time()：通知時刻を更新する。
    • # 修正7・修正9: 1件のファイル評価で予期しない例外(削除・移動・：1件の例外で全体が止まらないようにする修正。
    • # アクセス拒否等の競合状態を含む)が起きても、そのファイルだけ：そのファイルだけスキップする。
    • # スキップして検索全体は継続する(以前は例外がrun()全体まで：以前は例外が全体まで伝播していた。
    • # 伝播し、検索スレッドがそこで異常終了していた)。：検索スレッドが異常終了していた。
    • try:：1件の評価を例外処理で囲む。
    • matched, content_snippet = self._evaluate(name, full_path, is_dir)：1エントリを評価する。
    • except Exception as exc:：予期しない例外を捕捉する。
    • log_error("検索(エントリ評価)", f"{full_path}: {exc}")：ログに残す。
    • continue：次のエントリへ。
    • if not matched:：一致しなければ。
    • continue：次のエントリへ。
    • # 修正(v0.9.0・大量ファイル環境の走査最適化): 既にUI保持：大量ファイル環境の最適化。
    • # 上限を超えている場合(サイズ/日付範囲フィルタが無い時に限る)、：上限超過時はstat()を省略する。
    • # os.stat()呼び出し自体を省略する。上限超過分はどのみち：上限超過分はUIで捨てられる。
    • # UI側で捨てられ、正確なサイズ・更新日時は使われないため、：正確な値が使われないため。
    • # ここでの省略は結果の正しさに影響しない。：結果の正しさに影響しない。
    • if self._should_skip_stat_for_cap():：stat()を省略してよい場合。
    • size, mtime = (None, '')：ダミーのサイズと更新日時。
    • else:：stat()が必要な場合。
    • try:：stat()の例外に備える。
    • stat = os.stat(full_path)：stat()を取得する。
    • if self._has_stat_range_filter() and not self._within_range(stat, is_dir):：範囲フィルタがあり、範囲外なら。
    • continue：次のエントリへ。
    • size = None if is_dir else stat.st_size：フォルダならサイズはNone。
    • mtime = datetime.fromtimestamp(stat.st_mtime).strftime("%Y-%m-%d %H:%M")：更新日時を文字列化する。
    • except OSError:：stat()に失敗した場合。
    • if self._has_stat_range_filter():：範囲フィルタがある場合。
    • continue：範囲判定ができないのでスキップする。
    • size, mtime = (None, '')：範囲フィルタがなければダミーで続行する。
    • self.found_count += 1：見つかった件数を増やす。
    • batch.append({：結果をバッチへ追加する。
    • 'name': name, 'dir': dirpath, 'size': size, 'mtime': mtime,：名前・フォルダ・サイズ・更新日時。
    • 'path': full_path, 'is_dir': is_dir, 'content_snippet': content_snippet,：パス・フォルダ判定・内容スニペット。
    • })：辞書の終わり。
    • now = time.time()：現在時刻。
    • if len(batch) >= self.BATCH_SIZE or now - last_flush >= self.BATCH_INTERVAL:：バッチが50件か、0.15秒経過したら。
    • self.on_result_batch(batch)：バッチをUIへ送る。
    • batch = []：バッチを空にする。
    • last_flush = now：フラッシュ時刻を更新する。
    • if not self.recursive:：再帰検索でなければ。
    • break：1階層だけで終わる。
    • if batch:：残っているバッチがあれば。
    • self.on_result_batch(batch)：最後に送る。
    • if self.on_examined:：走査件数の最終通知。
    • self.on_examined(self.examined_count)：最終件数を通知する。
    • self.on_done(self._stop_event.is_set())：完了を通知する。引数は停止されたかどうか。
    • except Exception as exc:：全体の予期しない例外。
    • log_error("検索", str(exc))：ログに残す。
    • self.on_error(str(exc))：エラーを通知する。
    • class ParallelSearchWorker(SearchWorker):：並列検索ワーカー。SearchWorkerを継承する。
    • """SearchWorkerを継承し、フォルダ走査(os.walk)自体はメインスレッドのまま：フォルダ走査はメインスレッドのまま。
    • (シンボリックリンクの循環対策はSearchWorkerと同じロジックを流用)、：循環対策は同じロジック。
    • 各フォルダのエントリ評価(拡張子/キーワード判定・内容検索・stat呼び出し)を：エントリ評価だけを並列化する。
    • ThreadPoolExecutorで並列化する。stat()やファイル読み込み(内容検索)はGILを：stat()やファイル読み込みはI/O待ちが多い。
    • 解放するI/O待ちが多いため、スレッド並列化の効果が出やすい。：I/O待ちが多いので並列化の効果が出やすい。
    • 走査対象フォルダ数が少ない場合はスレッドプールの起動・同期コストの方が：フォルダ数が少ないと並列化の方が遅くなる。
    • 大きくなるため、直下のサブフォルダ数がSUBDIR_PARALLEL_THRESHOLD未満なら：閾値未満なら単一スレッドへフォールバックする。
    • 自動的に単一スレッド処理(SearchWorker.run())にフォールバックする。：単一スレッド処理へフォールバックする。
    • found_count/examined_countの更新はすべてメインスレッド側(_drain_futures)で：集計はメインスレッド側だけが行う。
    • 行い、ワーカースレッド側は結果をタプルで返すだけにすることで、：ワーカー側は結果を返すだけ。
    • ロックなしでもスレッドセーフに集計できるようにしている。：ロックなしで集計できるようにしている。
    • """：docstringの締め。
    • SUBDIR_PARALLEL_THRESHOLD = 4：並列化するかどうかの閾値。直下のサブフォルダが4つ以上なら並列化する。
    • MAX_PENDING_FUTURES_FACTOR = 4：溜め込むFutureの上限の係数。
    • # 修正(v0.9.0・検索処理の並列化改善): 従来は「1フォルダ＝1タスク」で：並列化の改善。
    • # ThreadPoolExecutorへ投入していたため、ファイル数が極端に多い1つの：1フォルダに数万件あると偏りが出る。
    • # フォルダ(例: 数万件のログファイルが並ぶフォルダ)があると、そのタスク：そのタスクだけが長くなる。
    • # だけ完了までに長時間かかり、他のワーカースレッドがアイドルのまま：他のワーカーが暇になる。
    • # 待たされる(負荷の偏り)ことがあった。1タスクに含めるエントリ数の：負荷の偏りがあった。
    • # 上限を設け、これを超える場合はフォルダ内であっても複数タスクに：1タスクのエントリ数に上限を設ける。
    • # 分割して投入することで、ワーカー間の負荷を均す。：複数タスクに分割- MAX_ENTRIES_PER_TASK = 500：1タスクに含めるエントリ数の上限。超えたらフォルダ内でも分割する。
    • def _worker_count(self):：並列ワーカー数を決める関数。
    • # 修正(v0.9.6・並列スレッド数の設定/低負荷モード): ユーザーが明示的に：ユーザー指定が最優先。
    • # 並列スレッド数を指定している場合はそれを最優先する。指定が無い：指定がない場合は自動算出。
    • # (0)場合は従来通りCPU数から自動算出するが、低負荷モードがONの：低負荷モードは控えめにする。
    • # 場合は控えめな値(CPU数と同程度)に抑える。：低負荷モードの説明。
    • override = getattr(self, 'parallel_worker_count', 0) or 0：ユーザー指定を取得。なければ0。
    • if override > 0:：指定がある場合。
    • return max(1, min(64, override))：1〜64にクランプして返す。
    • cpu_count = DYNAMIC_UI_PARAMS.get('cpu_count') or (os.cpu_count() or 4)：CPU数を取得。取れなければ4。
    • if getattr(self, 'low_load_mode', False):：低負荷モードなら。
    • return max(1, min(8, cpu_count))：最大8までに抑える。
    • return max(2, min(16, cpu_count * 2))：通常はCPU数の2倍、2〜16の範囲。
    • def _should_parallelize(self):：並列化する価値があるか判定する。
    • """root_dir直下のサブフォルダ数を軽くカウントし、並列化する価値が：直下のサブフォルダ数を数える。
    • あるかどうかを判定する(ここではSUBDIR_PARALLEL_THRESHOLD件見つかった：閾値に達したら打ち切るため軽量。
    • 時点で打ち切るため、直下に大量のサブフォルダがあっても軽量)。"""：大量にあっても軽い。
    • try:：走査の例外に備える。
    • count = 0：サブフォルダ数のカウンタ。
    • with os.scandir(self.root_dir) as it:：scandirで直下だけ見る。
    • for entry in it:：エントリごとに。
    • if self._stop_event.is_set():：停止したら。
    • return False：並列化しない。
    • try:：判定の例外に備える。
    • if entry.is_dir(follow_symlinks=False):：フォルダか確認。リンクは追わない。
    • count += 1：カウントを増やす。
    • if count >= self.SUBDIR_PARALLEL_THRESHOLD:：閾値に達したら。
    • return True：並列化する。
    • except OSError:：取得に失敗した場合。
    • continue：次のエントリへ。
    • return False：閾値に達しなければ並列化しない。
    • except OSError:：scandir自体に失敗した場合。
    • return False：並列化しない。
    • def run(self):：並列ワーカーの本体。
    • try:：全体を例外処理で囲む。
    • if not os.path.isdir(self.root_dir):：ルートがフォルダでなければ。
    • self.on_error(f"フォルダが見つかりません: {self.root_dir}")：エラーを通知する。
    • return：中止する。
    • if not self._should_parallelize():：並列化する価値がなければ。
    • # サブフォルダが少ない小規模な検索では、スレッドプールの：小規模ではプールの起動コストの方が大きい。
    • # 起動コストの方が上回るため従来の単一スレッド処理を使う。：単一スレッド処理へフォールバックする。
    • super().run()：親クラスのrun()を使う。
    • return：ここで終わる。
    • self._run_parallel()：並列処理を実行する。
    • except Exception as exc:：予期しない例外。
    • log_error("検索", str(exc))：ログに残す。
    • self.on_error(str(exc))：エラーを通知する。
    • def _evaluate_batch(self, dirpath, entries):：ワーカースレッドで1フォルダ分を評価する関数。
    • """ワーカースレッドで実行される。1フォルダ分のエントリをまとめて評価し、：ワーカー側の処理。
    • (走査件数の増分, 一致したエントリのdictリスト)を返す。selfの状態は：返り値は増分と一致リスト。
    • 読み取り専用でしか使わない(found_count/examined_countは書き換えない)。"""：集計は書き換えない。
    • examined_delta = 0：このタスクの走査件数。
    • matched_entries = []：一致したエントリのリスト。
    • for name, is_dir in entries:：エントリごとに。
    • if self._stop_event.is_set():：停止したら。
    • break：抜ける。
    • full_path = os.path.join(dirpath, name)：フルパスを作る。
    • examined_delta += 1：走査件数を増やす。
    • self._maybe_throttle(examined_delta)：低負荷モードならウェイトを入れる。
    • # 修正7・修正9: 通常検索と同様、1件の評価失敗でスレッドプールの：1件の失敗でタスク全体が止まらないようにする。
    • # ワーカータスク全体が異常終了しないようにする：コメント。
    • try:：1件の評価を例外処理で囲む。
    • matched, content_snippet = self._evaluate(name, full_path, is_dir)：1エントリを評価する。
    • except Exception as exc:：予期しない例外。
    • log_error("並列検索(エントリ評価)", f"{full_path}: {exc}")：ログに残す。
    • continue：次のエントリへ。
    • if not matched:：一致しなければ。
    • continue：次のエントリへ。
    • # 修正(v0.9.0): found_countの更新は_drain_futures()側(メインスレッド)：found_countはメインスレッド側で更新する。
    • # でのみ行われるため、ここでの参照はスナップショット的な近似値になるが、：ここでの参照は近似値。
    • # 「上限超過後は省略してよい」という最適化の判定用途としては十分。：最適化の判定には十分。
    • # 範囲フィルタが指定されている場合は判定にstat結果が必須のため省略しない。：範囲フィルタがある場合は省略しない。
    • if (self.max_result_cap is not None and self.found_count >= self.max_result_cap：上限に達していて。
    • and not self._has_stat_range_filter()):：範囲フィルタがない場合。
    • size, mtime = (None, '')：ダミーで済ませる。
    • else:：stat()が必要な場合。
    • try:：stat()の例外に備える。
    • stat = os.stat(full_path)：stat()を取得する。
    • if self._has_stat_range_filter() and not self._within_range(stat, is_dir):：範囲外なら。
    • continue：次のエントリへ。
    • size = None if is_dir else stat.st_size：フォルダならサイズはNone。
    • mtime = datetime.fromtimestamp(stat.st_mtime).strftime("%Y-%m-%d %H:%M")：更新日時を文字列化する。
    • except OSError:：stat()に失敗した場合。
    • if self._has_stat_range_filter():：範囲フィルタがある場合。
    • continue：範囲判定ができないのでスキップする。
    • size, mtime = (None, '')：範囲フィルタがなければダミーで続行する。
    • matched_entries.append({：一致したエントリを追加する。
    • 'name': name, 'dir': dirpath, 'size': size, 'mtime': mtime,：名前・フォルダ・サイズ・更新日時。
    • 'path': full_path, 'is_dir': is_dir, 'content_snippet': content_snippet,：パス・フォルダ判定・スニペット。
    • })：辞書の終わり。
    • return examined_delta, matched_entries：増分と一致リストを返す。
    • def _drain_futures(self, pending, batch, last_flush_holder, wait_all):：完了したFutureを回収して集計する関数。
    • """完了済みフューチャーの結果をメインスレッドで集計・バッチ発行する。：メインスレッド側の集計。
    • wait_all=Falseの場合は最低1件完了するまで待ち、pendingが減りすぎない：wait_all=Falseなら1件完了まで待つ。
    • 程度に間引きながら回収する(溜め込みすぎによるメモリ膨張を防ぐ)。"""：溜め込みすぎを防ぐ。
    • if not pending:：溜まっているものがなければ。
    • return：何もしない。
    • if wait_all:：すべて待つ場合。
    • done_set, _ = concurrent.futures.wait(pending)：すべて完了するまで待つ。
    • else:：1件でもよい場合。
    • done_set, _ = concurrent.futures.wait(：1件完了まで待つ。
    • pending, return_when=concurrent.futures.FIRST_COMPLETED)：最初の完了で返す。
    • for fut in done_set:：完了したものごとに。
    • pending.discard(fut)：待機中から外す。
    • try:：結果取得の例外に備える。
    • examined_delta, matched_entries = fut.result()：結果を取得する。
    • except Exception as exc:：タスク側で例外があった場合。
    • log_error("並列検索", str(exc))：ログに残す。
    • continue：次のFutureへ。
    • self.examined_count += examined_delta：走査件数を加算する。ここが唯一の集計場所。
    • if self.on_examined and (：走査件数の通知条件。
    • self.examined_count % self.EXAMINE_REPORT_EVERY == 0：200件ごと。
    • or time.time() - last_flush_holder[1] >= self.BATCH_INTERVAL):：または0.15秒経過。
    • self.on_examined(self.examined_count)：走査件数を通知する。
    • for entry in matched_entries:：一致したエントリごとに。
    • self.found_count += 1：見つかった件数を増やす。
    • batch.append(entry)：バッチへ追加する。
    • now = time.time()：現在時刻。
    • if len(batch) >= self.BATCH_SIZE or now - last_flush_holder[0] >= self.BATCH_INTERVAL:：バッチが溜まったら。
    • self.on_result_batch(list(batch))：バッチをUIへ送る。
    • batch.clear()：バッチを空にする。
    • last_flush_holder[0] = now：フラッシュ時刻を更新する。
    • last_flush_holder[1] = now：走査通知時刻も更新する。
    • def _run_parallel(self):：並列走査の本体。
    • worker_count = self._worker_count()：ワーカー数を決める。
    • executor = concurrent.futures.ThreadPoolExecutor(max_workers=worker_count)：スレッドプールを作る。
    • batch = []：結果バッチ。
    • # [0]=結果バッチの最終フラッシュ時刻, [1]=走査件数通知の最終時刻：last_flush_holderの意味。
    • last_flush_holder = [time.time(), time.time()]：最終時刻のホルダー。
    • pending_futures = set()：待機中のFutureの集合。
    • max_pending = max(worker_count * self.MAX_PENDING_FUTURES_FACTOR, worker_count)：溜め込む上限。ワーカー数の4倍。
    • try:：全体を例外処理で囲む。
    • visited_real = {os.path.normcase(os.path.realpath(self.root_dir))}：訪問済みの実体パス。
    • for dirpath, dirnames, filenames in os.walk(：os.walkで走査する。
    • self.root_dir, followlinks=False, onerror=self._on_walk_error):：リンクは追わない。
    • if self._stop_event.is_set():：停止したら。
    • break：抜ける。
    • if self._wait_while_paused():：一時停止中は待つ。
    • break：停止したら抜ける。
    • if self.exclude_folders:：除外フォルダがある場合。
    • dirnames[:] = [d for d in dirnames if d.lower() not in self.exclude_folders]：潜らないようにする。
    • entries = []：このフォルダのエントリ一覧。
    • if self.search_folders_too:：フォルダも対象なら。
    • entries.extend((d, True) for d in dirnames)：フォルダを追加。
    • entries.extend((f, False) for f in filenames)：ファイルを追加。
    • if self.recursive and dirnames:：再帰検索でサブフォルダがある場合。
    • safe_dirnames = []：安全なサブフォルダ一覧。
    • for d_idx, d in enumerate(dirnames):：サブフォルダごとに。
    • # 修正(v0.8.2・停止応答性): SearchWorker.run()と同様、：停止応答性の修正。
    • # 大量サブフォルダのrealpath解決中でも停止操作に速やかに応答する。：大量サブフォルダでも速やかに止まる。
    • if d_idx % 500 == 0 and self._stop_event.is_set():：500件ごとに停止確認。
    • break：抜ける。
    • try:：realpath取得の例外に備える。
    • real = os.path.normcase(os.path.realpath(os.path.join(dirpath, d)))：実体パスを解決する。
    • except OSError:：取得に失敗した場合。
    • continue：スキップする。
    • if real in visited_real:：訪問済みなら。
    • continue：循環を避けるためスキップする。
    • visited_real.add(real)：訪問済みへ追加する。
    • safe_dirnames.append(d)：安全なフォルダへ追加する。
    • dirnames[:] = safe_dirnames：安全なフォルダだけを残す。
    • if entries and not self._stop_event.is_set():：エントリがあり、停止していなければ。
    • # 修正(v0.9.0): 1フォルダのエントリ数がMAX_ENTRIES_PER_TASKを：エントリ数が多ければ分割する。
    • # 超える場合、複数タスクに分割して投入する(通常規模の：通常規模では1タスクのまま。
    • # フォルダでは従来どおり1タスクのままで、分割による：分割のオーバーヘッドは発生しない。
    • # オーバーヘッドは発生しない)。：コメント。
    • if len(entries) > self.MAX_ENTRIES_PER_TASK:：上限を超えたら。
    • for i in range(0, len(entries), self.MAX_ENTRIES_PER_TASK):：500件ずつ。
    • chunk = entries[i:i + self.MAX_ENTRIES_PER_TASK]：チャンクを切り出す。
    • pending_futures.add(executor.submit(self._evaluate_batch, dirpath, chunk))：タスクとして投入する。
    • else:：上限以内なら。
    • pending_futures.add(executor.submit(self._evaluate_batch, dirpath, entries))：1タスクとして投入する。
    • while len(pending_futures) >= max_pending and not self._stop_event.is_set():：溜まりすぎたら。
    • self._drain_futures(pending_futures, batch, last_flush_holder, wait_all=False)：1件以上回収して減らす。
    • if not self.recursive:：再帰でなければ。
    • break：1階層だけで終わる。
    • self._drain_futures(pending_futures, batch, last_flush_holder, wait_all=True)：残りをすべて回収する。
    • if batch:：残っているバッチがあれば。
    • # 停止指示があっても、それまでに集まった分は表示に反映する：停止しても集まった分は表示する。
    • self.on_result_batch(batch)：最後に送る。
    • if self.on_examined:：走査件数の最終通知。
    • self.on_examined(self.examined_count)：最終件数を通知する。
    • self.on_done(self._stop_event.is_set())：完了を通知する。
    • finally:：後始末は必ず行う。
    • # 修正3: stop()された場合でも、実行中/未実行のFutureを確実に片付ける。：停止時もFutureを片付ける。
    • # cancel_futures=Trueで未着手のFutureはキャンセルし、wait=Falseにすることで：未着手はキャンセルする。
    • # 呼び出し元(このrunメソッド)のスレッドはここでブロックしない：ここではブロックしない。
    • # (実行中タスクはバックグラウンドで完了後に自然にスレッドプールが閉じる)。：実行中タスクはバックグラウンドで終わる。
    • # 結果コールバック(on_result_batch等)はメインスレッド側で：コールバックはメインスレッド側で処理される。
    • # self.after(0, ...)経由・かつタブ生存確認(_tab_alive)込みで処理されるため、：タブ生存確認込みで処理される。
    • # 停止後に配信が遅れても破棄済みUIへアクセスすることはない。：破棄済みUIへアクセスすることはない。
    • executor.shutdown(wait=False, cancel_futures=True)：プールを閉じる。
## 第8章：総件数カウントとFTS5用クエリ
    • class ScanCountWorker(threading.Thread):：総エントリ数を数えるだけの軽量ワーカー。
    • """検索対象フォルダの総エントリ数(ファイル+フォルダ)を、名前の一致判定や：一致判定やstat()はしない。
    • stat()呼び出しを行わずに数えるだけの軽量ワーカー。：数えるだけ。
    • 検索速度履歴に頼らなくても、その場でリアルタイムに検索の分母(総件数)を：分母を実測する。
    • 計算し、SearchWorkerの examined_count と組み合わせて実測ベースの：examined_countと組み合わせて進捗率を出す。
    • 進捗率を出せるようにするために使う。"""：進捗率のため。
    • UPDATE_EVERY = 300：途中報告の間隔。
    • def __init__(self, root_dir, recursive, search_folders_too, include_hidden,：コンストラクタ。
    • exclude_folders=None, on_update=None, on_done=None):：引数の続き。
    • super().__init__(daemon=True)：デーモンスレッドとして初期化。
    • self.root_dir = root_dir：ルートを保存。
    • self.recursive = recursive：再帰の有無。
    • self.search_folders_too = search_folders_too：フォルダも数えるか。
    • self.include_hidden = include_hidden：隠しを含めるか。なお、このクラス内ではinclude_hiddenは保持されるだけで、実際のカウント時の除外には使われていない点に注意。検索本体側の条件と分母がずれる可能性がある。
    • self.exclude_folders = exclude_folders or set()：除外フォルダ。
    • self.on_update = on_update：途中報告のコールバック。
    • self.on_done = on_done：完了時のコールバック。
    • self._stop_event = threading.Event()：停止用のイベント。
    • def stop(self):：停止を依頼する。
    • self._stop_event.set()：停止イベントを立てる。
    • def run(self):：本体。
    • count = 0：カウントの初期値。
    • try:：全体を例外処理で囲む。
    • visited_real = {os.path.normcase(os.path.realpath(self.root_dir))}：訪問済みの実体パス。
    • for dirpath, dirnames, filenames in os.walk(：os.walkで走査する。
    • self.root_dir, followlinks=False, onerror=lambda exc: None):：エラーは無視する。分母の計算で止まらないため。
    • if self._stop_event.is_set():：停止したら。
    • break：抜ける。
    • if self.exclude_folders:：除外フォルダがある場合。
    • dirnames[:] = [d for d in dirnames if d.lower() not in self.exclude_folders]：潜らないようにする。
    • if self.search_folders_too:：フォルダも数える場合。
    • count += len(dirnames)：フォルダ数を加算する。
    • count += len(filenames)：ファイル数を加算する。
    • if self.on_update and count % self.UPDATE_EVERY < (len(dirnames) + len(filenames)):：300件ごとに途中報告する条件。countが300の倍数をまたいだかをこのフォルダの増分で判定している。
    • self.on_update(count)：途中件数を通知する。
    • if self.recursive and dirnames:：再帰でサブフォルダがある場合。
    • safe_dirnames = []：安全なサブフォルダ一覧。
    • for d_idx, d in enumerate(dirnames):：サブフォルダごとに。
    • # 修正(v0.8.2・停止応答性): 大量サブフォルダのrealpath解決中でも：停止応答性の修正。
    • # 停止操作に速やかに応答する(補助ワーカーだが検索本体の：補助ワーカーでも止め忘れを防ぐ。
    • # 進捗表示に使われるため、止め忘れによる待ち時間を防ぐ)。：進捗表示に使われるため。
    • if d_idx % 500 == 0 and self._stop_event.is_set():：500件ごとに停止確認。
    • break：抜ける。
    • try:：realpath取得の例外に備える。
    • real = os.path.normcase(os.path.realpath(os.path.join(dirpath, d)))：実体パスを解決する。
    • except OSError:：取得に失敗した場合。
    • continue：スキップする。
    • if real in visited_real:：訪問済みなら。
    • continue：循環を避けるためスキップする。
    • visited_real.add(real)：訪問済みへ追加する。
    • safe_dirnames.append(d)：安全なフォルダへ追加する。
    • dirnames[:] = safe_dirnames：安全なフォルダだけを残す。
    • if not self.recursive:：再帰でなければ。
    • break：1階層だけで終わる。
    • except Exception as exc:：予期しない例外。
    • # 修正1: 進捗の分母を数えるだけの補助ワーカーのため致命的ではないが、：分母の計算だけなので致命的ではない。
    • # 原因不明の不具合調査ができるようログには残す：ログは残す。
    • log_error("総件数カウント", f"{self.root_dir}: {exc}\n{traceback.format_exc()}")：スタックトレース付きでログに残す。
    • if self.on_done:：完了時のコールバックがある場合。
    • self.on_done(count, self._stop_event.is_set())：件数と停止したかどうかを通知する。
    • def _fts5_phrase_query(keyword):：FTS5のtrigram用フレーズクエリを作る関数。
    • """FTS5のtrigramトークナイザで部分一致(LIKE '%keyword%'相当)検索を行うための：trigramで部分一致を実現する。
    • フレーズクエリ文字列を組み立てる。キーワード全体をダブルクォートで囲むことで：キーワード全体をダブルクォートで囲む。
    • フレーズ検索となり、トライグラムが連続して一致する箇所=部分文字列一致として：トライグラムが連続して一致する箇所が部分文字列一致になる。
    • 扱われる。フレーズ内のダブルクォートはFTS5の構文上二重にエスケープする。"""：ダブルクォートは二重にエスケープする。
    • escaped = keyword.replace('"', '""')：ダブルクォートを二重にする。
    • return f'"{escaped}"'：全体をダブルクォートで囲んで返す。
## 第9章：IndexManager - 初期化とDB管理の土台
    • class IndexManager:：インデックスDBを管理するクラス。
    • FETCH_CHUNK = 500：検索結果の取得単位。
    • # 修正(v0.9.0・検索キャッシュの最適化): count_candidates()の短期キャッシュの：候補件数の短期キャッシュ。
    • # 有効期間(秒)。キーワード欄への連続入力(live_reindex)のような短時間の：連続入力時の再問い合わせを吸収する。
    • # 再問い合わせを吸収できれば十分なため、短めの値にとどめる：短めの値にとどめる。
    • # (長すぎるとインデックス更新直後に古い件数を見せてしまう恐れがあるため)。：長すぎると古い件数を見せる恐れがある。
    • CANDIDATE_CACHE_TTL_SEC = 3.0：キャッシュの有効期間は3秒。
    • def __init__(self, db_path=INDEX_DB_PATH):：コンストラクタ。
    • self.db_path = db_path：DBパスを保存。
    • self.dirty = False：変更があったかどうかのフラグ。
    • # 現在アクティブな書き込み系ワーカー(インデックス作成/VACUUM等)の数。：書き込み中ワーカーの数。
    • # 0より大きい間は、生DBファイルをそのまま読み込む暗号化処理を実行してはいけない：書き込み中は暗号化処理をしてはいけない。
    • # (書き込み途中のファイルを読むとDBが破損するため)。：書き込み途中のファイルを読むと破損するため。
    • self._active_writers = 0：書き込み中ワーカー数の初期値。
    • self._active_writers_lock = threading.Lock()：書き込み数への並行アクセス用のロック。
    • # 修正(v0.8.5・終了処理の安全性): IndexBuildWorker等、DBへの書き込みを：終了処理の安全性の修正。
    • # 行うワーカーをここへ登録しておくことで、アプリ終了時(shutdown())に：ワーカーを登録しておく。
    • # 「自動インデックス更新ワーカー」以外――IndexManagerDialogから手動で：手動作成のワーカーも含めて止められるようにする。
    • # 開始されたインデックス作成など――も含めて確実にstop()を呼びかけられる：確実に停止依頼できるようにする。
    • # ようにする。従来はapp側が自分で起動したワーカー(_auto_index_worker)：従来は自動更新ワーカーしか止められなかった。
    • # しか終了時に停止依頼できず、ダイアログ経由の作成中インデックスは：ダイアログ経由の作成中インデックスは止められなかった。
    • # 完了を待つ以外に手段が無かった(大きなフォルダだと終了が長時間：大きなフォルダだと終了が長く待たされる。
    • # 待たされる)。stop()は依頼のみで即時終了は保証しないため、実際の：stop()は依頼のみ。
    • # 完了待ちは従来どおりhas_active_writers()で行う。：完了待ちはhas_active_writers()で行う。
    • # list_roots()はタブ切り替えやキーワード候補取得などで何度も呼ばれるため、：list_roots()は頻繁に呼ばれる。
    • # 結果をキャッシュしておき、index_metaに変更が入るタイミング：結果をキャッシュしておく。
    • # (インデックス作成完了・フォルダ削除など)でのみ_invalidate_roots_cache()：変更があったときだけキャッシュを破棄する。
    • # を呼んで作り直す。：作り直す。
    • self._roots_cache = None：ルート一覧のキャッシュ。
    • self._roots_cache_lock = threading.Lock()：キャッシュ用のロック。
    • # 修正(v0.9.0・検索キャッシュの最適化): count_candidates()は、：候補件数のキャッシュ。
    • # ユーザーが検索条件欄を編集するたび(live_reindex機能や、複数フォルダ：編集のたびに繰り返し呼ばれる。
    • # 検索の各フォルダ処理開始時など)に同一条件で繰り返し呼ばれやすい。：同一条件で繰り返し呼ばれる。
    • # 同一の(root, scope_dir, keyword, match_mode, search_folders_too)に：同一条件ならキャッシュを使える。
    • # 対する結果を短時間(CANDIDATE_CACHE_TTL_SEC)だけキャッシュし、：短時間だけキャッシュする。
    • # 無駄なSQL発行を削減する。インデックスの内容が変わりうるタイミング：無駄なSQLを減らす。
    • # (インデックス作成完了・フォルダ削除等、_invalidate_roots_cache()が：内容が変わりうるタイミングで破棄する。
    • # 呼ばれるタイミングと同じ)でキャッシュ全体を破棄し、古い件数を：古い件数を返し続けないようにする。
    • # 返し続けないようにする。：コメント。
    • self._candidate_count_cache = {}：候補件数のキャッシュ。
    • self._candidate_count_cache_lock = threading.Lock()：キャッシュ用のロック。
    • # 修正(v0.9.7): 作成中(index_metaへの登録前)のフォルダ。自動最適化が：作成中のフォルダの印。
    • # 「未登録フォルダの行」として作成途中の行を削除しないようにするための印。：作成途中の行を消さないようにする。
    • self._building_roots = set()：作成中のフォルダの集合。
    • self._building_lock = threading.Lock()：作成中集合用のロック。
    • self._orphan_cleanup_lock = threading.Lock()：孤立行削除用のロック。
    • # FTS5のtrigramトークナイザが使えるかどうか。SQLiteのビルドによっては：FTS5が使えるかどうかのフラグ。
    • # trigramトークナイザが同梱されていない場合があるため、_ensure_schema()内で：ビルドによっては同梱されていない。
    • # 実際にCREATE VIRTUAL TABLEを試みた結果によってTrue/Falseを確定させる。：実際に試して確定させる。
    • # Falseの場合、search()/count_candidates()は従来のLIKE検索にフォールバックする。：使えなければLIKE検索へ。
    • self.fts5_available = False：初期値はFalse。
    • os.makedirs(os.path.dirname(db_path), exist_ok=True)：DB用のフォルダを作る。
    • # テーブル作成(CREATE TABLE IF NOT EXISTS)自体は軽い処理なので、：テーブル作成は軽い。
    • # IndexManager生成時(=インデックス機能を初めて使った時)に毎回行って問題ない。：生成時に毎回行ってよい。
    • # 一方、PRAGMA integrity_check は大きいDBでは時間がかかるため、：integrity_checkは重い。
    • # maybe_run_integrity_check() で条件付きにする(呼び出しは利用側に委ねる)。：条件付きにする。
    • self._ensure_schema()：スキーマを作成する。
    • def maybe_run_integrity_check(self, first_launch=False, previous_abnormal_exit=False, force=False):：条件付きで整合性チェックを行う関数。
    • """DB整合性チェック(PRAGMA integrity_check)は重いため、以下の場合のみ実行する。：重いので条件付き。
    • ・アプリの初回起動時 (first_launch)：初回起動時。
    • ・前回のセッションが異常終了だった場合 (previous_abnormal_exit)：前回が異常終了だった場合。
    • ・ユーザーがデータベース修復を明示的に実行した場合 (force)：修復を明示的に実行した場合。
    • ・上記に該当しなくても、最後に実行してから：それ以外でも定期的に。
    • INTEGRITY_CHECK_INTERVAL_DAYS 日以上経過している場合(月1回程度の定期チェック)：月1回程度の定期チェック。
    • 通常時(2回目以降の起動・正常終了後)は、DBを開いてテーブル確認をするだけで：通常時はスキップする。
    • integrity_check 自体はスキップする。"""：integrity_check自体はスキップする。
    • if not os.path.exists(self.db_path):：DBがなければ。
    • return：チェックしない。
    • conn = sqlite3.connect(self.db_path, timeout=10)：DBへ接続する。
    • ok = True：結果の初期値。
    • try:：チェック処理を例外処理で囲む。
    • conn.execute("CREATE TABLE IF NOT EXISTS app_meta (key TEXT PRIMARY KEY, value TEXT)")：メタ情報を保存するテーブルを作る。
    • conn.commit()：確定する。
    • should_run = force or first_launch or previous_abnormal_exit：実行すべきか判定する。
    • if not should_run:：まだ実行条件に当てはまらなければ。
    • row = conn.execute(：前回の実行時刻を取得する。
    • "SELECT value FROM app_meta WHERE key = 'last_integrity_check'").fetchone()：前回の時刻を取得する。
    • last_str = row[0] if row else None：値があれば取り出す。
    • if not last_str:：記録がなければ。
    • should_run = True：実行する。
    • else:：記録がある場合。
    • try:：日付パースの例外に備える。
    • last_dt = datetime.strptime(last_str, "%Y-%m-%d %H:%M:%S")：前回の時刻をパースする。
    • should_run = (datetime.now() - last_dt).days >= INTEGRITY_CHECK_INTERVAL_DAYS：間隔が空いていれば実行する。
    • except ValueError:：パースに失敗した場合。
    • should_run = True：実行する。
    • if not should_run:：実行しなくてよい場合。
    • return：ここで終わる。
    • try:：integrity_checkの例外に備える。
    • result = conn.execute("PRAGMA integrity_check").fetchone()：整合性チェックを実行する。
    • ok = bool(result) and result[0] == "ok"：結果がokなら正常。
    • except sqlite3.Error:：実行に失敗した場合。
    • ok = False：異常扱い。
    • conn.execute(：実行時刻を保存する。
    • "INSERT OR REPLACE INTO app_meta (key, value) VALUES ('last_integrity_check', ?)",：前回時刻を更新する。
    • (datetime.now().strftime("%Y-%m-%d %H:%M:%S"),))：現在時刻を保存する。
    • conn.commit()：確定する。
    • finally:：接続は必ず閉じる。
    • conn.close()：接続を閉じる。
    • if ok:：正常なら。
    • return：何もしない。
    • self._quarantine_corrupted_db()：破損していれば退避する。
    • def _quarantine_corrupted_db(self):：破損したDBを退避する関数。
    • """破損したDBファイルを別名で退避し、次回以降は空のDBから作り直せるようにする：別名で退避して空のDBから作り直せるようにする。
    • (インデックスは自動で再構築される)。"""：インデックスは自動で再構築される。
    • log_error("SQLite", f"インデックスDBの破損を検出したため退避します: {self.db_path}")：ログに残す。
    • timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")：タイムスタンプを作る。
    • quarantine_path = f"{self.db_path}.corrupt_{timestamp}"：退避先のパス。
    • try:：退避の例外に備える。
    • os.replace(self.db_path, quarantine_path)：DBファイルを別名へ移動する。
    • except OSError as exc:：移動に失敗した場合。
    • log_error("SQLite", f"破損DBの退避に失敗しました: {exc}")：ログに残す。
    • try:：削除の例外に備える。
    • os.remove(self.db_path)：移動できなければ削除を試みる。
    • except OSError:：削除に失敗した場合。
    • pass：ここでは追加処理をしない。
    • _cleanup_wal_files()：WAL関連ファイルを掃除する。
    • self._ensure_schema()：空のスキーマを作り直す。
    • def begin_write(self):：書き込み開始を宣言する。
    • with self._active_writers_lock:：ロックして。
    • self._active_writers += 1：書き込み中ワーカー数を増やす。
    • def end_write(self):：書き込み終了を宣言する。
    • with self._active_writers_lock:：ロックして。
    • self._active_writers = max(0, self._active_writers - 1)：0未満にならないよう減らす。
    • def has_active_writers(self):：書き込み中のワーカーがあるか確認する。
    • with self._active_writers_lock:：ロックして。
    • return self._active_writers > 0：0より大きければ書き込み中。
    • def mark_building(self, root):：作成開始を宣言する。
    • """作成開始時に呼ぶ。最適化の孤立行削除(短時間)の最中なら終わるまで待つ。"""：最適化中なら終わるまで待つ。
    • with self._orphan_cleanup_lock:：孤立行削除のロックを取る。
    • with self._building_lock:：作成中集合のロックを取る。
    • self._building_roots.add(root)：作成中に追加する。
    • def unmark_building(self, root):：作成終了を宣言する。
    • with self._building_lock:：ロックして。
    • self._building_roots.discard(root)：作成中から外す。
    • def register_worker(self, worker):：書き込み系ワーカーを登録する。
    • """修正(v0.8.5): 書き込み系ワーカー開始時に自身を登録する。"""：ワーカー開始時に登録する。
    • with self._active_writers_lock:：ロックして。
    • if not hasattr(self, '_active_workers'):：集合がなければ。
    • self._active_workers = set()：集合を作る。
    • self._active_workers.add(worker)：ワーカーを追加する。
    • def unregister_worker(self, worker):：ワーカーの登録を解除する。
    • with self._active_writers_lock:：ロックして。
    • if hasattr(self, '_active_workers'):：集合がある場合。
    • self._active_workers.discard(worker)：ワーカーを外す。
    • def stop_all_workers(self):：登録済みワーカーすべてに停止を依頼する。
    • """修正(v0.8.5・終了処理の安全性): 登録済みの書き込み系ワーカーすべてに：終了処理の安全性。
    • 停止を依頼する。呼び出し元(アプリ終了処理)は、これを呼んだ後：呼び出し元はこの後に完了を待つ。
    • has_active_writers()が False になるまで待つことで、DBへの書き込みが：書き込みが止まるまで待つ。
    • 完全に止まってから暗号化・VACUUMへ進めるようにする。"""：書き込みが止まってから暗号化などへ進める。
    • with self._active_writers_lock:：ロックして。
    • workers = list(getattr(self, '_active_workers', ()))：登録済みワーカーの一覧をコピーする。
    • for worker in workers:：ワーカーごとに。
    • try:：停止依頼の例外に備える。
    • worker.stop()：停止を依頼する。
    • except Exception as exc:：停止依頼に失敗した場合。
    • log_error("SQLite", f"終了処理でのワーカー停止に失敗しました: {exc}")：ログに残す。
    • def _connect(self):：DBへの接続を作る関数。PRAGMA調整もここで行う。
    • conn = sqlite3.connect(self.db_path, timeout=30)：30秒のタイムアウトで接続する。
    • conn.execute("PRAGMA journal_mode=WAL")：WALモードにする。
    • # 修正(v0.8.9・検索性能改善/UIフリーズ対策): 複数スレッド(検索・：複数スレッドからのアクセス対策。
    • # インデックス作成・VACUUM等)が同時にDBへアクセスした際、既定では：既定では即座にロックエラーになる。
    • # 即座に"database is locked"エラーになりうる。busy_timeoutを設定：busy_timeoutで待ってから再試行する。
    • # することで、ロック解放を一定時間(ミリ秒)待ってから再試行するように：ロック解放を待つ。
    • # なり、体感のフリーズやエラー発生を減らせる(sqlite3.connect()の：フリーズやエラーを減らす。
    • # timeout引数はPython側の接続確立時のみに効くため、接続後の個々の：接続後のクエリにも効くようPRAGMAで設定する。
    • # クエリ実行時にも効くようPRAGMAとして別途設定する)。：PRAGMAでも設定する。
    • # 修正(v0.9.7): connect(timeout=30)で既に30秒のbusy待ちが設定されているのに、：以前は5秒に短縮されていた問題。
    • # ここで5000(5秒)を指定すると待ち時間が短縮されてしまい、自動インデックス：待ち時間が短くなっていた。
    • # 更新と手動作成が重なった際などに"database is locked"が出やすくなっていた。：重なったときにロックエラーが出やすかった。
    • # connect側と同じ30000msにそろえる。：30秒に揃える。
    • try:：PRAGMA設定の例外に備える。
    • conn.execute("PRAGMA busy_timeout=30000")：30秒待つ。
    • except sqlite3.Error:：失敗しても。
    • pass：致命的ではないので無視する。
    • # 以下は検索・インデックス作成のパフォーマンス向上のためのPRAGMA調整。：パフォーマンス向上のための調整。
    • # synchronous=NORMAL: WALモードでは既定のFULLより緩い同期でも：WALでは緩い同期でも破損しにくい。
    • # クラッシュ時にDB破損を起こさないため、書き込み速度を優先する。：書き込み速度を優先する。
    • # temp_store=MEMORY: ソートや一時テーブルをディスクではなくメモリ上に置く。：一時テーブルをメモリへ。
    • # cache_size: ページキャッシュを増やし、繰り返しアクセスするページの：キャッシュを増やす。
    • # ディスクI/Oを減らす(負値はKB単位。約20MB)。：約20MB。
    • # mmap_size: 対応環境ではメモリマップドI/Oを使い読み込みを高速化する。：メモリマップドI/Oで高速化する。
    • # 未対応/失敗しても致命的ではないため、個別にtry/exceptで無視する。：失敗しても無視する。
    • try:：synchronous設定の例外に備える。
    • conn.execute("PRAGMA synchronous=NORMAL")：同期を緩めにする。
    • except sqlite3.Error:：失敗しても。
    • pass：無視する。
    • try:：temp_store設定の例外に備える。
    • conn.execute("PRAGMA temp_store=MEMORY")：一時データをメモリへ。
    • except sqlite3.Error:：失敗しても。
    • pass：無視する。
    • try:：cache_size設定の例外に備える。
    • conn.execute("PRAGMA cache_size=-20000")：約20MBのキャッシュ。
    • except sqlite3.Error:：失敗しても。
    • pass：無視する。
    • try:：mmap_size設定の例外に備える。
    • conn.execute("PRAGMA mmap_size=268435456")：256MBのメモリマップ。
    • except sqlite3.Error:：失敗しても。
    • pass：無視する。
    • return conn：接続を返す。
    • def _ensure_schema(self):：テーブルとインデックスを作る関数。
    • conn = self._connect()：接続する。
    • try:：スキーマ作成の例外処理。ここではfinallyで接続を閉じる。
    • conn.execute("""：filesテーブルを作る。
    • CREATE TABLE IF NOT EXISTS files (：ファイル一覧のテーブル。
    • path TEXT PRIMARY KEY,：パスが主キー。
    • name TEXT NOT NULL,：ファイル名。
    • dir TEXT NOT NULL,：親フォルダ。
    • size INTEGER,：サイズ。
    • mtime TEXT,：更新日時。
    • is_dir INTEGER NOT NULL,：フォルダかどうか。
    • root TEXT NOT NULL：検索ルート。
    • )：テーブルの終わり。
    • """)：SQLの終わり。
    • conn.execute("CREATE INDEX IF NOT EXISTS idx_files_root ON files(root)")：ルートでの絞り込み用インデックス。
    • conn.execute("CREATE INDEX IF NOT EXISTS idx_files_name ON files(name)")：名前での絞り込み用インデックス。
    • # (root, dir)の複合インデックス。search()やcount_candidates()での：複合インデックスの説明。
    • # 「特定rootのうち特定フォルダ配下」という絞り込みを高速化する。：特定フォルダ配下の絞り込みを速くする。
    • conn.execute("CREATE INDEX IF NOT EXISTS idx_files_root_dir ON files(root, dir)")：複合インデックスを作る。
    • conn.commit()：確定する。
    • # FTS5のtrigramトークナイザによる部分一致検索用の仮想テーブル。：FTS5用の仮想テーブル。
    • # 外部コンテンツ方式(content=)にすると、files.path(TEXT PRIMARY KEY)の：外部コンテンツ方式は使わない理由。
    • # INSERT OR REPLACEでrowidが変化してしまい同期が壊れるため、ここでは：rowidが変化して同期が壊れるため。
    • # 独立したFTS5テーブルとしてpath・nameを保持し、IndexBuildWorker側で：独立したFTS5テーブルとして保持する。
    • # 手動同期する(_sync_fts_batch/_sync_fts_delete参照)。：手動で同期する。
    • # trigramトークナイザ未対応の古いSQLiteビルドでは例外が発生するため、：未対応のビルドでは例外になる。
    • # その場合はfts5_availableをFalseのままにし、LIKE検索にフォールバックする。：LIKE検索へフォールバックする。
    • try:：FTS5テーブル作成の例外に備える。
    • conn.execute("""：FTS5仮想テーブルを作る。
    • CREATE VIRTUAL TABLE IF NOT EXISTS files_fts USING fts5(：FTS5仮想テーブル。
    • name, path UNINDEXED, tokenize='trigram'：nameを検索対象、pathはインデックスしない。
    • )：仮想テーブルの終わり。
    • """)：SQLの終わり。
    • conn.commit()：確定する。
    • self.fts5_available = True：FTS5が使える。
    • except sqlite3.Error as exc:：FTS5が使えない場合。
    • log_error("SQLite", f"FTS5(trigram)が利用できないためLIKE検索にフォールバックします: {exc}")：ログに残す。
    • self.fts5_available = False：FTS5なしとして扱う。
    • conn.execute("""：index_metaテーブルを作る。
    • CREATE TABLE IF NOT EXISTS index_meta (：インデックスのメタ情報。
    • root TEXT PRIMARY KEY,：ルートが主キー。
    • indexed_at TEXT,：インデックス作成時刻。
    • file_count INTEGER,：ファイル数。
    • total_size INTEGER：合計サイズ。
    • )：テーブルの終わり。
    • """)：SQLの終わり。
    • conn.execute("""：access_errorsテーブルを作る。
    • CREATE TABLE IF NOT EXISTS access_errors (：アクセスエラーの記録。
    • root TEXT NOT NULL,：ルート。
    • path TEXT NOT NULL,：パス。
    • type TEXT,：エラーの種類。
    • message TEXT：メッセージ。
    • )：テーブルの終わり。
    • """)：SQLの終わり。
    • conn.execute("CREATE INDEX IF NOT EXISTS idx_access_errors_root ON access_errors(root)")：ルートでの絞り込み用インデックス。
    • conn.commit()：確定する。
    • cols = [row[1] for row in conn.execute("PRAGMA table_info(index_meta)")]：index_metaの列一覧を取得する。
    • if "total_size" not in cols:：古いDBでtotal_sizeがない場合。
    • conn.execute("ALTER TABLE index_meta ADD COLUMN total_size INTEGER DEFAULT 0")：列を追加する。
    • conn.commit()：確定する。
    • finally:：接続は必ず閉じる。
    • conn.close()：接続を閉じる。
    • def save_access_errors(self, root, errors):：アクセスエラーをDBへ保存する。
    • conn = self._connect()：接続する。
    • try:：保存処理の例外に備える。
    • conn.execute("DELETE FROM access_errors WHERE root = ?", (root,))：古い記録を消す。
    • if errors:：エラーがある場合。
    • conn.executemany(：まとめて挿入する。
    • "INSERT INTO access_errors (root, path, type, message) VALUES (?,?,?,?)",：挿入SQL。
    • [(root, e['path'], e.get('type'), e.get('message')) for e in errors])：エラーの一覧をタプル化する。
    • conn.commit()：確定する。
    • self.dirty = True：変更があったことを記録する。
    • except sqlite3.Error as exc:：保存に失敗した場合。
    • log_error("SQLite", f"アクセスエラー保存エラー ({root}): {exc}")：ログに残す。
    • finally:：接続は必ず閉じる。
    • conn.close()：接続を閉じる。
    • def get_access_errors(self, root):：アクセスエラーをDBから取得する。
    • conn = self._connect()：接続する。
    • try:：取得処理。
    • rows = conn.execute(：ルート指定で取得する。
    • "SELECT path, type, message FROM access_errors WHERE root = ?", (root,)：取得SQL。
    • ).fetchall()：すべて取得する。
    • return [{'path': p, 'type': t, 'message': m} for p, t, m in rows]：辞書のリストへ変換する。
    • finally:：接続は必ず閉じる。
    • conn.close()：接続を閉じる。
    • def _invalidate_roots_cache(self):：ルート一覧と候補件数のキャッシュを破棄する。
    • with self._roots_cache_lock:：ロックして。
    • self._roots_cache = None：ルート一覧のキャッシュを破棄する。
    • # 修正(v0.9.0): インデックスの内容が変わりうるタイミング(list_rootsの：候補件数のキャッシュも合わせて破棄する。
    • # キャッシュを破棄するのと同じタイミング)で、候補件数キャッシュも：同じタイミングで破棄する。
    • # 古いまま残らないよう合わせて破棄する。：古いまま残さない。
    • with self._candidate_count_cache_lock:：ロックして。
    • self._candidate_count_cache.clear()：候補件数のキャッシュを破棄する。
    • def list_roots(self):：登録済みルートの一覧を返す。キャッシュあり。
    • with self._roots_cache_lock:：ロックして。
    • if self._roots_cache is not None:：キャッシュがある場合。
    • return self._roots_cache：キャッシュを返す。
    • conn = self._connect()：接続する。
    • try:：取得処理。
    • rows = conn.execute(：ルート一覧を取得する。
    • "SELECT root, file_count, indexed_at, COALESCE(total_size, 0) "：ルート・件数・時刻・合計サイズ。
    • "FROM index_meta ORDER BY root"：ルート順に並べる。
    • ).fetchall()：すべて取得する。
    • finally:：接続は必ず閉じる。
    • conn.close()：接続を閉じる。
    • with self._roots_cache_lock:：ロックして。
    • self._roots_cache = rows：キャッシュへ保存する。
    • return rows：返す。
    • def list_root_names(self):：ルート名だけを返す軽量版。
    • """修正(v0.8.9・起動速度): list_roots()はUI一覧表示用にfile_count・：起動速度のための軽量版。
    • indexed_at・total_sizeまで取得するが、自動インデックス更新キューへの：list_roots()は列が多い。
    • 投入(_refresh_all_known_indexes)のようにroot列の値しか使わない場面では：ルートだけしか使わない場面がある。
    • これらの列は不要である。root列のみを取得する軽量版を用意し、：ルートだけを取得する。
    • 起動直後のprewarm時などで無駄な列走査を避ける。list_roots()のような：無駄な列走査を避ける。
    • キャッシュは持たない(呼び出し頻度が低く、キャッシュの複雑さに見合わないため)。"""：キャッシュは持たない。
    • conn = self._connect()：接続する。
    • try:：取得処理。
    • rows = conn.execute("SELECT root FROM index_meta ORDER BY root").fetchall()：ルートだけを取得する。
    • return [r[0] for r in rows]：リストへ変換する。
    • except sqlite3.Error as exc:：取得に失敗した場合。
    • log_error("SQLite", f"インデックスフォルダ一覧(軽量版)取得エラー: {exc}")：ログに残す。
    • return []：空リストを返す。
    • finally:：接続は必ず閉じる。
    • conn.close()：接続を閉じる。
    • def remove_root(self, root):：ルートのインデックスを削除する。
    • conn = self._connect()：接続する。
    • try:：削除処理。
    • if self.fts5_available:：FTS5が使える場合。
    • conn.execute(：FTS5側も削除する。
    • "DELETE FROM files_fts WHERE path IN (SELECT path FROM files WHERE root = ?)",：対象ルートのパスを削除する。
    • (root,))：ルートを指定する。
    • conn.execute("DELETE FROM files WHERE root = ?", (root,))：本体側も削除する。
    • conn.execute("DELETE FROM index_meta WHERE root = ?", (root,))：メタ情報も削除する。
    • conn.execute("DELETE FROM access_errors WHERE root = ?", (root,))：アクセスエラーも削除する。
    • conn.commit()：確定する。
    • self.dirty = True：変更があったことを記録する。
    • self._invalidate_roots_cache()：キャッシュを破棄する。
    • except sqlite3.Error as exc:：削除に失敗した場合。
    • log_error("SQLite", f"インデックス削除エラー ({root}): {exc}")：ログに残す。
    • raise：呼び出し元へ例外を伝える。
    • finally:：接続は必ず閉じる。
    • conn.close()：接続を閉じる。
    • OPTIMIZE_OFFLINE_ROOT_DAYS = 30：オフラインのルートを削除するまでの日数。30日。
    • def optimize_stale_entries(self, stop_event=None):：インデックスの自動最適化を行う関数。
    • """修正(v0.9.7・インデックスの自動最適化): 起動後しばらくしてから自動実行される：自動実行される軽量な整理処理。
    • 軽量な整理処理(VACUUMのような全体の作り直しはしない)。：全体の作り直しはしない。
    • 1. 登録されていないフォルダ(index_meta無し)の行・アクセスエラー記録を削除する：未登録フォルダの行を削除する。
    • 2. files側に実体の無い検索用(FTS5)の行を削除する(一度に5000件ずつ)：FTS5の孤立行を削除する。
    • 3. フォルダが存在せず、最終更新から30日以上経過した登録を削除する：古いオフライン登録を削除する。
    • (取り外し中のUSB/ネットワークドライブを誤って消さないため、：取り外し中のドライブを誤って消さないため。
    • 直近に更新されたものは対象にしない)：直近のものは対象にしない。
    • 4. クエリプランナの統計更新とWALのチェックポイントを行う：統計更新とチェックポイント。
    • 戻り値: 各項目の削除件数の辞書。呼び出し側はbegin_write()/end_write()で囲むこと：呼び出し側は書き込み宣言で囲むこと。
    • (終了処理が完了を待てるようにするため)。"""：終了処理が完了を待てるようにする。
    • result = {'orphan_files': 0, 'orphan_fts': 0, 'orphan_errors': 0, 'old_roots': 0}：結果の辞書を初期化する。
    • conn = self._connect()：接続する。
    • try:：最適化処理。
    • # 作成中のフォルダの行は「未登録」に見えるので対象から外す(判定と削除は：作成中のフォルダは対象から外す。
    • # 同じロックの中で行い、その間に作成が始まっても行を消さないようにする)：判定と削除を同じロックの中で行う。
    • with self._orphan_cleanup_lock:：孤立行削除のロックを取る。
    • with self._building_lock:：作成中集合のロックを取る。
    • building = list(self._building_roots)：作成中のフォルダ一覧をコピーする。
    • excl = ""：除外条件の初期値。
    • excl_params = []：除外条件のパラメータ。
    • if building:：作成中のフォルダがある場合。
    • excl = " AND root NOT IN (%s)" % ",".join("?" * len(building))：作成中を除外する条件を作る。
    • excl_params = building：パラメータに作成中一覧を設定する。
    • if self.fts5_available:：FTS5が使える場合。
    • conn.execute(：FTS5側の未登録行を削除する。
    • "DELETE FROM files_fts WHERE path IN "：FTS5側の削除。
    • "(SELECT path FROM files WHERE root NOT IN (SELECT root FROM index_meta)"：未登録ルートのパス。
    • + excl + ")", excl_params)：作成中を除外して実行する。
    • cur = conn.execute(：本体側の未登録行を削除する。
    • "DELETE FROM files WHERE root NOT IN (SELECT root FROM index_meta)" + excl, excl_params)：本体側の削除。
    • result['orphan_files'] = max(0, cur.rowcount or 0)：削除件数を記録する。
    • cur = conn.execute(：アクセスエラーの未登録行を削除する。
    • "DELETE FROM access_errors WHERE root NOT IN (SELECT root FROM index_meta)" + excl,：アクセスエラー側の削除。
    • excl_params)：パラメータ。
    • result['orphan_errors'] = max(0, cur.rowcount or 0)：削除件数を記録する。
    • conn.commit()：確定する。
    • if self.fts5_available:：FTS5が使える場合。
    • while True:：孤立したFTS5行がなくなるまで繰り返す。
    • if stop_event is not None and stop_event.is_set():：停止したら。
    • break：抜ける。
    • rows = conn.execute(：実体のないFTS5行を5000件ずつ取得する。
    • "SELECT rowid FROM files_fts WHERE path NOT IN (SELECT path FROM files) LIMIT 5000"：5000件ずつ。
    • ).fetchall()：取得する。
    • if not rows:：なければ。
    • break：抜ける。
    • conn.executemany("DELETE FROM files_fts WHERE rowid = ?", rows)：まとめて削除する。
    • conn.commit()：確定する。
    • result['orphan_fts'] += len(rows)：削除件数を加算する。
    • old_roots = []：古いオフラインルートの一覧。
    • for root, _count, indexed_at, _size in conn.execute(：登録済みルートをすべて取得する。
    • "SELECT root, file_count, indexed_at, COALESCE(total_size, 0) FROM index_meta").fetchall():：取得SQL。
    • if os.path.isdir(root):：フォルダが存在すれば。
    • continue：対象にしない。
    • try:：日付パースの例外に備える。
    • last = datetime.strptime(indexed_at, "%Y-%m-%d %H:%M")：最終更新時刻をパースする。
    • except (TypeError, ValueError):：パースに失敗した場合。
    • continue：対象にしない。
    • if (datetime.now() - last).days >= self.OPTIMIZE_OFFLINE_ROOT_DAYS:：30日以上経過していたら。
    • old_roots.append(root)：削除候補へ追加する。
    • try:：統計更新などの例外に備える。
    • conn.execute("PRAGMA optimize")：クエリプランナの統計を更新する。
    • conn.execute("PRAGMA wal_checkpoint(PASSIVE)")：WALのチェックポイントを行う。
    • except sqlite3.Error:：失敗しても。
    • pass：致命的ではないので無視する。
    • finally:：接続は必ず閉じる。
    • conn.close()：接続を閉じる。
    • for root in old_roots:：古いルートごとに。
    • if stop_event is not None and stop_event.is_set():：停止したら。
    • break：抜ける。
    • try:：削除の例外に備える。
    • self.remove_root(root)：ルートを削除する。
    • result['old_roots'] += 1：削除件数を増やす。
    • except sqlite3.Error as exc:：削除に失敗した場合。
    • log_error("インデックス自動整理", f"{root}: {exc}")：ログに残す。
    • if any(result.values()):：どれか1つでも削除・整理があった場合。
    • self.dirty = True：変更があったことを記録する。
    • self._invalidate_roots_cache()：キャッシュを破棄する。
    • return result：結果の辞書を返す。
# おわりに
今回は、FileSearchExplorerの1493〜3456行目を解説しました。
今回の範囲では、テキスト判定とプレビュー、進捗表示・実行確認・パスを開く処理、隠し/システムファイルと保護フォルダ、危険操作の確認ダイアログ、検索条件のパースとあいまい検索、内容検索、1エントリ評価と検索ワーカー、総件数カウント、FTS5用クエリ、IndexManagerの初期化とDB管理の土台などを扱いました。
特に、UTF-16をバイナリと誤認しないための判定順序、保護フォルダ判定のフェイルクローズとシンボリックリンク対策、1件の例外で検索全体を止めない工夫、並列検索での負荷分散とスレッドセーフな集計、インデックスの自動最適化で作成中の行を消さないためのロックについては、実装上の注意点があります。権限設定や暗号化処理と同じく、コードを書くだけで完全な安全性が保証されるわけではありません。
今後も、コードの動作だけでなく、失敗時の挙動や安全性についても確認しながら解説を続けていきたいと思います。
最後まで読んでいただき、ありがとうございました！
