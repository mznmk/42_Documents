# pipexに取り組む前に… チュートリアル

課題挑戦時(2022/3)における理解をまとめました。  
未提出なので間違いあるかもしれません。  


## pipex repository

[public](https://github.com/mznmk/pipex) ←未提出  


## ちょっとずつ着実に

ストリームを使ったプログラミングはとにかくややこしくて、混乱しがちです。  
ストリームの流れをデバッグするのは大変で、パイプの状態がよくわからないまま、BadFileDiscripterなんたらかんたら…などとエラーが出ようものなら大変です。  
まず確実に動く小さなものを作って、それを徐々に大きくしていくのが良いです。  


## パイプの感覚をつかむために

まず最初にですが、名前付きパイプを使って、２つのターミナル間でデータのやり取りをしながら、パイプの感覚を掴みます。  

1. 名前付きパイプを作成します。  
	```sh
	mkfifo ppp
	```
	ls -l で確認します。一番左が"p"になっている、名前付きパイプができています。  
	このパイプでデータのやり取りができます。  
	```
	prw-rw-r-- 1 mika mika     0 Apr  3 18:54 ppp
	```

2. ターミナルを２つ開き、左右に並べます。  

3. パイプへの送信待ち を試します。  
	右のターミナルで`cat ppp`コマンドを実行して、パイプの出口側に`cat ppp`のプロセスの入力元を繋ぎます。まだパイプの入口側には何も繋がれてすらいないので、ひたすらデータを待ち続ける状態のまま、処理を終わることができずにいます。  
	それを確認した上で、`ls -l > ppp`コマンドを実行して、パイプの入口側に出力先を繋げ、データを流してやります。実行した瞬間、右のターミナルにデータが表示され、`cat ppp`の処理が完了します。  

4. パイプからの受信待ち を試します。  
	左のターミナルで`ls -l > ppp`コマンドを実行して、パイプの入口側に出力先を繋げ、データを流します。まだパイプの出口側には何も繋がれてすらいないので、行き場のないデータが留まり続ける状態のまま、処理を終わることができずにいます。  
	それを確認した上で、`cat ppp`コマンドを実行して、パイプの出口側に入力先を繋げ、データを受け取ります。実行した瞬間、右のターミナルにデータが表示され、`ls -l > ppp` `cat ppp`の処理が同時に完了します。  

5. こういう感じでいろいろ試してみて、感じを掴みます。  

6. 感じがつかめたな、と思ったら、`rm ppp` で名前付きパイプを削除します。  


## 入力元ファイル(infile.txt)

[infile.txt](https://github.com/mznmk/pipex/blob/master/files/infile.txt)  
```
aa xx xx xx bb
aa xx yy yy bb
aa xx yy zz bb
aa yy zz yy bb

aa xx xx xx cc
aa xx yy yy cc
aa xx yy zz cc
aa yy zz yy cc

bb xx xx xx cc
bb xx yy yy cc
bb xx yy zz cc
bb yy zz yy cc
```

## サンプルコード１(pipe==0; command==1;)

最初に作るものは、本当にシンプルなものです。  
プロセスの入力元にファイルを繋げ、コマンドを実行するだけのものです。  
プロセスの出力先は変更していませんので、標準出力のままとなります。  

実行コマンドはこちらです。  
```sh
./p0c1 files/infile.txt
```
次のコマンドと同じ挙動となります。  
```sh
< files/infile.txt wc -w
```

[sample_pipe0_cmd1.c](https://github.com/mznmk/pipex/blob/master/srcs/sample_pipe0_cmd1.c?ts=4)  
```c
	// [ set stream ]
	// open file: upstream-end
	fd_read = open(argv[1],	O_RDONLY,
				S_IRWXU | S_IRWXG | S_IRWXO);	// (1)
	// set stream: upstream-end
	close(STDIN_FILENO);		// (2)
	dup2(fd_read, STDIN_FILENO);		// (3)
	close(fd_read);			// (4)
	// set stream: downstream-end

	// [ execute command ]
	execlp("wc", "wc", "-w", NULL);		// (5)
```

※ ファイルディスクリプターは長いので、"FD"と略します。  

1. ファイルをオープンし、FDを取得します。（便宜的にFD==3とします）  
2. FD==0（標準入力）をクローズします。  
3. dup2で、FD==0としてFD==3を複製します。  
	FD==0（標準入力）からファイルにアクセスできるようになります。  
4. FD==3（`fd_read`）をクローズします。  
	これでFD==0（標準入力）からのみ、ファイルにアクセスできるようになります。  
5. `wc -w`コマンドを実行します。  
	コマンドは標準入力からデータを取得しようとしますが、FD==0がファイルにマッピングされているため、ファイルからデータを取得します。出力先は変更していませんので、標準出力のままとなります。  


## サンプルコード２(pipe==1; command==2;)

次にパイプを使った処理を行ってみます。  
２つのプロセスを１本のパイプで繋げて、プロセス同士のデータのやり取りをします。コマンドの実行回数は２回です。  
最初のプロセスの入力元にファイルを繋げ、出力先にはパイプの入口を繋ぎます。コマンドを実行すると、パイプを通じて実行結果を２番目のプロセスに流します。  
２番目のプロセスの入力先にパイプの出口を繋ぎ、出力先にはファイルを繋ぎます。コマンドを実行すると、パイプを通じて最初のプロセスでの実行結果を受け取り、コマンドの実行結果をファイルに流します。  

実行コマンドはこちらです。  
```sh
./p1c2 files/infile.txt files/outfile.txt
```
次のコマンドと同じ挙動となります。  
```sh
< files/infile.txt grep aa | wc -w > files/outfile.txt
```

[sample_pipe1_cmd2](https://github.com/mznmk/pipex/blob/master/srcs/sample_pipe1_cmd2.c?ts=4)  
```c
int
main(int argc, char **argv, char **envp)
{
	// [ execute command ]
	// create pipe
	pipe(fd);		// (1)
	// fork process
	pid = fork();		// (2)
	if (0 == pid)
	{
		child_process(argv, fd);	// (3)
	}
	else if (0 < pid)
	{
		waitpid(pid, NULL, 0);		// (4)
		parent_process(argv, fd);	// (5)
	}
}
```

<プロセス１>  
1. システムコール`pipe()`でパイプを作成します。プロセス同士でデータをやり取りするためです。  
2. システムコール`fork()`でプロセスを複製します。  
	親プロセス・子プロセスの２つになりますが、パイプは１本のままです。入口と出口がそれぞれ２つあって、親プロセス・子プロセス両方に繋がっている、不思議なパイプの状態になります。  
	その状態で片方のプロセス側(A)のパイプの出口を閉じ、もう片方のプロセス側(B)のパイプの入口を閉じると、プロセス(A)のパイプの入口からプロセス(B)のパイプの出口へとデータが流れます。  
3. 子プロセスを実行します。  
4. 子プロセスの終了を待ちます。  
5. 親プロセスを実行します。  

[sample_pipe1_cmd2](https://github.com/mznmk/pipex/blob/master/srcs/sample_pipe1_cmd2.c?ts=4)  

```c
static void
child_process(char **argv, int *fd)
{
	// [ set "child-side" stream ]
	// open file: upstrem-end
	fd_read = open(argv[1], O_RDONLY,
				S_IRWXU | S_IRWXG | S_IRWXO);	// (1)
	// set stream: upstream-end
	close(fd[0]);		// (5)
	close(STDIN_FILENO);		// (2)
	dup2(fd_read, STDIN_FILENO);	// (3)
	close(fd_read);		// (4)
	// set stream: downstream-end
	close(STDOUT_FILENO);		// (6)
	dup2(fd[1], STDOUT_FILENO);		// (7)

	// [ execute command ]
	execlp("grep", "grep", "aa", NULL);	// (8)
}
```

※ ファイルディスクリプターは長いので、"FD"と略します。  

<プロセス２（子プロセス）>  
1. ファイルをオープンし、FDを取得します。（便宜的にFD==3とします）  
2. FD==0（標準入力）をクローズします。  
3. dup2で、FD==0としてFD==3を複製します。FD==0（標準入力）からファイルにアクセスできるようになります。  
4. FD==3（`fd_read`）をクローズします。これでFD==0（標準入力）からのみ、ファイルにアクセスできるようになります。  
5. パイプの出口を閉じます。（入力にパイプを使わないので）  
6. FD==1（標準出力）をクローズします。  
7. dup2で、FD==1（標準出力）として`fd[1]`を複製します。FD==1（標準出力）からパイプの入口にアクセスできるようになります。  
8. `grep aa`コマンドを実行します。  
	- コマンドは標準入力からデータを取得しようとしますが、FD==0（標準入力）がファイルにマッピングされているため、ファイルからデータを取得します。  
	- コマンドは標準出力へデータを出力しようとしますが、FD==1（標準出力）がパイプの入口にマッピングされているため、パイプの入口に対してデータの出力を行います。  

[sample_pipe1_cmd2](https://github.com/mznmk/pipex/blob/master/srcs/sample_pipe1_cmd2.c?ts=4)  
```c
static void
parent_process(char **argv, int *fd)
{
	// [ set "parent-side" stream ]
	// set stream: upstream-end
	close(STDIN_FILENO);		// (1)
	dup2(fd[0], STDIN_FILENO);		// (2)
	// open file: downstream-end
	fd_write = open(argv[2], O_WRONLY | O_CREAT | O_TRUNC,
				S_IRWXU | S_IRWXG | S_IRWXO);		// (3)
	// set stream: downstream-end
	close(fd[1]);		// (7)
	close(STDOUT_FILENO);		// (4)
	dup2(fd_write, STDOUT_FILENO);	// (5)
	close(fd_write);		// (6)

	// [ execute command ]
	execlp("wc", "wc", "-w", NULL);		// (8)
}
```

※ ファイルディスクリプターは長いので、"FD"と略します。  

<プロセス１（親プロセス）>  
1. FD==0（標準入力）をクローズします。  
2. dup2で、FD==0（標準入力）として`fd[0]`を複製します。FD==0（標準入力）からパイプの出口にアクセスできるようになります。  
3. ファイルをオープンし、FDを取得します。（便宜的にFD==3とします）  
4. FD==1（標準出力）をクローズします。  
5. dup2で、FD==1（標準出力）としてFD==3を複製します。FD==1（標準出力）からファイルにアクセスできるようになります。  
6. FD==3（`fd_write`）をクローズします。これでFD==1（標準出力）からのみ、ファイルにアクセスできるようになります。  
7. パイプの入口を閉じます。（出力にパイプを使わないので）  
8. `wc -w`コマンドを実行します。  
	- コマンドは標準入力からデータを取得しようとしますが、FD==0（標準入力）がパイプの出口にマッピングされているため、パイプの出口からデータの取得を行います。  
	- コマンドは標準出力へデータを出力しようとしますが、FD==1（標準出力）がファイルにマッピングされているため、ファイルへデータを出力します。  


## サンプルコード３(pipe==5; command==6;)

最後に複数のパイプを使った処理を行ってみます。  
６つのプロセスを５本のパイプで繋げて、プロセス同士のデータのやり取りをします。コマンドの実行回数は６回です。  
まず、ベースとなるプロセスの入力元にファイルを繋げます。最初の５回のコマンドは、サンプルコード２のように、子プロセス・親プロセスを使ってデータのやり取りをします。最後の１回のコマンドは、プロセスの出力先にファイルを繋げてから、実行します。これで実行結果をファイルに流せます。  

サンプルコード２では、プロセスが２つだけだったので、最初のプロセスの入力元にファイルを繋げ、２番目のプロセスの出力先にはファイルを繋ぎました。  
それだとループを使ったうまい処理ができないので、ファイルの入出力は子プロセス・親プロセスの部分から独立させました。  

ちなみにですが、`main()`の`cmd_cnt = 6`を1〜6の範囲内で変更すると、コマンドの実行回数を変更できます。  

実行コマンドはこちらです。  
```sh
./p2c3 files/infile.txt files/outfile.txt
```
次のコマンドと同じ挙動となります。  
```sh
< files/infile.txt grep aa | grep xx | grep yy | grep zz | grep bb | wc -w > files/outfile.txt
```

[sample_pipe2_cmd3](https://github.com/mznmk/pipex/blob/master/srcs/sample_pipe2_cmd3.c?ts=4)  
```c
int
main(int argc, char **argv, char **envp)
{
	// [ declear const variable ]
	// preset command count (1/2/3/4/5/6)
	cmd_cnt = 6;

	// [ execute command ]
	// set stream (read stream)
	set_read_stream(argv);		// (1)
	// execute command (0 <= cmd_num < cmd_cnt-1)
	cmd_num = 0;
	for (; cmd_num < cmd_cnt - 1; cmd_num++)
	{
		// create pipe
		pipe(fd);		// (2)

		// fork process
		pid = fork();		// (3)
		if (0 == pid)
		{
			// set stream (upstream-end)
			// & execute command (0 <= cmd_num < cmd_cnt-1)
			child_process(fd, cmd_cnt, cmd_num);	// (4)
		}
		else if (0 < pid)
		{
			// set stream (downstream-end)
			waitpid(pid, NULL, 0);	// (5)
			parent_process(fd);		// (6)
		}
	}
	// set stream (write stream)
	// & execute command (cmd_num == cmd_cnt-1)
	set_write_stream(argv, cmd_cnt, cmd_num);	// (8)
}
```

<プロセス１>  
1. プロセスの入力元にファイルを繋げる処理です。  
2. システムコール`pipe()`でパイプを作成します。プロセス同士でデータをやり取りするためです。  
3. システムコール`fork()`でプロセスを複製します。  
	親プロセス・子プロセスの２つになりますが、パイプは１本のままです。入口と出口がそれぞれ２つあって、親プロセス・子プロセス両方に繋がっている、不思議なパイプの状態になります。  
	その状態で片方のプロセス側(A)のパイプの出口を閉じ、もう片方のプロセス側(B)のパイプの入口を閉じると、プロセス(A)のパイプの入口からプロセス(B)のパイプの出口へとデータが流れます。  
4. 子プロセスを実行します。  
5. 子プロセスの終了を待ちます。  
6. 親プロセスを実行します。
7. ループを使って 2.〜6. の処理を繰り返し行い、複数回のコマンドの実行を行います。  
8. プロセスの出力先にファイルを繋げる処理です。  

[sample_pipe2_cmd3](https://github.com/mznmk/pipex/blob/master/srcs/sample_pipe2_cmd3.c?ts=4)  
```c
static void
set_read_stream(char **argv)
{
	// [ set stream ]
	// open file
	fd_read = open(argv[1], O_RDONLY,
				S_IRWXU | S_IRWXG | S_IRWXO);	// (1)
	// set stream: upstream-end
	close(STDIN_FILENO);		// (2)
	dup2(fd_read, STDIN_FILENO);	// (3)
	close(fd_read);		// (4)
}
```

※ ファイルディスクリプターは長いので、"FD"と略します。  

<プロセス１>  
1. ファイルをオープンし、FDを取得します。（便宜的にFD==3とします）  
2. FD==0（標準入力）をクローズします。  
3. dup2で、FD==0としてFD==3を複製します。  
	FD==0（標準入力）からファイルにアクセスできるようになります。  
4. FD==3（`fd_read`）をクローズします。  
	これでFD==0（標準入力）からのみ、ファイルにアクセスできるようになります。  

[sample_pipe2_cmd3](https://github.com/mznmk/pipex/blob/master/srcs/sample_pipe2_cmd3.c?ts=4)  
```c

static void
child_process(int *fd, int cmd_cnt, int cmd_num)
{
	// [ set "child-side" stream ]
	// set stream: upstream-end
	close(fd[0]);		// (1)
	// set stream: downstream-end
	close(STDOUT_FILENO);		// (2)
	dup2(fd[1], STDOUT_FILENO);		// (3)

	// [ execute command ]
	exec_comand(cmd_cnt, cmd_num);	// (4)
}
```

※ ファイルディスクリプターは長いので、"FD"と略します。  

<プロセス２（子プロセス）>  
1. パイプの出口を閉じます。（入力にパイプを使わないので）  
2. FD==1（標準出力）をクローズします。  
3. dup2で、FD==1（標準出力）として`fd[1]`を複製します。FD==1（標準出力）からパイプの入口にアクセスできるようになります。  
4. `grep`コマンドを実行します。  
	- コマンドは標準入力からデータを取得しようとします。  
		初回コマンド実行時は、FD==0（標準入力）がファイルにマッピングされているため、ファイルからデータを取得します。  
		２回目〜５回目のコマンド実行時ですが、これがちょっとややこしいです。前回のコマンド実行時に、実行結果をパイプを通じて親プロセスに送ったのですが、親プロセスでコマンドが実行されておらず、行き場のないデータが標準入力に留まっている状態が続いています。コマンドを実行した瞬間にそれが流れ込んできます。
	- コマンドは標準出力へデータを出力しようとしますが、FD==1（標準出力）がパイプの入口にマッピングされているため、パイプの入口に対してデータの出力を行います。  

[sample_pipe2_cmd3](https://github.com/mznmk/pipex/blob/master/srcs/sample_pipe2_cmd3.c?ts=4)  
```c
static void
parent_process(int *fd)
{
	// [ set "parent-side" stream ]
	// set stream: upstream-end
	close(STDIN_FILENO);
	dup2(fd[0], STDIN_FILENO);
	// set stream: downstream-end
	close(fd[1]);
}
```

※ ファイルディスクリプターは長いので、"FD"と略します。  

<プロセス１（親プロセス）>  
1. FD==0（標準入力）をクローズします。  
2. dup2で、FD==0（標準入力）として`fd[0]`を複製します。FD==0（標準入力）からパイプの出口にアクセスできるようになります。  
3. パイプの入口を閉じます。（出力にパイプを使わないので）  

※ ここではコマンドは実行しません。  
	FD==0（標準入力）がパイプの入口にマッピングされているため、子プロセスからコマンドの実行結果が送られてきているのですが、それに対して何も行わないので、データが留まったままになっています。次にコマンドを実行した際、それが読み込まれることとなります。  

[sample_pipe2_cmd3](https://github.com/mznmk/pipex/blob/master/srcs/sample_pipe2_cmd3.c?ts=4)  
```c
static void
set_write_stream(char **argv, int cmd_cnt, int cmd_num)
{
	// [ set stream ]
	// open file
	fd_write = open(argv[2], O_WRONLY | O_CREAT | O_TRUNC,
					S_IRWXU | S_IRWXG | S_IRWXO);	// (1)
	// set stream: downstream-end
	close(STDOUT_FILENO);		// (2)
	dup2(fd_write, STDOUT_FILENO);	// (3)
	close(fd_write);		// (4)

	// [ execute command ]
	exec_comand(cmd_cnt, cmd_num);	// (5)
}
```

※ ファイルディスクリプターは長いので、"FD"と略します。  

<プロセス１>  
1. ファイルをオープンし、FDを取得します。（便宜的にFD==3とします）  
2. FD==1（標準出力）をクローズします。  
3. dup2で、FD==1（標準出力）としてFD==3を複製します。FD==1（標準出力）からファイルにアクセスできるようになります。  
4. FD==3（`fd_write`）をクローズします。これでFD==1（標準出力）からのみファイルにアクセスできるようになります。  
5. `wc -w`コマンドを実行します。  
	- コマンドは標準入力からデータを取得しようとします。  
		前回のコマンド実行時に、実行結果をパイプを通じて親プロセスに送ったのですが、親プロセスでコマンドが実行されておらず、行き場のないデータが標準入力に留まっている状態が続いています。コマンドを実行した瞬間にそれが流れ込んできます。  
	- コマンドは標準出力へデータを出力しようとしますが、FD==1（標準出力）がファイルにマッピングされているため、ファイルへデータを出力します。  

