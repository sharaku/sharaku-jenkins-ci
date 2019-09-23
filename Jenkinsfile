#!groovy
// MIT License
// 
// Copyright (c) 2019 t.abe
// 
// Permission is hereby granted, free of charge, to any person obtaining a copy
// of this software and associated documentation files (the "Software"), to deal
// in the Software without restriction, including without limitation the rights
// to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
// copies of the Software, and to permit persons to whom the Software is
// furnished to do so, subject to the following conditions:
// 
// The above copyright notice and this permission notice shall be included in all
// copies or substantial portions of the Software.
// 
// THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
// IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
// FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
// AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
// LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
// OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
// SOFTWARE.
//
// 
// 必須プラグイン:
//   Pipeline Utility Steps
// Docker機能使用時に必要:
//   Docker Pipeline
// 解析機能を使用するときに必要:
//   Warnings Next Generation

// ---------------------------------------------------------------------
// スクリプトを実行する。
// def script	: 実行するスクリプト一覧 (map)
// ---------------------------------------------------------------------
def __exec_script(def script)
{
	if (script != null) {
		script.each { __script ->
			if (__script.sh != null) {
				// shellを実行する
				sh __script.sh
			} else if (__script.echo  != null) {
				// echoを実行する
				echo __script.echo
			} else if (__script.powershell  != null) {
				// powershellを実行する
				powershell __script.sh
			} else if (__script.docker  != null) {
				// dockerを実行する
				__exec_docker __script.docker
			} else if (__script.archiveArtifacts  != null) {
				// 成果物をアーカイブする
				archiveArtifacts __script.archiveArtifacts
			} else if (__script.stash  != null) {
				// 成果物をstashする
				stash __script.stash
			} else if (__script.unstash  != null) {
				// 成果物をunstashする
				unstash __script.stash
			} else if (__script.error  != null) {
				// エラーさせる
				error __script.error
			} else if (__script.failure  != null) {
				// エラーさせる
				echo __script.failure
				currentBuild.result = 'FAILURE'
			}
		}
	}
}

def __exec_docker(def docker_param)
{
	// docker buildを行う
	if (docker_param.command == "build") {
		if (docker_param.Dockerfile != null) {
			def param = "-f ${docker_param.Dockerfile} ."
			docker.build(docker_param.tag, param)
		} else {
			docker.build(docker_param.tag)
		}
	} else if (docker_param.command == "exec") {
		// execコマンドであった場合、dockerコンテナを起動してその中でスクリプトを動かす。
		// startコマンドであった場合、dockerコンテナを起動する。
		// これはデータベースをテスト用に立ち上げる場合に利用される。
		if (docker_param.script != null) {
			def param = [:]
			param["image"] = docker_param.image
			if (docker_param.args != null) {
				param["args"] = docker_param.args
			}
			withDockerContainer(param) {
				__exec_script(docker_param.script)
			}
		}
	} else if (docker_param.command == "run") {
		// runコマンドであった場合、dockerコンテナを起動する。
		// これはデータベースをテスト用に立ち上げる場合に利用される。
		// ここで起動したdockerは
		if (docker_param.script != null) {
			if (docker_param.args != null) {
				docker.image(docker_param.image).withRun(docker_param.args) {
					__exec_script(docker_param.script)
				}
			} else {
				docker.image(docker_param.image).withRun() {
					__exec_script(docker_param.script)
				}
			}
		}
	}

}

// ---------------------------------------------------------------------
// スクリプトを実行する。(環境変数指定があれば環境変数を設定する)
// def env		: 実行するスクリプトの環境変数 (list)
// def stage_param	: 実行するjobのパラメータ (map)
// ---------------------------------------------------------------------
def __exec_script_withEnv(def env, def stage_param)
{
	// ENV設定に従って設定を行う。
	if (env != "") {
		withEnv(env) {
			try {
				__exec_script(stage_param.script)
				if (stage_param.post != null && stage_param.post.success != null) {
					__exec_script(stage_param.post.success)
				}
			} catch (exc) {
				if (stage_param.post != null && stage_param.post.failure != null) {
					__exec_script(stage_param.post.failure)
				} else {
					currentBuild.result = 'FAILURE'
				}
			} finally {
				if (stage_param.post != null && stage_param.post.always != null) {
					__exec_script(stage_param.post.always)
				}
			}
		}
	} else {
		try {
			__exec_script(stage_param.script)
			if (stage_param.post != null && stage_param.post.success != null) {
				__exec_script(stage_param.post.success)
			}
		} catch (exc) {
			if (stage_param.post != null && stage_param.post.failure != null) {
				__exec_script(stage_param.post.failure)
			} else {
				currentBuild.result = 'FAILURE'
			}
		} finally {
			if (stage_param.post != null && stage_param.post.always != null) {
				__exec_script(stage_param.post.always)
			}
		}
	}
}

// ---------------------------------------------------------------------
// 1つのstageを実行する。
// def stage_name	: 実行するjobのname (string)
// def ow_env		: 実行するスクリプトの上書き用環境変数 (list)
// def stage_param	: 実行するjobのパラメータ (map)
// ---------------------------------------------------------------------
def __exec_scripts(def stage_name, def ow_env, def stage_param)
{
	def _node

	// 使用するnodeを決定する。
	// nodeが指定されていない場合は、masterを使用する。
	if (stage_param.node == null) {
		_node = "master"
	} else {
		_node = stage_param.node
	}

	// もし必須ではない場合は、nodeの存在をチェック
	if (stage_param.required != null && stage_param.required == false) {
		def _nodelist = []
		_nodelist = nodesByLabel _node
		if (_nodelist.size() == 0) {
			echo "SKIP '${stage_name}' stage."
			return 1
		}
	}

	node (_node) {
		// unstashが設定されている場合は全部実行する。
		deleteDir()
		if (stage_param.unstash != null) {
			stage_param.stash.each { __stash ->
				unstash __stash
			}
		} else {
			unstash '____initialize____'
		}

		// スクリプトがなければ何もしない。
		if (stage_param.script != null) {
			def __env = ""
			// 環境変数定義があれば、環境変数を設定してから、
			// shellを実行していく
			if (stage_param.env != null) {
				__env = stage_param.env
			}
			ow_env.each { line ->
				__env += line
			}

			// docker設定があれば、dockerを使用する。
			// Docker Pipelineの関数を使う
			if (stage_param.docker != null) {
				withDockerContainer(stage_param.docker) {
					__exec_script_withEnv(__env, stage_param)
				}
			} else {
				__exec_script_withEnv(__env, stage_param)
			}
		}

		// stashはそのまま渡す。
		// よって、name, excludesを設定すること。
		if (stage_param.stash != null) {
			stage_param.stash.each { __stash ->
				stash __stash
			}
		}

		// 結果をstashする。
		if (stage_param.result != null) {
			stash allowEmpty: true, includes: stage_param.result, name: "____result_${stage_name}____"
		}
	}
	return 0
}



// ---------------------------------------------------------------------
// 1つのスクリプトの塊を実行する。
// def stage_name	: 実行するjobのname (string)
// def ow_env		: 実行するスクリプトの上書き用環境変数 (list)
// def stage_param	: 実行するjobのパラメータ (map)
// ---------------------------------------------------------------------
def __exec_subproject(def stage_name, def ow_env, def stage_param)
{
	def __env = []
	def __yaml = ".jenkins-ci.yml"

	// サブプロジェクトの設定ファイルを読み込む
	// Pipeline Utility Steps Pluginの関数を使う
	yaml = readYaml(file: "${stage_param.subproject}/${__yaml}")

	if (yaml.config.env != null) {
		__env = yaml.config.env
	}
	__env += "JOBPATH=${stage_param.subproject}/"

	withEnv(__env) {
		__exec_stages(yaml.stages, yaml.stage)
	}
}



// ---------------------------------------------------------------------
// stagesを実行する。
// ---------------------------------------------------------------------
// 行は次のフォーマットとする
// {jobname} [env:xxx=xxx,xxx=xxx]
def __mk_env(def list)
{
	def __ow_env=[]

	list.each { __line ->
		def vals = __line.split(":")
		if (vals[0] == "env") {
			vals[1].split(",").each { __env ->
				__ow_env += __env
			}
		}
	}
	return __ow_env
}

def __exec_stages(def stages, def stage_list)
{
	withEnv(["JOBPATH="]) {
		stages.each { __line ->
			def __line_split = __line.split(" ")
			def __job = __line_split[0]
			def __ow_env=[]

			// 追加引数リストを作る
			__ow_env = __mk_env(__line_split)

			if (stage_list[__job] == null) {
				// 指定されたjobはありませんでした。
				echo "${__job} is not found."
			} else {
				stage(__job) {
					if (stage_list[__job].parallel != null) {
						__exec_parallel(__job, stage_list, __ow_env, stage_list[__job])
					} else if (stage_list[__job].subproject != null) {
						__exec_subproject(__job, __ow_env, stage_list[__job])
					} else {
						def res = __exec_scripts(__job, __ow_env, stage_list[__job])
						if (res == 0) {
							if (stage_list[__job].result != null) {
								unstash "____result_${__job}____"
							}
						}
					}
				}
			}
		}
	}
}



// ---------------------------------------------------------------------
// 1つのstage（parallel）を実行する。
// ---------------------------------------------------------------------
def __exec_parallel(def stage_name, def stage_list, def ow_env, def stage_param)
{
	def __parallel = [:]

	stage_param.parallel.each { __line ->
		__parallel[__line] = {
			stage(__line) {
				if (stage_list[__line].parallel != null) {
					__exec_parallel(__line, stage_list, ow_env, stage_list[__line])
				} else if (stage_list[__line].subproject != null) {
					__exec_subproject(__line, ow_env, stage_list[__line])
				} else {
					__exec_scripts(__line, ow_env, stage_list[__line])
				}
			}
		}
	}

	stage(stage_name) {
		try {
			parallel(__parallel)
			if (stage_param.post != null && stage_param.post.success != null) {
				__exec_script(stage_param.post.success)
			}
		} catch (exc) {
			if (stage_param.post != null && stage_param.post.failure != null) {
				__exec_script(stage_param.post.failure)
			} else {
				currentBuild.result = 'FAILURE'
			}
		} finally {
			if (stage_param.post != null && stage_param.post.always != null) {
				__exec_script(stage_param.post.always)
			}
		}
	}

	stage_param.parallel.each { __line ->
		if (stage_list[__line].result != null) {
			unstash "____result_${__line}____"
		}
	}
}



// ---------------------------------------------------------------------
// 実行後の集計処理
// ---------------------------------------------------------------------
def __issues(def yaml)
{
	def __tools = [:]
	def __tool_one = ""

	if (yaml.config.issues == null) {
		return
	}

	// junitを実行する
	if (yaml.config.issues.junit != null) {
		junit yaml.config.issues.junit
	}

	// --------------------------------------
	// Warnings Next Generation プラグイン
	// --------------------------------------
	// cmakeを実行する
	if (yaml.config.issues.cmake != null) {
		recordIssues(aggregatingResults: true, tools: [cmake( yaml.config.issues.cmake )])
	}
	// cppcheckを実行する
	if (yaml.config.issues.cppcheck != null) {
		recordIssues(aggregatingResults: true, tools: [cppCheck( yaml.config.issues.cppcheck )])
	}
	// cpplintを実行する
	if (yaml.config.issues.cpplint != null) {
		recordIssues(aggregatingResults: true, tools: [cppLint( yaml.config.issues.cpplint )])
	}
	// doxygenを実行する
	if (yaml.config.issues.doxygen != null) {
		recordIssues(aggregatingResults: true, tools: [doxygen( yaml.config.issues.doxygen )])
	}
	// gcc3を実行する
	if (yaml.config.issues.gcc3 != null) {
		recordIssues(aggregatingResults: true, tools: [gcc3( yaml.config.issues.gcc3 )])
	}
	// gccを実行する
	if (yaml.config.issues.gcc != null) {
		recordIssues(aggregatingResults: true, tools: [gcc( yaml.config.issues.gcc )])
	}
}



// *********************************************************************
// ここからがエントリ。
// *********************************************************************
node {
	def yaml

	// clean checkoutする
	checkout scm
	stash name: '____initialize____'

	script {
		// 設定ファイルを読み込む
		// Pipeline Utility Steps Pluginの関数を使う
		yaml = readYaml(file: ".jenkins-ci.yml")

		timestamps {
			// 環境変数定義がある場合は環境変数を設定する。
			// 上書き用の環境変数定義があれば設定する。
			def __env = []
			def __stages = []

			if (yaml.config.env != null) {
				__env = yaml.config.env
			}
			if (params.env != null) {
				def __ow_env = params.env.split("\n")
				__ow_env.each { line ->
					__env += line
				}
			}

			// job一覧の上書き設定がある場合は上書きする。
			// params.stagesは、環境変数設定も同時に入ってくるので、
			// 分離してリストにする。
			if (yaml.stages != null) {
				__stages = yaml.stages
			}
			if (params.stages != null && params.stages != "") {
				def __ow_stages = params.stages.split("\n")
				__ow_stages.each { line ->
					__stages += line
				}
			}

			if (__env != []) {
				withEnv(__env) {
					__exec_stages(__stages, yaml.stage)
				}
			} else {
				__exec_stages(__stages, yaml.stage)
			}

		}

		__issues(yaml)

		// archiveArtifactsを使って成果物を保存する
		if (yaml.config.archiveArtifacts != null) {
			archiveArtifacts yaml.config.archiveArtifacts
		}
	}
}

