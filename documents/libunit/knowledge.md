# 必要な知識

課題挑戦時(2021/5)における理解をまとめた。  


## 利用可能関数に fork() exit() wait() ってあるけど…

process を使えってこと。具体的にはchild processでテストを走らせろってこと。  


### processって？

最初は「実行中のプログラム」的なフワッとした理解で大丈夫。具体例でイメージを掴むとわかりやすい。  
`top`でシステム全体の、`ps`で自身が起動している、processを確認できる。  
非同期であるというのがポイント。Node.js的な挙動を思い浮かべたら腹落ちするんじゃないかと思う。  


### child process でテストを実行する理由

child process を作成した場合/しなかった場合 の結果は次のようになる。  
エラーが起こった時点で終了して、すべてのテストが走らないまま終了してしまうと、テスターとしての役目は果たせない。  
なので、child process を作成してテストを実行する必要がある。  


#### child process を作成しないでテストを実行した場合

テストが異常終了(segmentation fault など)した場合、その時点で終了してしまって、次のテストを走らせることができない。  


#### child process を作成してテストを実行した場合

テストが異常終了しても、テストを実行していたchild processが終了するだけで、parent processは終了しないので、そのまま次のテストを走らせることができる。  


## fork() exit() wait() 関数 を理解する

- fork() で child process を作成する。  
- exit() で child process を終了させる。  
- parent process 内で wait() で child process の終了を待ち受け、その後 wait() 以降を実行する。  

この流れで、止まらないテストを実行している。  


### fork()

child processを作成する。  
fork()した時点で現時点のprocessが複製され、parent process と child process に分岐される。  
返してくる process id で、child process / parent process の判断ができる。  
process id == 0 -> child process  
process id > 0 -> parent process  
```c
pid = fork();	// return value == process id
if (pid == 0)	// child process
	run_child_process(unittest);
else if (pid > 0)	// parent process
	run_parent_process(unittest, &test_success, &test_failure);
else	// fait to fork()
	exit_unittests(unittests, "error: fail to execute unittest");
```


### exit()

processを終了させる。  
第１引数の値を終了ステータスとして返す。  
child process 終了時に実行して、終了させる。  
```c
void run_child_process()
{
	:
	:
	exit(status);
}
```


### wait()

child process の終了を待ち受け、その後 parent process の wait() 以降の処理を実行する。  
ポインタとして渡した第１引数で process id を受け取れる。  
```c
void run_parent_process()
{
	wait(&status);	// receive process id
	:
	:
}
```


## 利用可能マクロに <sys/wait.h> <signal.h> ってあるけど…

process のステータスを受け取るのに使え(<sys/wait.h>)、どのシグナルかを判定するのに使え(<signal.h>)ってこと。  


### <sys/wait.h>マクロ を理解する

マクロに process id を渡すことで process のステータスを受け取れる。  
最低限必要なものは以下のもの。man より引用。  

- WIFEXITED(wstatus)  
子プロセスが正常に終了した場合に真を返す。  
「正常に」とは、 exit(3) か _exit(2) が呼び出された場合、もしくは main() から復帰した場合である。  
- WEXITSTATUS(wstatus)  
子プロセスの終了ステータスを返す。  
終了ステータスは status 引き数の下位 8ビットで構成されており、 exit(3) や _exit(2) の呼び出し時に渡された値、もしくは main() の return 文の 引き数として指定された値である。  
このマクロを使用するのは WIFEXITED が真を返した場合だけにすべきである。  
- WIFSIGNALED(wstatus)  
子プロセスがシグナルにより終了した場合に真を返す。  
- WTERMSIG(wstatus)  
子プロセス終了の原因となったシグナルの番号を返す。  
このマクロを使用するのは WIFSIGNALED が真を返した場合だけにすべきである。  


### <signal.h>マクロ を理解する

シグナル番号を定数として定義してくれている。  
シグナル番号11だとイメージしにくいが、SIGSEGVだとイメージしやすいし、見通しがよくなるので使おう。  
ちなみに`kill -l`でシグナル一覧を確認できる。  
最低限必要なものは以下のもの。  

- SIGSEGV  
segmentation fault が起こった時に発生するシグナル。シグナル番号は11。  

- SIGBUS  
Bus error が起こった時に発生するシグナル。シグナル番号は7。  


### 具体的使用例

```c
if (WIFEXITED(status))	// process は正常終了？
{
	if (WEXITSTATUS(status) == 0)	// 終了ステータスは 0 ？
		print_test_status(unittest->testname, "OK", ESC_CLR_GREEN);
	else
		print_test_status(unittest->testname, "KO", ESC_CLR_RED);
}
if (WIFSIGNALED(status))	// signal で終了してる？
{
	if (WTERMSIG(status) == SIGSEGV)	// そのシグナルは segmentation fault ？
		print_test_status(unittest->testname, "SEGV", ESC_CLR_MAGENTA);
	if (WTERMSIG(status) == SIGBUS)		// そのシグナルは Bus error ?
		print_test_status(unittest->testname, "BUSE", ESC_CLR_MAGENTA);
}
```

