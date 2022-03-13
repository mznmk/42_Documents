# framework

テスト用のマイクロフレームワーク。  

## struct

```
typedef struct s_unittest
{
	char			*testname;		// test name
	int			(*testfunc)(void);	// function pointer
	struct s_unittest	*next;
}				t_unittest;
```


## source

### [directry] utility

ft_putchar_fd / ft_strlen などの utility 関数を入れている。  
名前が競合しないよう、ft ではなく ut にしている。  


### list_utilities.c

unittest を格納する list(t_unittest) に関する関数。  

- create_unittest
- add_unittest
- clear_unittests


### print_header.c

unittest test それぞれのヘッダ部分を表示する関数。  

- print_framework_header
- print_unittests_header


### print_status.c

テストのステータスを表示する関数。  

- print_unittest_status
- print_unittests_number
- print_unittests_result
- print_unittests_score
- print_error_message


### run_child_process.c

child process 実行関数。  

- run_child_process


### run_child_process_bonus.c

child process 実行関数 (Bonus用)。  

- exit_timeout
- run_child_process


### run_parent_process.c

parent process 実行関数。  

- run_parent_process_normal_term
- run_parent_process_signal_term
- run_parent_process

### run_unittests.c

メイン部分。  
テスト実行に関する関数。  

- exit_unittests
- run_all_unittests
- run_unittests