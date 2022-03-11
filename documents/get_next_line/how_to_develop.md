# 実装内容について

実装内容のおおまかな説明です。  
[repository](https://github.com/mznmk/get_next_line_2)    


## main.c

get_next_line のテスト用につくりました。  
Makefile も作成してありますので、make コマンドでコンパイルできます。  
実行ファイル `get_next_line` ができあがります。  

- コンパイル方法  
	```sh
	make re
	```

- 実行方法  
	- 引数なしだと、標準入力から入力を受け取り、その内容を標準出力に出力します。  
		```sh
		./get_next_line
		```

	- 引数にファイルのアドレスを指定すると、指定したファイルからの入力を受け取り、その内容を標準出力に出力します。  
	最大 1024 - 2 ファイル (fd: 3 - 1024) 開けるようにコードを書いたつもりですが、現実的ではありませんね。  
		```sh
		./get_next_line ./testfiles/half_marge_bottom ./testfiles/half_marge_top ...
		```


## get_next_line.c

### get_next_line()

```c
char	*get_next_line(int fd)
```
関数のエントリーポイントです。次の内容を行っています。最上流工程です。  

- 関数 `allocate_memory()` で buffer にメモリの割当を行う。  

- 関数 `append_buffer()` で 読み込んだ内容を buffer に追加する。  

- 関数 `split_buffer()` で 改行・終端文字による buffer の分割を行う。  

- 関数 `validate_buffer()` で 切り出した内容が適正か判断＆修正を行う。  

- 結果 を出力する。  

### append_buffer()

```c
static char	*append_buffer(int fd, char *buff)
```
読み込んだ内容を buffer に追加する処理を行っています。  

- 関数 `read()` で 読み込みを行う。  

- 関数 `join_buffer` で buffer と 読み込んだ内容 との連結を行う。  

### join_buffer()

```c
static char	*join_buffer(char *buff1, char*buff2)
```
buffer と 読み込んだ内容 との連結を行っています。  

### split_buffer()

```c
static char	*split_buffer(char **buff)
```
改行・終端文字による buffer の分割を行っています。  

### validate_buffer()

```c
static char	*validate_buffer(char *buff)
```
切り出した１行分の文字列の内容が適正か判断し、修正まで行っています。  


## get_next_line_utils.c

### allocate_memory()

```c
char	*allocate_memory(int size)
```
動的メモリの割り当てを行っています。  

### deallocate_memory()

```c
char	*deallocate_memory(char **memory)
```
動的メモリの解放を行っています。  

### ft_strlen()

```c
long	ft_strlen(const char *s)
```
文字数のカウントを行っています。  

### ft_substr()

```c
char	*ft_substr(char *s, long start, long len)
```
文字列の動的な部分列を作成しています。  

### find_index()

```c
long	find_index(const char *s, char c)
```
指定した文字が含まれている場所を探します。  

