# sharaku-Jenkins-ci
Jenkins CIツール

## sharaku-Jenkins-ciとは

jenkinsのpipelineはgitlabのようにyamlで簡単に設定できない。その代わり、groovyで細かなパイプラインを構築できる。この自由度の高いjenkins pipelineの特徴を持ちつつ、yamlでの簡単設定を行えるようにしたものがsharaku-Jenkins-ciです。

## sharaku-Jenkins-ciの特徴

以下を行えます。
- yamlファイルによるpipelineの設定
- jenkinsの"ビルドのパラメータ化"を利用して、以下を実行ごとに変更可能
 - 環境変数
 - 実行jobの変更
