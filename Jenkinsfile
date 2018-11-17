#!/usr/bin/env groovy

node {

	// 1. Jenkins URL: http://ec2-54-254-226-241.ap-southeast-1.compute.amazonaws.com

	// 2. To lean Jenkins pipeline syntax: http://ec2-54-254-226-241.ap-southeast-1.compute.amazonaws.com/job/user-management-multibranch-pipeline/job/master/pipeline-syntax/

	// 3. Traefik: http://alb-1236468975.ap-southeast-1.elb.amazonaws.com/ui/dashboard/

	def branchname = env.BRANCH_NAME

	stage('PREPARATION') {
		// TOTO: clean up docker images which were built before
		// IDEA: use 'Shell Script' step to remove all docker images
		sh returnStatus: true, script: 'docker rmi -f $(docker images -q)'

		// TODO: setup tools: Java, Maven...
		// IDEA: use 'Tool' step to get path of installed Java, then set Java path into env.PATH
		// Setup Java
		env.JAVA_HOME="${tool 'Java8'}"
		env.PATH="${env.JAVA_HOME}/bin:${env.PATH}"
		sh 'java -version'

		// TODO: checkout project, please remember to checkout only your branch!
		// IDEA: use 'Checkout' step
		checkout([$class: 'GitSCM', branches: [[name: '*/$branchname']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'ci-user-ssh', url: 'git@github.com:tranductrinh/user-management.git']]])


		// TODO: build image tag, later we will use this tag to tag docker image in this build
		// IDEA: some of global variables that might interesting!
		// http://ec2-54-254-226-241.ap-southeast-1.compute.amazonaws.com/job/user-management-multibranch-pipeline/job/master/pipeline-syntax/globals

		// TODO: may be change display name of this build to display image tab
		// IDEA: currentBuild.displayName in http://ec2-54-254-226-241.ap-southeast-1.compute.amazonaws.com/job/user-management-multibranch-pipeline/job/master/pipeline-syntax/globals#currentBuild
	}

}