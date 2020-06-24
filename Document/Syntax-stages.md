# stages構文

実行するstageを記述します。記述したstageを上から1つずつ順に実行します。

#### 構文

```yaml

stages:
  - <stage名>
  [- <stage名> ...]
```

#### 例

```yaml
stages:
  - stage1
  - stage2

stage:
  stage1:
    script:
      echo: stage1
  stage2:
    script:
      echo: stage1

```
