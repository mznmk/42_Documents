# 必要な知識

課題挑戦時(2022/3)における理解をまとめた。  


## 利用可能関数

### open()

pathname で指定されたファイルをオープンする。  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/open.2.html  
```c
```

### close()

ファイルディスクリプターをクローズする。  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/close.2.html  
```c
```

### read()

ファイルディスクリプター (file descriptor) fd から最大 count バイトを  
buf で始まるバッファーへ読み込もうとする。  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/read.2.html  
```c
```

### write()
buf から始まるバッファーから、ファイルディスクリプター fd が参照するファイルへ、  
最大 count バイトを書き込む。  
```c
```

### malloc()

size バイトを割り当て、割り当てられたメモリーに対するポインターを返す。  
メモリーの内容は初期化されない。  
https://linuxjm.osdn.jp/html/LDP_man-pages/man3/malloc.3.html  
```c
```

### free()

ポインター ptr が指すメモリー空間を解放する。  
https://linuxjm.osdn.jp/html/LDP_man-pages/man3/malloc.3.html  
```c
```

### perror()

システムコールやライブラリ関数の呼び出しにおいて、  
最後に発生した エラーに関する説明メッセージを生成し、標準エラー出力に出力する。  
https://linuxjm.osdn.jp/html/LDP_man-pages/man3/perror.3.html  
```c
```

### strerror()

引数 errnum で渡されたエラーコードについての説明が入った文字列へのポインターを返す。  
https://linuxjm.osdn.jp/html/LDP_man-pages/man3/strerror.3.html  
```c
```

### access()

呼び出し元プロセスがファイル pathname にアクセスできるかどうかをチェックする。  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/access.2.html  
```c
//#define FILE_PATH "/path/to/file.txt"

// ファイルが存在し、読み込み権限および実行権限があるかを確認
// 権限の確認がすべてOKに場合には、0 がリターンされる
// 権限がない、またはその他のエラーの場合は -1 がリターンされる
if (access (FILE_PATH, (R_OK | X_OK)))
{
	// ファイルが存在しない、読み取り権限がない、実行権限がない、
	// またはその他のエラー
	perror ("access");
}
```

### dup()

ファイルディスクリプター oldfd のコピーを作成し、   
最も小さい番号の未使用のファイルディスクリプターを 新しいディスクリプターとして使用する。  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/dup.2.html  
```c
```

### dup2()

dup() と同じ処理を実行するが、  
番号が最も小さい未使用のファイルディスクリプターを使用する代わりに、  
newfd で指定されたファイルディスクリプター番号を使用する。  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/dup.2.html  
```c
```

### execve()

pathname で参照されるプログラムを実行する。  
これにより、呼び出し元のプロセスで現在実行されているプログラムは、  
スタック、ヒープ、(初期化および未初期化の)データセグメントが新たに初期化された、  
新しいプログラムに置き換わる。  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/execve.2.html  
```c
```

### exit()

プロセスを正常に終了させ、  
status の最下位バイトを親プロセスへ返す呼び出したところでプログラムを終了させる。  
https://linuxjm.osdn.jp/html/LDP_man-pages/man3/exit.3.html  
```c
```

### fork()

呼び出し元プロセスを複製して新しいプロセスを生成する。  
新しいプロセスは「子」プロセスと呼ばれ、  
呼び出し元プロセスは「親」プロセスと呼ばれる。   
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/fork.2.html  
```c
```

### pipe()

パイプを生成する。  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/pipe.2.html  
```c
```

### unlink()

ファイルシステム上の名前を削除する。  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/unlink.2.html  
```c
```

### wait()

子プロセスのいずれかが終了するまで、  
呼び出し元のスレッドの実行を一時停止する。  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/wait.2.html  
```c
```

###  waitpid()

pid 引数で指定した子プロセスの状態変化が起こるまで、  
呼び出し元のスレッドの実行を一時停止する。  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/wait.2.html
```c
```


