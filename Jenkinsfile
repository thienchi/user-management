#!/usr/bin/env groovy

node {

	// 1. Jenkins URL: http://ec2-54-254-226-241.ap-southeast-1.compute.amazonaws.com

	// 2. To lean Jenkins pipeline syntax: http://ec2-54-254-226-241.ap-southeast-1.compute.amazonaws.com/job/user-management-multibranch-pipeline/job/master/pipeline-syntax/

	// 3. Traefik: http://alb-1236468975.ap-southeast-1.elb.amazonaws.com/ui/dashboard/
    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
        credentialsId: 'aws',
        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {

        // CONSTANTS
        def AWS_ACCOUNT_ID = accountId()
        def AWS_DEFAULT_REGION = 'ap-southeast-1'
        def AWS_ECR_REPOSITORY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
        def AWS_ECR_REPOSITORY_URL = "https://${AWS_ECR_REPOSITORY}"
        def AWS_ECR_CREDENTIALS_ID = "ecr:${AWS_DEFAULT_REGION}:aws"


        def branch = 'nghia'
        def buildTag = ''
        def imageName = 'user-management'

        stage('PREPARATION') {
            // Clean up
            sh returnStatus: true, script: 'docker rmi -f $(docker images "*nghia*" -q)'

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

                    sh "docker build -t $imageName:$buildTag ./backend"
                    sh "docker tag $imageName:$buildTag ${AWS_ECR_REPOSITORY}/$imageName:$buildTag"
                }
                stage('PUSH IMAGE') {
                    // use 'withDockerRegistry' step, we have constant AWS_ECR_REPOSITORY_URL and AWS_ECR_CREDENTIALS_ID
                    withDockerRegistry(credentialsId: AWS_ECR_CREDENTIALS_ID, url: AWS_ECR_REPOSITORY_URL) {
                        sh "docker push ${AWS_ECR_REPOSITORY}/$imageName:$buildTag"
                    }
                }

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
