## lambda_layer_libraries
aws sam とかサーバーレスFWを使うのめんどくさくなってきた  
FW使わずにやる場合どうするのか調べる

### Problem 
- Lambda関数実行するのに、ネイティブなライブラリ(gem)使おうとすると  
環境依存で実行できない
- いっぱい使ったらサイズがでかくなりがち
- 毎回gemごとパッケージングして上げ直すの面倒

### Try
- よく使うgemをLambda Layerで共通化

<hr>


#### [WIP] ハマりポイント覚書  

#### 1.  ネイティブライブラリのビルドはAWS側のLambda実行環境に合わせる必要がある
  - MacのHostで `bundle install` したNokogiriとかが関数実行時にコケるのはこのため
  - 専用のdockerイメージがあるので、Mac側のホストのディレクトリをマウントしつつ  
  docker内でビルドしたものを使う  
  ```shell
  docker run -v `pwd`:/var/task -it lambci/lambda:build-ruby2.7 bundle install --path ./
  ```
  
- Dockerイメージ  lambci/lambda  
https://hub.docker.com/r/lambci/lambda/tags?page=1&name=ruby


参考: Ruby on Lambdaで実現する、Eightの大規模画像処理基盤  
　　　　/native extensionsに依存する gem を使った Lambda 実装  
  
https://buildersbox.corp-sansan.com/entry/2019/04/17/110000  

#### 2.  Lambda側の環境変数で GEM_PATH を指定する必要がある  
Lambda実行環境下でgemが参照する標準のパスは `/opt/ruby/gems/2.5.0` 等で  
これを以下のパスに変更する必要がある  
  
**これが地味にわかりづらい...**  
出るエラーはgemが読み込めないしか出ないので
|キー|値|
|-|-|
|GEM_PATH|/opt/ruby/2.7.0|


参考: AWS Lambda layers - AWS Lambda  
https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html#configuration-layers-path

#### 3. Lambda実行サーバの時計がUTC
Lambda側の環境変数でタイムゾーンを指定すればOK
|キー|値|
|-|-|
|TZ|Asia/Tokyo|
