# 1. Install
`$ yum -y install rsync`

# 2. Command
`rsync [option] SROURCE DEST`

# 3. Option
### -r
再帰的に同期

### -t
タイムスタンプを保持

### -a
-rlptgoD の指定と同じ

### -l
シンボリックリンクを保持

### -p
パーミッションを保持

### -g
グループを保持

### -o
オーナーを保持

### -D
デバイスファイルや特殊ファイルを保持

### -v
詳細を画面表示

### -z
圧縮転送（転送データの圧縮のみで、転送先ファイルは圧縮されない）

### --inplace
同期先で一時ファイルをつくらない

### --progress
ファイルごとに進捗を画面表示

### --delete
同期元にないファイルが同期先にある場合は同期先を削除する

### /source/ /dest/
source ディレクトリは作成されず中身だけが同期される

### /source /dest/
source ディレクトリが同期先に作成され同期される

# 3. Warning
転送先の容量を事前確認すること


# 4. Example
_進捗表示、転送先バックアップをとらない_<br>
`$ rsync -rtv --progress --inplace /src_dir ssh_user@xxx.xxx.xxx.xxx:/dst_dir/`<br>
<br>
_nohup & background Execution_<br>
`$ nohup rsync -rt --inplace /src_dir ssh_user@xxx.xxx.xxx.xxx:/dst_dir/ &`
