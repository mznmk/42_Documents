# 継続的インテグレーション

課題挑戦時(2021/5)における理解をまとめた。  


## 継続的インテグレーション(Continuous Integration)って？  

CI/CDパイプラインの一連の流れの一部分。  


### CI/CD パイプラインって？

継続的インテグレーション/継続的デリバリー パイプライン。  
新しいバージョンのアプリケーションをリリースするために実行する必要のある一連の流れのこと。  
これを自動で行えるようにしておくと、リリースまでの時間を短縮することができる。  
自動化を実現するには、一般的にはCI/CDツールを使う。  
(Jenkins, CircleCI, GitHub Actions, ...)  
継続的インテグレーション 継続的デリバリー 継続的デプロイメント、この境目は曖昧。  


### 継続的インテグレーション(Continuous Integration)って？

開発者が行ったアプリケーションへの変更に対して、バグがないかを自動的にテストする。  
事前にテストは書いておく。エラーが出たら、そこで処理はストップする。  


### 継続的デリバリー(Continuous Delivery)って？

継続的インテグレーション  
＋  
リポジトリへのアップロード  


### 継続的デプロイメント(Continuous Deployment)って？

継続的インテグレーション  
＋  
本番環境へのデプロイ  


## GitHub Actionsって？

CI/CDツール。  
GitHubにおける様々なイベントをトリガーにして、指定された作業を実行できる。  
実行したい内容を .yamlファイルに書き、それを指定されたディレクトリに配置するだけで使える。  
書き方次第で何でもできる。テストだけ行うというのも可能だし、テストからデプロイまで一気に行うこともできる。  


### 設定ファイルの書式

```
[workflow]
  └ jobs:
    └ [job]
       └ steps:
         └ [action]
```

#### [workflow]

.github/workflows/ に配置した .yamlファイル それぞれが1つのワークフローになる。  


#### [job]

job 毎に別のインスタンスで処理が実行される。  


#### [steps]

job の実行内容のまとまり。  


#### [action]

job の実行内容の最小単位。  
uses: 公開アクション(GitHub公式 / サードパーティ)を実行する。  
run: スクリプトを実行する。  

### 具体的利用例

次の設定だと、リポジトリへの push をトリガーにして、real-tests という job が走る。  
job は MacOS最新版で行われる。  
Repository から workspace にコードをコピーし、ビルドとテストを行う make test_bonus コマンドを実行している。  

```
name: real-tests

on: push	# <- GitHub Repository への push をトリガーにする

jobs:
  real-tests:			# <- Job 名
    runs-on: macos-latest	# <- MacOS 最新版 で作業が行われる。
    steps:

      - name: checkout git repository	# <- action 名
        uses: actions/checkout@v2	# <- リポジトリから workspece にコードをコピーする

      - name: run "make test_bonus"	# <- action 名
        run: bash -c "make test_bonus -C real-tests"	# <- ビルドとテストを行う make test_bonus コマンドを実行している
```