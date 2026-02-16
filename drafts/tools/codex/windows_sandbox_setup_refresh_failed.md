# Codex の `windows sandbox: setup refresh failed` 原因と結果

## 事象
- Codex のサンドボックス実行で毎回以下が発生。
- `windows sandbox: setup refresh failed with status exit code: 1`

## 原因
- サンドボックス初期化処理が `C:\Windows\Temp` の ACL 更新で失敗していた。
- 具体的には `CreateFileW failed for C:\Windows\Temp` により write ACE 付与ができず、setup refresh がエラー終了していた。

参照:
- `C:\Users\t.arihori\.codex\.sandbox\setup_error.json`
- `C:\Users\t.arihori\.codex\.sandbox\sandbox.log`

## 調査で確認した事実
- サンドボックス内コマンドは失敗、昇格実行は成功した。
- 環境変数で `TMP` が `C:\WINDOWS\TEMP` を向いていた。
- `icacls C:\Windows\Temp` は `Access is denied.` となり、対象 ACL の操作可否に問題があった。

## 実施した対応
1. User 環境変数を変更
- `TMP=C:\Users\t.arihori\AppData\Local\Temp`
- `TEMP=C:\Users\t.arihori\AppData\Local\Temp`

2. タイムゾーン設定を統一
- User 環境変数: `TZ=Asia/Tokyo`
- Windows: `Tokyo Standard Time`（`Asia/Tokyo` 相当）

3. Codex を再起動
- 既存プロセスに残っていた古い `TMP` を解消するため。

## 結果
- サンドボックス内の `shell_command` が正常実行に復旧。
- `git status --short --branch` など基本コマンドが成功。
- 時刻は `+09:00` で確認。

## 補足: `Cannot open external editor` について
表示:
- `Cannot open external editor: set $VISUAL or $EDITOR before starting Codex.`

意味:
- Codex が外部エディタ起動を試みたが、利用するエディタ（`VISUAL` / `EDITOR`）が未設定で開けなかった。

対処例（PowerShell）:
```powershell
[Environment]::SetEnvironmentVariable('VISUAL', 'code --wait', 'User')
[Environment]::SetEnvironmentVariable('EDITOR', 'code --wait', 'User')
```
- 設定後に Codex を再起動すると反映される。

## 備考
- `mise.toml are not trusted` 警告は、今回のサンドボックス障害とは別問題。
