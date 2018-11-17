#!/usr/bin/env groovy

node {

	// 1. Jenkins URL: http://ec2-54-254-226-241.ap-southeast-1.compute.amazonaws.com

	// 2. To lean Jenkins pipeline syntax: http://ec2-54-254-226-241.ap-southeast-1.compute.amazonaws.com/job/user-management-multibranch-pipeline/job/master/pipeline-syntax/

	// 3. Traefik: http://alb-1236468975.ap-southeast-1.elb.amazonaws.com/ui/dashboard/

	stage('HELLO PIPELINE') {
		println "Hello, this is my first pipeline!"
	}
	stage('PREPARATION'){
		//set up java
		env.JAVA_HOME = "${tool name: 'Java8', type: 'jdk'}"
		env.PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
		def branch = env.BRANCH_NAME

		checkout([$class: 'GitSCM',
				branches: [
					[
						name: "*/$branch"
					]
				],
				doGenerateSubmoduleConfigurations: false,
				extensions: [],
				extensions: [
					[$class: 'CleanBeforeCheckout'],
					[$class: 'LocalBranch', localBranch: "**"]
				],
				submoduleCfg: [],
				userRemoteConfigs: [
					[
						credentialsId: 'ci-user-ssh',
						url: 'git@github.com:tranductrinh/user-management.git'
					]
				]
			])

		//setup currentBuild:
		def artifactVersion = fileExists('pom.xml') ? readMavenPom(file:"pom.xml").version : '' 
		def gitrev = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
		currentBuild.displayName = "$artifactVersion-$branch-$gitrev"

	}
	stage('BUILD PROJECT'){
		sh './mvnw clean install'
	}
}

