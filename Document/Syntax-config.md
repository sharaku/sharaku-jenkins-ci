# config構文

## env

環境変数を定義します。

#### 構文

#### 例

```yaml
config:
  env:
    - env_A=0
    - env_B=1
```

## issues

jenkins結果の問題収集を行う。

### junit

junitの結果を収集する

#### 構文

#### 例

```yaml
config:
  issues:
    junit:
      allowEmptyResults: true
      keepLongStdio: true
      testResults: 'result/*.xml'
```
### cmake

cmakeの結果を収集する

#### 構文

#### 例

```yaml
config:
  issues:
    cmake:
      reportEncoding: 'UTF-8'
      pattern: 'result/cmake.*'
```
### cppcheck

cppcheckの結果を収集する

#### 構文

#### 例

```yaml
config:
  issues:
    cppcheck:
      reportEncoding: 'UTF-8'
      pattern: 'result/cppcheck.*.xml'
```

### cpplint

cpplintの結果を収集する

#### 構文

#### 例

```yaml
config:
  issues:
    cpplint:
      reportEncoding: 'UTF-8'
      pattern: 'result/cpplint.*'
```

### doxygen

doxygenの結果を収集する

#### 構文

#### 例

```yaml
config:
  issues:
    doxygen:
      reportEncoding: 'UTF-8'
      pattern: 'result/doxygen.*'
```

### gcc3

gcc3の結果を収集する

#### 構文

#### 例

```yaml
config:
  issues:
    gcc3:
      reportEncoding: 'UTF-8'
      pattern: 'result/gcc3.*'
```

### gcc

gccの結果を収集する

#### 構文

#### 例

```yaml
config:
  issues:
    gcc3:
      reportEncoding: 'UTF-8'
      pattern: 'result/gcc.*'
```

## archiveArtifacts

jenkinsの結果を保存する。

#### 構文

#### 例

```yaml
config:
  archiveArtifacts:
    artifacts: 'result/*'
```
