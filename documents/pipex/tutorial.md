# pipexに取り組む前に… チュートリアル

課題挑戦時(2022/3)における理解をまとめました。  
未提出なので間違いあるかもしれません。  


## ちょっとずつ理解していきましょ？

ストリームを使ったプログラミングはとにかくややこしいです。  
書いてるうちに、何と何をつないでるか、だんだんわからなくなってきます。  
まず確実に動く小さなものを作って、それを徐々に大きくしていきましょう。  


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

最初に作るものは、「ファイルをコマンドの入力元に繋げてコマンドを実行する」だけのものです。  
実行コマンドは`wc -w`で、ソースにベタ書きしてあります。  
出力先は特に設定しないので、デフォルトである標準出力のままとなります。  
次のコマンドと同じ挙動となります。  
```sh
< files/infile.txt wc -w
```

[sample_pipe0_cmd1.c](https://github.com/mznmk/pipex/blob/master/srcs/sample_pipe0_cmd1.c?ts=4)  
```c
	// [ set stream ]
	// open file: upstream-end
	fd_read = open(argv[1], O_RDONLY,
							S_IRWXU | S_IRWXG | S_IRWXO);	// (1)
	// set stream: upstream-end
	close(STDIN_FILENO);									// (2)
	dup2(fd_read, STDIN_FILENO);							// (3)
	close(fd_read);											// (4)
	// set stream: downstream-end

	// [ execute command ]
	execlp("wc", "wc", "-w", NULL);							// (5)
```
※ ファイルディスクリプターは長いので、`FD`と略します。  
※ ファイルディスクリプターについてはこちら参照。→[what to do?](./what_to_do.md)  

1. ファイルをオープンし、FDを取得します。（便宜的にFD==3とします）  
2. FD==0（標準入力）をクローズします。  
3. dup2で、FD==0としてFD==3を複製します。FD==0（標準入力）からファイルにアクセスできるようになります。  
4. FD==3（fd_read）をクローズします。これでFD==0（標準入力）からだけファイルにアクセスできるようになります。  
5. `wc -w`コマンドを実行します。  
	コマンドは標準入力からデータを取得しようとしますが、FD==0がファイルにマッピングされているため、ファイルからデータを取得します。	出力先は変更してませんので、標準出力のままとなります。  
	`files/infile.txt`は60単語なので、60と出力されるはずです。    


## サンプルコード２(pipe==1; command==2;)






## サンプルコード３(2 < pipe; command==pipe+1;)








