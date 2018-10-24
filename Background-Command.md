# 1. Command
## 1-1. &
_バックグラウンド実行。これだけでは、SSH ログアウトした場合に、子プロセスのコマンドも終了する_

## 1-2. nohup
_SSH の親プロセスが終了しても、子プロセスのコマンドを終了させない。_

# 2. Example
## 2-1. rsync
_nohup と & を併用しないと、次のコマンド（例：exit）が実行できない_<br>
`$ nohup rsync -rt --inplace /src_dir user@xxx.xxx.xxx.xxx:/dst_dir/ &`

## 2-2. mysql restore
`$ nohup mysql -h xxx.xxx.xxx.xxx -u user -f < db.sql > ./out.log 2> ./error.log &`
