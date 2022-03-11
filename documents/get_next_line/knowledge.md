# 必要な知識について

課題再挑戦時(2022/3)における理解をまとめました。  
未提出なので間違いあるかもしれません。  


## 利用可能関数

### read()

man:  
https://linuxjm.osdn.jp/html/LDP_man-pages/man2/read.2.html  
```
ファイルディスクリプター (file descriptor) fd から最大 count バイトを  
buf で始まるバッファーへ読み込もうとする。  
```
実装例：  
バッファサイズ ずつ読み込み、読み込んだバッファとつなぎ合わせる処理です。  
```c
while (find_index(temp, '\n') == -1 && status != 0)
{
	status = read(fd, temp, BUFFER_SIZE);
	if (status < -1)
		return (deallocate_memory(&temp));
	temp[status] = '\0';
	buff = join_buffer(buff, temp);
	if (!buff)
		return (deallocate_memory(&temp));
}
```

### malloc()

man:  
https://linuxjm.osdn.jp/html/LDP_man-pages/man3/malloc.3.html  
```
size バイトを割り当て、割り当てられたメモリーに対するポインターを返す。  
メモリーの内容は初期化されない。  
```

利用例：  
引数で受け取ったサイズ＋NULL文字 分の文字列格納用のメモリを確保し、メモリを `'\0'` で埋めたあと、呼び出し元に返します。メモリの確保に失敗したらNULLを返します。  
```c
char	*allocate_memory(int size)
{
	char	*memory;

	size += 1;
	memory = (char *)malloc(sizeof(char) * size);
	if (!memory)
		return (NULL);
	while (size--)
		memory[size] = '\0';
	return (memory);
}
```

### free()

man:  
https://linuxjm.osdn.jp/html/LDP_man-pages/man3/malloc.3.html  
```
ポインター ptr が指すメモリー空間を解放する。  
```

実装例；  
メモリを開放したあと、ダブルフリー防止のため、解放したメモリのアドレスをNULLを埋めています。呼び出し元にはNULLを返しています。  
```c
char	*deallocate_memory(char **memory)
{
	if (*memory)
		free(*memory);
	*memory = NULL;
	return (NULL);
}
```

## 自然変数と静的変数

これまで使ってきた普通の変数（自然変数）は、関数の処理の終了後は値が保持されません。  
一方静的変数は、関数の処理の終了後も、プログラム終了後まで値を保持し続けます。  
宣言時に `static` をつければ、静的変数となります。  

実装例：  
`*line` (１行分の文字列)に設定したポインターは、関数の処理終了後、値を失います。  
`*buff` (バッファ文字列)に設定したポインターは、関数の処理の終了後も値を保持し続けます。次回呼び出し時、引き続き値の利用が可能です。  
```c
char	*get_next_line(int fd)
{
	char		*line;
	static char	*buff;
	:
	:
}

