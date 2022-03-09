# 必要な知識

課題挑戦時(2021/6)における理解をまとめた。  


## 利用可能関数


### signal()

signal を受信するために使う。  
第1引数には 受け取りたいシグナル、第2引数には シグナルを受け取った時の処理への関数ポインタ、をセットすればよい。  
`SIGUSR1` を受信した時の動作の具体的設定例はこちら。  

```c
static void	handle_signal(int signal)
{
	g_receive_signal = signal;
}

void	set_signal(void)
{
	signal(SIGUSR1, &handle_signal);
}
```

### sigaction() sigemptyset() sigaddset()

signal を受信するために使う。  
`signal`関数は移植性が低いため、推奨されていないらしく、`sigaction()`関数を使うほうが良いとのこと。  
`struct sigaction` に行いたい設定をセットし、それへのポインタを `sigaction` の第2引数にセットすると、そのように動作する。  
`sa_handler` もしくは `sa_sigaction` に シグナルを受け取った時の処理への関数ポインタ を渡せば、そのように動作する。2つの違いは引数。  
`sa_mask` をセットすると、受け取れるシグナルにマスクをかけることができる。`sigemptyset()` はマスクをすべてオフに、 `sigfullset()` はマスクを全てオンに `sigaddset()` は指定されたマスクだけオンにする。  
`sigaction` の第1引数には 受け取りたいシグナル をセットすればよい。  
`SIGUSR1` を受信した時の動作の具体的設定例はこちら。  

```c
static void	handle_signal(int signal)
{
	g_receive_signal = signal;
}

void	set_signal(void)
{
	struct sigaction	sa;

	sa.sa_handler = &handle_signal;
	sigemptyset(&sa.sa_mask);
	sigaddset(&sa_sigusr1.sa_mask, SIGUSR1);}
	sigaction(SIGUSR1, &sa, NULL);
}
```

### kill()

signal を送信するのに使う。  
`kill` には Process を終了させるイメージがあるが、それは shell の `kill` コマンドの影響だと考える。shell の `kill` コマンドにはデフォルトで `SIGTERM` が設定されているため、何もシグナルを指定しないと Process を終了させる処理を行ってしまうからだ。  
`kill()` 関数は、指定した ProcessID に向けて、指定した signal を送信する。  
第1引数には 送りたい相手のProcessID、第2引数には 送りたいシグナル、をセットすればよい。  
signal を送信する具体例はこちら。  

```c
kill(server_process_id, send_signal_number);
```

### getpid()

自身の ProcessID を取得する。  
具体例はこちら。  

```c
pid_client = getpid();
```

### pause()

signal が送られてくるまで、処理を停止する。  
`pause()` で signal を待ち受ける時の具体例はこちら。  

```c
while (1)
{
	if (g_receive_signal == SIGUSR1 || g_receive_signal == SIGUSR2)
		receive_bit(&vars, g_receive_signal);
	if (g_receive_signal == SIGQUIT || g_receive_signal == SIGINT)
		exit_server(&vars, SUCCESS_MSG_TERM_SERVER, true);
	pause();
}
```

### sleep() / usleep()

指定した時間、処理を停止する。  
2つの違いは、停止時間の単位。`sleep()` はミリ(1/1000)秒単位、 `usleep()` はマイクロ(1/1000000)秒単位。  
わざわざ具体例を出すまでもないけれども…。  

```
sleep(1);		// 1/1000秒停止
usleep(1000);	// 1/1000秒停止
```

