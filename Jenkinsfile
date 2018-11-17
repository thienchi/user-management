#!/usr/bin/env groovy

node {

	// 1. Jenkins URL: http://ec2-54-254-226-241.ap-southeast-1.compute.amazonaws.com

	// 2. To lean Jenkins pipeline syntax: http://ec2-54-254-226-241.ap-southeast-1.compute.amazonaws.com/job/user-management-multibranch-pipeline/job/master/pipeline-syntax/

	// 3. Traefik: http://alb-1236468975.ap-southeast-1.elb.amazonaws.com/ui/dashboard/

	def branch = env.BRANCH_NAME
	def imageName = 'user-management'
	def imageTag = ''

	stage('PREPARATION') {
		// Delete all images, which have been built before!
		sh returnStatus: true, script: 'docker rmi -f $(docker images -q)'

		// Setup Java
		env.JAVA_HOME="${tool 'Java8'}"
		env.PATH="${env.JAVA_HOME}/bin:${env.PATH}"
		sh 'java -version'

		// Checkout branch
		// Need to set LocalBranch extension, since: https://stackoverflow.com/questions/44006070/jenkins-gitscm-finishes-the-clone-in-a-detached-head-state-how-can-i-make-sure
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

		imageTag = buildImageTagFromPomFile(branch)

		// Change build name
		currentBuild.displayName = imageTag
	}

	stage('BUILD PROJECT') {
		sh './mvnw clean install -P prod'
	}

	if (!currentBuild.result) {
		stage('BUILD IMAGE') {
			// TODO: write Dockerfile or you could use any plugin to build an image
			// IDEA: if you like Dockerfile https://spring.io/guides/gs/spring-boot-docker/

			// TODO: enable info endpoint (Spring Actuator), we will use later to verify that application is deployed successfully
			// IDEA: https://www.baeldung.com/spring-boot-actuators

			// TODO: build and tag an image
			// IDEA: use 'Shell Script' step to execute docker command
		}

	}


}

// GENERAL HELPERS

String buildImageTagFromPomFile(String branch) {
	def artifactVersion = fileExists('pom.xml') ? readMavenPom(file: 'pom.xml').version : ''
	artifactVersion = artifactVersion - '-SNAPSHOT'
	def gitRev = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()

	return "$artifactVersion-$branch-$gitRev"
}