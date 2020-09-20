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
