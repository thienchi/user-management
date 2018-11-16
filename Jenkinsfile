#!/usr/bin/env groovy

node {

	// 1. Jenkins URL: http://ec2-54-254-226-241.ap-southeast-1.compute.amazonaws.com

	// 2. To lean Jenkins pipeline syntax: http://ec2-54-254-226-241.ap-southeast-1.compute.amazonaws.com/job/user-management-multibranch-pipeline/job/master/pipeline-syntax/

	// 3. Traefik: http://alb-1236468975.ap-southeast-1.elb.amazonaws.com/ui/dashboard/

    def branch = 'nghia'
    def buildTag = ''

	stage('PREPARATION') {


	    env.JAVA_HOME="${tool 'Java8'}"
        env.PATH="${env.JAVA_HOME}/bin:${env.PATH}"
        sh 'java -version'

        checkout([$class: 'GitSCM',
		   branches: [[name: '*/nghia']],
		   doGenerateSubmoduleConfigurations: false,
		   extensions: [[$class: 'LocalBranch', localBranch: 'nghia'], [$class: 'CleanBeforeCheckout']],
		   submoduleCfg: [],
		   userRemoteConfigs: [[
		   credentialsId: 'tranductrinh',
		   url: 'https://github.com/tranductrinh/user-management.git']]])


		 buildTag = buildImageTagFromPomFile(branch)

		currentBuild.displayName = buildTag
	}

    stage('BUILD PROJECT') {
		sh './mvnw clean install'
	}

}

// GENERAL HELPERS

String buildImageTagFromPomFile(String branch) {
	def artifactVersion = fileExists('pom.xml') ? readMavenPom(file: 'pom.xml').version : ''
	artifactVersion = artifactVersion - '-SNAPSHOT'
	def gitRev = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()

	return "$artifactVersion-$branch-$gitRev"
}