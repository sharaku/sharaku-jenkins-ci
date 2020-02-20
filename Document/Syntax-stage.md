# stage構文

## stage

実行するstageを定義します。
stageには以下の3種類があります。

- Single stage
- Parallel stage
- Subproject stage

それぞれ以下の動作を行います。

### Single stage
ノーマルなstage設定です。指定されたnode, env設定をもとにして、scriptを順次実行します。masterのファイルは指定されたnodeへ転送・展開され、実行後にresult設定に従ってmasterへ情報収集されます。

### Parallel stage
指定されたstageを並列で実行します。含めることのできるstageに制限はありません。このため、Parallel stageの中にParallel stageを含めることも可能です。ただし、循環するようなstage設定はガードしていないため、指定しないようにしてください。  
結果は、1つでもfailedの場合はfailedと判断されます。ただし、1つがfailedでも、実行中のstageを停止することはありません。

### Subproject stage
サブディレクトリ内の.jenkins-ci.ymlを読み込んで実行します。env設定は読み込んだ先のenv設定が優先されます。

## stage構文

#### 構文

```yaml
stage:
  <stage名>:
    <stageパラメータ>
```

#### 例

```yaml
stage:
  stage1:

  stage2:

```

## Single stage専用stageパラメータ構文

### node
stageを実行するnode名を指定します。指定しない場合はmasterで実行します。
特定のOSや環境で実行が必要な場合はnode名を指定してください。

#### 構文
```yaml
    node: <node名>
```
#### 例

```yaml
stage:
  stage1:
    node: master
  stage2:
    node: docker
```

### env

環境変数を定義します。この定義は```config: env:```の設定を上書きします。

#### 構文
```yaml
    env:
      - <env名> = <値>
      - <env名> = <値>
```

#### 例

```yaml
stage:
  stage1:
    env:
      - env_A=0
      - env_B=1
```

### docker

scriptの実行をdockerコンテナ上で実行します。  
引数で指定された値は```Docker Pipeline```の```withDockerContainer```へ渡されて解釈されます。

#### 構文
```yaml
    docker:
      image: <実行するコンテナimage名>
```

#### 例

```yaml
```
### script

指定されたスクリプトを実行します。

#### sh

shellを実行します。unix系のOSおよび、msysのnodeで有効です。

##### 構文
１行
```yaml
      - sh: <shellコマンド>
```

複数行
```yaml
      - sh: |
        <shellコマンド>
        <shellコマンド>
        <shellコマンド>
```

#### echo

jenkinsのコンソールログへ出力を行います。

##### 構文
```yaml
      - echo: <文字列>
```

#### powershell

powershellを実行します。windows系のnodeで有効です。

##### 構文
１行
```yaml
      - powershell: <powershellコマンド>
```

複数行
```yaml
      - powershell: |
        <powershellコマンド>
        <powershellコマンド>
        <powershellコマンド>
```

#### docker

dockerを使って操作を行います。
- ```docker build```は指定したDockerfileをbuildします。コマンドは指定nodeで行われます。
- ```docker exec```は指定したscriptをコンテナ上で実行します。stageの一部のみコンテナ上で実行する用途に使用します。
- ```docker run```は指定したコンテナイメージをdocker runコマンドで実行した後、指定されたscriptを指定node上で実行します。DBコンテナを実行し、DBを利用してテストを行うような動作に使用します。script終了後、コンテナは停止します。

##### 構文
docker build
```yaml
      - docker:
        command: build
        Dockerfile: <dockerfileパス>
        tag: <タグ>
```

docker exec
```yaml
      - docker:
        command: exec
        image: <実行するコンテナimage名>
        args: <コンテナ起動時の引数>
        script:
          {script}
```

docker run
```yaml
      - docker:
        command: build
        image: <実行するコンテナimage名>
        args: <コンテナ起動時の引数>
        script:
          {script}
```

## Parallel stage専用stageパラメータ構文

### parallel

指定されたstage一覧をを並列で実行します。

#### 構文

```yaml
    parallel:
      - <stage名>
      [- <stage名> ...]
```

#### 例

```yaml
stage:
  stage1:
    parallel:
      - stage1
      - stage2
```

## Subproject stage専用stageパラメータ構文

### subproject

指定されたサブディレクトリ内の.jenkins-ci.ymlを読み込んで実行します。

#### 構文

```yaml
    subproject: <サブディレクトリパス>
```

#### 例

```yaml
stage:
  stage1:
    subproject: subtest
```

## 共通stageパラメータ構文
### result
結果を収集し、masterノードへ転送します。

#### 構文

```yaml
      result: <結果を保存するpath>
```

#### 例

```yaml
stage:
  stage1:
    script:
      - sh: echo "result" > result/test_result.txt
    result: 'result/*'
```

### post

script実行後の動作を指定します。  
```always:``` では、stage終了後に必ず動作するスクリプトを記述する。  
```success:```では、stage成功時に動作するスクリプトを記述する。    
```failure:```では、stage失敗時に動作するスクリプトを記述する。  
スクリプト構文ははscriptを参照。

#### 構文

```yaml
    post:
      always:
        {script}
      success:
        {script}
      failure:
        {script}  
```

#### 例

```yaml
stage:
  stage1:
    script:
      - sh: ls /aaaaaaaaaaaaaaaaa
    post:
      always:
        - echo: always script
      success:
        - echo: success script
      failure:
        - echo: failure script
```
