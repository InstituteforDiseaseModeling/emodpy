podTemplate(cloud: 'OpenShift', label: 'rpmbuild') {
  node('rpm-based') {
    container('dtk-build'){
			stage('Cleanup Workspace') {		    
				cleanWs()
				echo "Cleaned Up Workspace For Project"
				echo "${params.BRANCH}"
			}
			stage('Prepare') {
				sh 'python --version'
				sh 'python3.6 --version'
				sh 'pip3.6 --version'

				sh 'python3.6 -m pip install --upgrade pip'
				//sh 'python3.6 -m pip install --upgrade wheel'
				sh "pip3.6 install wheel"
				sh 'python3.6 -m pip install --upgrade setuptools'
				sh 'pip3.6 freeze'
				sh 'yum -y remove mpich'
				sh 'yum -y install mpich-3.2'
				sh 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib64/mpich/lib'
			}
			stage('Code Checkout') {
				if (env.CHANGE_ID) {
					echo "I execute on the pull request ${env.CHANGE_ID}"
					checkout([$class: 'GitSCM',
					branches: [[name: "pr/${env.CHANGE_ID}/head"]],
					doGenerateSubmoduleConfigurations: false,
					extensions: [],
					gitTool: 'Default',
					submoduleCfg: [],
					userRemoteConfigs: [[refspec: '+refs/pull/*:refs/remotes/origin/pr/*', credentialsId: '704061ca-54ca-4aec-b5ce-ddc7e9eab0f2', url: 'git@github.com:InstituteforDiseaseModeling/emodpy.git']]])
				} else {
					echo "I execute on the ${env.BRANCH_NAME} branch"
					git branch: "${env.BRANCH_NAME}",
					credentialsId: '704061ca-54ca-4aec-b5ce-ddc7e9eab0f2',
					url: 'git@github.com:InstituteforDiseaseModeling/emodpy.git'
				}
			}
			stage('Build') {
				sh 'pwd'
				sh 'ls -a'
				sh 'python3.6 setup.py bdist_wheel'
				 
			}
			stage('Install') {
				def curDate = sh(returnStdout: true, script: "date").trim()
				echo "The current date is ${curDate}"

				echo "I am installing emodpy from wheel file built from code"
				def wheelFile = sh(returnStdout: true, script: "find ./dist -name '*.whl'").toString().trim()
				//def wheelFile = sh(returnStdout: true, script: "python3.6 ./.github/scripts/get_wheel_filename.py --package-file package_setup.py").toString().trim()
				echo "This is the package file: ${wheelFile}"
				sh "pip3.6 install $wheelFile --index-url=https://packages.idmod.org/api/pypi/pypi-production/simple"
				 
				sh "pip3.6 install py-make"
				sh "pip3.6 install dataclasses"
				sh "pip3.6 install idmtools_test --index-url=https://packages.idmod.org/api/pypi/pypi-production/simple"
				sh 'pip3.6 install keyrings.alt'
				sh "pip3.6 freeze"
			}
			stage('Login and Test') {
				withCredentials([string(credentialsId: 'Comps_emodpy_user', variable: 'user'), string(credentialsId: 'Comps_emodpy_password', variable: 'password'),
				                 string(credentialsId: 'Bamboo_id', variable: 'bamboo_user'), string(credentialsId: 'Bamboo', variable: 'bamboo_password')]) {
					sh 'python3.6 ".dev_scripts/create_auth_token_args.py" --comps_url https://comps2.idmod.org --username $user --password $password'
					dir('tests') {
					    sh 'python3.6 bamboo_login_with_arguments.py -u $bamboo_user -p $bamboo_password'
				    }
				}
				echo "Running Emodpy Tests"
				dir('tests') {
					sh 'python3.6 --version'
					sh "pytest test_download_from_bamboo.py"
					sh 'pytest -v -m emod --junitxml="result.xml"'
				    junit '*.xml'
				}
			}
		}
	}
  }