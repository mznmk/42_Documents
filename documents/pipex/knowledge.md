# 必要な知識

課題挑戦時(2022/3)における理解をまとめました。  


## 利用可能関数

### open()

man:  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/open.2.html  
```
説明：
pathname で指定されたファイルをオープンする。
戻り値：
ファイルディスクリプター
```
```c
fd = open(v->argv[v->infile_index],
		O_RDONLY,
		S_IRWXU | S_IRWXG | S_IRWXO);
```

### close()

man:  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/close.2.html  
```
説明：
ファイルディスクリプターをクローズする。  
戻り値：
ステータス
```
```c
status = close(fd);
```

### read()

man:  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/read.2.html  
```
説明：
ファイルディスクリプター (file descriptor) fd から最大 count バイトを buf で始まるバッファーへ読み込もうとする。
戻り値：
読み込んだバイト数
```
```c
status = read(fd, temp, BUFFER_SIZE);
```

### write()

man:  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/write.2.html  
```
説明：
buf から始まるバッファーから、ファイルディスクリプター fd が参照するファイルへ、最大 count バイトを書き込む。
戻り値：
書き込んだバイト数
```
```c
未使用
```

### malloc()

man:  
https://linuxjm.osdn.jp/html/LDP_man-pages/man3/malloc.3.html  
```
説明：
size バイトを割り当て、割り当てられたメモリーに対するポインターを返す。メモリーの内容は初期化されない。
戻り値：
メモリーへのポインター  
```

```c
pathname = (char *)malloc(sizeof(char) * (len1 + len2 + 2));
```

### free()

man:  
https://linuxjm.osdn.jp/html/LDP_man-pages/man3/malloc.3.html  
```
説明：
ポインター ptr が指すメモリー空間を解放する。
戻り値：
なし
```
```c
while (memory[++i])
{
	free(memory[i]);
	memory[i] = NULL;
}
free(memory);
memory = NULL;
```

### perror()

man:  
https://linuxjm.osdn.jp/html/LDP_man-pages/man3/perror.3.html  
```
説明：
システムコールやライブラリ関数の呼び出しにおいて、最後に発生した エラーに関する説明メッセージを生成し、標準エラー出力に出力する。
戻り値：
なし
```
```c
int	exit_pipex(int exit_status)
{
	perror(FNT_BOLD CLR_PINK "ERROR" ESC_RESET);
	exit(exit_status);
}
```

### strerror()

man:  
https://linuxjm.osdn.jp/html/LDP_man-pages/man3/strerror.3.html  
```
説明：
引数 errnum で渡されたエラーコードについての説明が入った文字列へのポインターを返す。
戻り値：
エラー内容を説明する文字列
```
```c
未使用
```

### access()

man:  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/access.2.html  
```
説明：
呼び出し元プロセスがファイル pathname にアクセスできるかどうかをチェックする。
戻り値：
ステータス
```
```c
if (access(pathname, F_OK) == 0)
	return (pathname);
```

### dup()

man:  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/dup.2.html  
```
ファイルディスクリプター oldfd のコピーを作成し、   
最も小さい番号の未使用のファイルディスクリプターを 新しいディスクリプターとして使用する。  
```
```c
未使用
```

### dup2()

man:  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/dup.2.html  
```
説明：
dup() と同じ処理を実行するが、番号が最も小さい未使用のファイルディスクリプターを使用する代わりに、newfd で指定されたファイルディスクリプター番号を使用する。
戻り値：
ファイルディスクリプター
```
```c
dup2(fd[1], STDOUT_FILENO);
```

### execve()

man:  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/execve.2.html  
```
説明：
pathname で参照されるプログラムを実行する。これにより、呼び出し元のプロセスで現在実行されているプログラムは、スタック、ヒープ、(初期化および未初期化の)データセグメントが新たに初期化された、新しいプログラムに置き換わる。
戻り値：
ステータス
```
なお、exec()ファミリーには、`execve()` の他に  
`execl()` `execlp()` `execle()` `execv()` `execvp()` `execvpe()` などがあります。  
たくさんあって混乱しそうですが、以下のように考えると理解しやすいです。  
`l` → 引数をリストで1つずつ渡す  
`v` → 引数を配列で渡す  
`e` → 環境変数を配列で渡す  
```c
execve(pathname, argv, envp);
```

### exit()

man:  
https://linuxjm.osdn.jp/html/LDP_man-pages/man3/exit.3.html  
```
説明：
プロセスを正常に終了させ、status の最下位バイトを親プロセスへ返す呼び出したところでプログラムを終了させる。
戻り値：
なし
```
```c
int	exit_pipex(int exit_status)
{
	perror(FNT_BOLD CLR_PINK "ERROR" ESC_RESET);
	exit(exit_status);
}
```

### fork()

man:  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/fork.2.html  
```
説明：
呼び出し元プロセスを複製して新しいプロセスを生成する。新しいプロセスは「子」プロセスと呼ばれ、呼び出し元プロセスは「親」プロセスと呼ばれる。
戻り値：
プロセスID
```
```c
while (cmd_num < v->cmd_count - 1)
{
	if (pipe(fd) == -1)
		exit_pipex(EXIT_FAILURE);
	pid = fork();
	if (0 == pid)
		child_process(v, fd, cmd_num);
	else if (0 < pid)
		parent_process(fd, pid);
	else
		exit_pipex(EXIT_FAILURE);
	cmd_num++;
}
```

### pipe()

man:  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/pipe.2.html  
```
説明：
パイプを生成する。
戻り値：
ステータス
```
```c
while (cmd_num < v->cmd_count - 1)
{
	if (pipe(fd) == -1)
		exit_pipex(EXIT_FAILURE);
	pid = fork();
	if (0 == pid)
		child_process(v, fd, cmd_num);
	else if (0 < pid)
		parent_process(fd, pid);
	else
		exit_pipex(EXIT_FAILURE);
	cmd_num++;
}
```

### unlink()

man:  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/unlink.2.html  
```
説明：
ファイルシステム上の名前を削除する。
戻り値：
ステータス
```
```c
未使用
```

### wait()

man:  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/wait.2.html  
```
説明：
子プロセスのいずれかが終了するまで、呼び出し元のスレッドの実行を一時停止する。  
戻り値：
ステータス
```
```c
未使用
```

###  waitpid()

man:  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/wait.2.html
```
説明：
pid 引数で指定した子プロセスの状態変化が起こるまで、呼び出し元のスレッドの実行を一時停止する。
戻り値：
ステータス
```
```c
waitpid(pid, NULL, 0);
```

