#!groovy


// ---------------------------------------------------------------------
// スクリプトを実行する。
// ---------------------------------------------------------------------
def __exec_script(def script)
{
	if (script != null) {
		script.each { __script ->
			if (__script.sh != null) {
				sh __script.sh
			} else if (__script.echo  != null) {
				echo __script.echo
			} else if (__script.powershell  != null) {
				powershell __script.sh
			}
		}
	}
}



// ---------------------------------------------------------------------
// 1つのstageを実行する。
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

			if (__env != "") {
				withEnv(__env) {
					__exec_script(stage_param.script)
				}
			} else {
				__exec_script(stage_param.script)
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
}



// ---------------------------------------------------------------------
// 1つのスクリプトの塊を実行する。
// ---------------------------------------------------------------------
def __exec_single_stage(def stage_name, def ow_env, def stage_param)
{
	__exec_scripts(stage_name, ow_env, stage_param)
}



// ---------------------------------------------------------------------
// 1つのスクリプトの塊を実行する。
// ---------------------------------------------------------------------
def __exec_subproject(def stage_name, def ow_env, def stage_param)
{
	def __env = []

	// サブプロジェクトの設定ファイルを読み込む
	// Pipeline Utility Steps Pluginの関数を使う
	yaml = readYaml(file: "${stage_param.subproject}/config.yml")

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
						__exec_single_stage(__job, __ow_env, stage_list[__job])
						if (stage_list[__job].result != null) {
							unstash "____result_${__job}____"
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
					__exec_single_stage(__line, ow_env, stage_list[__line])
				}
			}
		}
	}

	stage(stage_name) {
		parallel(__parallel)
	}

	stage_param.parallel.each { __line ->
		if (stage_list[__line].result != null) {
			unstash "____result_${__line}____"
		}
	}
}



// *********************************************************************
// ここからがエントリ。
// *********************************************************************
node {
	def yaml

	// clean checkoutする
	deleteDir()
	checkout scm
	stash name: '____initialize____'

	script {
		// 設定ファイルを読み込む
		// Pipeline Utility Steps Pluginの関数を使う
		yaml = readYaml(file: 'config.yml')

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

		// junitを実行する
		if (yaml.config.junit != null) {
			junit yaml.config.junit
		}

		// archiveArtifactsを使って成果物を保存する
		if (yaml.config.archiveArtifacts != null) {
			archiveArtifacts yaml.config.archiveArtifacts
		}
	}
}

