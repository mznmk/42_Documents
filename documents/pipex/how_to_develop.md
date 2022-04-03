# 実装内容について

実装内容のおおまかな説明です。  
[public](https://github.com/mznmk/pipex) ←未提出  


## main_manda.c

### main()

```c
int	main(int argc, char **argv, char **envp)
```
pipex(mandatory)のエントリーポイントです。  
pipexの実際の処理は `exec_pipex()` から始まります。  
manda_bonus.c とは引数の数、初期設定が違うので、入口部分だけ分けています。  

- 関数 `init_pipex()` で変数の初期化を行っています。  

- 関数 `exec_pipex()` で実際の処理を開始します。  

### init_pipex()

```c
static void	init_pipex(t_vars *v, int argc, char **argv, char **envp)
```
初期設定を行っています。  


## main_bonus.c

### main()

```c
int	main(int argc, char **argv, char **envp)
```
pipex(bonus)のエントリーポイントです。  
pipexの実際の処理は `exec_pipex()` から始まります。  
manda_manda.c とは引数の数、初期設定が違うので、入口部分だけ分けています。  

- 関数 `init_pipex()` で変数の初期化を行っています。  

- 関数 `exec_pipex()` で実際の処理を開始します。  

### init_pipex()

```c
static void	init_pipex(t_vars *v, int argc, char **argv, char **envp)
```
初期設定を行っています。  


## pipex.c

### exec_pipex()

```c
void	exec_pipex(t_vars *v)
```
pipex のメインルーチンです。  

- 関数 `read_from_file()` でファイルから、`read_from_heredoc()` でhere_docから、入力を受け取ります。  

- ループを使い、コマンドを実行（最後の１回以外）しています。  

	- `pipe()`でパイプを作成を行います。  
	- `fork()`でプロセスの複製を行います。 
	- `child_process()`で子プロセスの実行を行います。  
	- 子プロセスの完了を待ち、`parent_process()`で親プロセスの実行を行います。   

- 関数 `write_into_file()`でファイルへ出力を行います。最後の１回のコマンドもここで行います。  

### exit_pipex()

```c
int	exit_pipex(int exit_status)
```
エラーによる異常終了の出口です。  
エラーメッセージの表示も行います。  

### child_process()

```c
static void	child_process(t_vars *v, int *fd, int cmd_num)
```
子プロセスの実行部分です。  
パイプの設定、コマンドの実行を行っています。  

### parent_process()

```c
static void	parent_process(int *fd, pid_t pid)
```
親プロセスの実行部分です。  
子プロセスの完了を待ち、パイプの設定を行っています。  
ここでコマンドの実行を行っていないため、パイプを通じて受け取ったデータは溜まったままになっています。次のコマンド実行時に読み込まれます。  


## openclose.c

### open_read_fd()

```c
int	open_read_fd(t_vars *v)
```
読み込み用としてファイルを開きます。 

### open_write_fd()

```c
int	open_write_fd(t_vars *v)
```
書き出し用としてファイルを開きます。  

### close_fd()

```c
int	close_fd(int fd)
```
ファイルディスクリプターをクローズして解放しています。  
異常が起こった時の終了処理も行っています。  


## stream.c

### read_from_heredoc()

```c
void	read_from_heredoc(t_vars *v, int *fd)
```
here_docを読み込む処理を行っています。  
リダイレクトを行うため、`pipe()` `fork()`〜の一連の流れを行っています。  
関数の呼び出し側から パイプ用`fd[2]`をポインタで渡さないとデータが流れない、ここで結構嵌まるかも。  

### read_from_heredoc_helper()

```c
static void	read_from_heredoc_helper(char *limiter)
```
here_docを読み込む処理の実読み込み部分の処理を行っています。  
get_next_lineで１行ずつ読み込み、標準出力へ淡々と吐き出しています。  

### read_from_file()

```c
void	read_from_file(t_vars *v)
```
読み込み用としてファイルを開き、出力先を標準入力に繋げています。  

### write_into_file()

```c
void	write_into_file(t_vars *v, int cmd_num)
```
書き出し用としてファイルを開き、入力元を標準出力に繋げています。  
ファイルにデータを流すため、最後の１回のコマンドもここで行っています。  


## execute.c

### exec_command()

```c
void	exec_command(char **envp, char *command)
```
環境変数を利用して、コマンドのパスを作成し、コマンドを実行しています。  

### find_pathname()

```c
static char	*find_pathname(char **envp, char *command)
```
環境変数を利用して、コマンドのパスを作成しています。  

### join_pathname()

```c
static char	*join_pathname(char *path, char *name)
```
環境変数PATHをバラしたもの と "/" と コマンド を繋げて、コマンドのフルパスを作成しています。  

### deallocate_memory()

```c
static void	deallocate_memory(char **memory)
```
割り当てたメモリの解放を行っています。  


## get_next_line.c / get_next_line_util.c

here_doc の読み出し用に使っています。  

