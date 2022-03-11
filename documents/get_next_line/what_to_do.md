# 何をやったらいいの？

課題再挑戦時(2022/3)における理解をまとめました。  
未提出なので間違いあるかもしれません。  


## 何をやったらいいの？

標準入力やファイルから、１行ずつデータを読み込む関数をつくればいいです。  
つくるのは関数です。他の関数から呼び出して使います。

ただ単に１行ずつデータを読み込むだけならいろいろ方法があります。ページ全体を読み込んでから、１行分のデータを抜き出すという方法もありますが、それは禁止されています。一定のバッファサイズずつ読み込まなければなりません。  


## 具体的に何をやったらいいの…？

具体的には、read関数で指定されたバッファサイズ分の読み込みを繰り返し、読み込んだバッファを貯め込みます。そして改行またはファイル終端が現れたら、必要に応じて切り出して出力すればいいです。  


## どうやって読み込むの？

以下の文章で考えていきましょう。  
```
I have a dream that one day on the red hills of Georgia, the sons of former slaves and the sons of former slave owners will be able to sit down together at the table of brotherhood.

I have a dream that one day even the state of Mississippi, a state sweltering with the heat of injustice, sweltering with the heat of oppression, will be transformed into an oasis of freedom and justice.

I have a dream that my four little children will one day live in a nation where they will not be judged by the color of their skin but by the content of their character.

I have a dream today!
```
`BUFFER_SIZE=32` とします。  
`read`関数で指定されたサイズ(32Byte)での読み込みを繰り返し、バッファに貯め込んでいきます。  

- １回目の読み込みは `I have a dream that one day on t` です。  
読み込み後のバッファの内容は次のようになります。  
	```
	I have a dream that one day on t
	```
- ２回目の読み込みは `he red hills of Georgia, the son` です。  
読み込み後のバッファの内容は次のようになります。  
	```
	I have a dream that one day on the red hills of Georgia, the son
	```
- ３回目の読み込みは `s of former slaves and the sons ` です。  
読み込み後のバッファの内容は次のようになります。  
	```
	I have a dream that one day on the red hills of Georgia, the sons of former slaves and the sons 
	```
- ４回目の読み込みは `of former slave owners will be a` です。  
読み込み後のバッファの内容は次のようになります。  
	```
	I have a dream that one day on the red hills of Georgia, the sons of former slaves and the sons of former slave owners will be a
	```
- ５回目の読み込みは `ble to sit down together at the ` です。  
読み込み後のバッファの内容は次のようになります。  
	```
	I have a dream that one day on the red hills of Georgia, the sons of former slaves and the sons of former slave owners will be able to sit down together at the 
	```
- ６回目の読み込みは `table of brotherhood.<改行><改行>I have a ` です。  
読み込み後のバッファの内容は次のようになります。  
	```
	I have a dream that one day on the red hills of Georgia, the sons of former slaves and the sons of former slave owners will be able to sit down together at the table of brotherhood.<改行><改行>I have a 
	```
	改行が含まれているので、切り出して出力します。出力の内容は次のようになります。  
	```
	I have a dream that one day on the red hills of Georgia, the sons of former slaves and the sons of former slave owners will be able to sit down together at the table of brotherhood.<改行>
	```
	出力後のバッファの内容は次のようになります。  
	```
	<改行>I have a 
	```
- ７回目の読み込みは `dream that one day even the stat` です。  
読み込み後のバッファの内容は次のようになります。  
	```
	<改行>I have a dream that one day even the stat
	```
	改行が含まれているので、切り出して出力します。出力の内容は次のようになります。  
	```
	<改行>
	```
	出力後のバッファの内容は次のようになります。  
	```
	I have a dream that one day even the stat
	```
- 以下繰り返し…。

この流れで最終的にファイルの最後に到達して、読み込み完了となります。ステップに分けていくと、意外と簡単ですね。   

ちなみにですが、ASCII以外の文字の読み込みもできます。たとえば、次の文章も読み込めます。  
```
吾々は、かならず卑屈なる言葉と怯懦なる行爲によつて、祖︀先を辱しめ、人間を冐瀆󠄂してはならぬ。そうして人の世の冷たさが、何んなに冷たいか、人間を勦はる事が何んであるかをよく知つてゐる吾々は、心から人生の熱と光を願求禮讃するものである。

水平󠄁社は、かくして生れた。

人の世に熱あれ、人間に光あれ。
```
なぜかというと、コンピューターはあくまで01のビット列としてしか認識していないからです。指示通りにバイトの受け渡しが成功すると、受け渡し前と受け渡し後は同じバイトの状態が保たれます。文字コードはデータの受け渡しには関係なく、あくまで表示するときの問題なのです。  

