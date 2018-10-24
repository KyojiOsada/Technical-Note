# 1. Type
## 1-1. gzip
Compression Ratio: 3

### 1-1-1. Compression
`$ tar -zcvf dest_file.tar.gz src_file`

### 1-1-2. Progress Compression
`$ tar cf - src_file | pv -s $(du -sb src_file | awk '{print $1}') | gzip > dest.tar.gz`

### 1-1-3. Decompression
`$ tar -zxvf file.tar.gz`

### 1-1-4. Progress Decompression
`$ pv dest.tar.gz | tar zxf -`

## 1-2. bzip2
_Compression Ratio: 2_

### 1-2-1. Compression
`$ tar -jcvf dest_file.tar.bz2 src_file`

### 1-2-2. Progress Compression
`$ tar cf - src_file | pv -s $(du -sb src_file | awk '{print $1}') | bzip2 > dest.tar.bz2`

### 1-2-3. Decompression
`$ tar -jxvf file.tar.bz2`

### 1-2-4. Progress Decompression
`$ pv dest.tar.bz2 | tar xf -`

## 1-3. xz
_Compression Ratio: 1_

### 1-3-1. Compression
`$ tar -Jcvf dest_file.tar.xz src_file`

### 1-3-2. Progress Compression
`$ tar cf - src_file | pv -s $(du -sb src_file | awk '{print $1}') | xz > dest.tar.xz`

### 1-3-3. Decompression
`$ tar -Jxvf file.tar.xz`

### 1-3-4. Progress Decompression
`$ pv dest.tar.xz | tar xf -`

# 2. Option
### --remove-files
_圧縮前のファイルを削除_<br>
`$ tar -Jcvf dest_file.tar.xz src_file --remove-files`

### -a
_圧縮形式を拡張子から自動認識_

### -z
_gzip 処理（拡張子：.gz）_

### -j
_bzip2 処理（拡張子：.bg2）_

### -J
_xz 処理（拡張子：.xz）_

### -c
_圧縮 (tar ファイルを作成)_

### -x
_解凍 (tar ファイルを解凍)_

### -t
_ファイルを表示(tar のファイルリストを表示する)_

### -f
_指定したファイル名を使用_

### -v
_処理したファイルの一覧を表示(verbose)_

