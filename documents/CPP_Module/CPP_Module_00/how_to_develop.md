## どのように実装するか

未提出なので、間違いあるかもしれません。  


## Exercise 00: Megaphone

実行例から、実行ファイルの引数として受け取った文字列を大文字にすればよいとわかります。  
`std::transform()`は使えません（`char **argv`で受け取ることから）ので、`std::toupper()`で１文字ずつセコセコ処理すればよいです。  


## Exercise 01: My Awesome PhoneBook

８個しか登録できない、ファッキンな電話帳を実装します。  
ADD/SEARCH/EXIT コマンドを実装する必要があります。  

１個のデータには、次の5項目が必要です。  
first name, last name, nickname, phone number, darkest secret  
```
※ Piscine_CPP 版は、次の11項目を実装する必要がありました。  
firstName, lastName, nickname, login, postalAddress, emailAddress, phoneNumber, birthdayDate, favoriteMeal, underwearColor, darkestSecret  
```
１個のデータを格納するクラス(contact.cpp/hpp)を配列で８個作成します。  
（動的確保してはいけないので、new は使わない。）  
そしてそれをコントロールするクラス(contacts.cpp/hpp)を作成して、制御すればいけます。  

### 実装の流れ

無限ループでREPLを回します。  
std::getlineを使って読み込みを行い、そこからコマンド毎に処理を振り分けます。  

- ADD  
	データ追加モードに入ります。  
	（`contacts.addContact();` を実行します。）  
	書き込み可能な場所を探し、データを追加します。  
	空きがなければ一番古いところを上書きするので、通算番号で管理する必要があります。  
	ひと項目ずつ入力待ちしながら、データを入力を受け付けます。  

- SEARCH  
	データ検索モードに入ります。  
	（`contacts.searchContact();`を実行します。）  
	検索したいデータを選ぶため、電話帳データ（省略版）を表示します。  
	そのうえで、表示したい詳細データのインデックスの入力待ちをします。
	入力したインデックスに問題がなければ、詳細なデータを表示します。    

- EXIT  
	REPLを抜け、Phonebookを終了します。  

### 注意点

c++=98 では `to_string()` が使えませんので、IDを表示するときには工夫が必要です。  
contact[8]は継承したりしているわけではないので、ちゃんとデストラクターが走るはずなので、デストラクターに`virtual`をつける必要はないはずです。  

