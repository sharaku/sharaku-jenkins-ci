# cppcheck for docker

dockerを使用してcppcheckを実行するstage例です。

node項目にはdockerを実行可能なnodeを指定します。  
`docker: image:`には`sharaku/cppcheck`を指定します。  
`script:`では、cppcheckを実行するコードを定義します。  
`result:`では、masterへ転送するパスを指定します。  
`config: cppcheck: pattern:`には、masterに転送されたcppcheckの結果を指定します。

#### 例

以下の場合の設定例です。

|項目|値|
|-|-|
|dockerを実行するノード |docker|
|cppcheckの対象 |src|
|cppcheckの出力結果 |result/cppcheck.xml|

```yaml
config:
  cppcheck:
    pattern: 'result/*.xml'

stage:
  cppcheck:
    node: docker
    docker:
      image: sharaku/cppcheck
    script:
     - sh: cppcheck --enable=all --xml src 2> result/cppcheck.xml
    result: 'result/*'
```
