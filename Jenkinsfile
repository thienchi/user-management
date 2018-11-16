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

	if (!currentBuild.result) {
    		stage('BUILD IMAGE') {
    			// TODO: write Dockerfile or you could use any plugin to build an image
    			// IDEA: if you like Dockerfile https://spring.io/guides/gs/spring-boot-docker/

    			// TODO: enable info endpoint (Spring Actuator), we will use later to verify that application is deployed successfully
    			// IDEA: https://www.baeldung.com/spring-boot-actuators

    			// TODO: build and tag an image
    			// IDEA: use 'Shell Script' step to execute docker command

    			sh "docker build -t $buildTag ./backend"
    		}
    		stage('PUSH IMAGE') {
                // TODO: push image to Amazon ECR
                // IDEA: use 'Shell Script' step to execute docker command

                // TODO: in order to push to Amazon ECR, we need to login to the repository!
                // use 'withDockerRegistry' step, we have constant AWS_ECR_REPOSITORY_URL and AWS_ECR_CREDENTIALS_ID
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

// AMAZON HELPERS
String accountId() {
	def getCallerIdentityCmd = "aws sts get-caller-identity"
	println "Executing get caller identity cmd: ${getCallerIdentityCmd}"

	def getCallerIdentityResponse = sh returnStdout: true, script: getCallerIdentityCmd
	println "Response of get caller identity cmd: ${getCallerIdentityResponse}"

	def getCallerIdentityJson = readJSON text: getCallerIdentityResponse

	return getCallerIdentityJson.Account
}