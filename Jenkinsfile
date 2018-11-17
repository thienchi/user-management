#!/usr/bin/env groovy

node {

    // 1. Jenkins URL: http://ec2-54-254-226-241.ap-southeast-1.compute.amazonaws.com

    // 2. To lean Jenkins pipeline syntax: http://ec2-54-254-226-241.ap-southeast-1.compute.amazonaws.com/job/user-management-multibranch-pipeline/job/master/pipeline-syntax/

    // 3. Traefik: http://alb-1236468975.ap-southeast-1.elb.amazonaws.com/ui/dashboard/
    withCredentials([[$class           : 'AmazonWebServicesCredentialsBinding',
                      accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                      credentialsId    : 'aws',
                      secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {

        // CONSTANTS
        def AWS_ACCOUNT_ID = accountId()
        def AWS_DEFAULT_REGION = 'ap-southeast-1'
        def AWS_ECR_REPOSITORY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
        def AWS_ECR_REPOSITORY_URL = "https://${AWS_ECR_REPOSITORY}"
        def AWS_ECR_CREDENTIALS_ID = "ecr:${AWS_DEFAULT_REGION}:aws"

        def imageName = 'user-management'
        def imageTag = ''


        stage('PREPARATION') {
            // TOTO: clean up docker images which were built before
            // IDEA: use 'Shell Script' step to remove all docker images
            sh returnStatus: true, script: 'docker rmi -f $(docker images -q)'

            // TODO: setup tools: Java, Maven...
            // IDEA: use 'Tool' step to get path of installed Java, then set Java path into env.PATH
            env.JAVA_HOME = "${tool name: 'Java8', type: 'jdk'}"
            env.PATH = "${env.JAVA_HOME}/bin:${env.PATH}"

            // TODO: checkout project, please remember to checkout only your branch!
            // IDEA: use 'Checkout' step
            checkout([
                    $class                           : 'GitSCM',
                    branches                         : [
                            [name: '*/nvlinh']
                    ],
                    doGenerateSubmoduleConfigurations: false,
                    extensions                       : [],
                    submoduleCfg                     : [],
                    userRemoteConfigs                : [[credentialsId: 'ci-user-ssh', url: 'https://github.com/tranductrinh/user-management.git']]
            ])

            // TODO: build image tag, later we will use this tag to tag docker image in this build
            // IDEA: some of global variables that might interesting!
            // http://ec2-54-254-226-241.ap-southeast-1.compute.amazonaws.com/job/user-management-multibranch-pipeline/job/master/pipeline-syntax/globals
            imageTag = "$BRANCH_NAME-$BUILD_NUMBER"

            // TODO: may be change display name of this build to display image tab
            // IDEA: currentBuild.displayName in http://ec2-54-254-226-241.ap-southeast-1.compute.amazonaws.com/job/user-management-multibranch-pipeline/job/master/pipeline-syntax/globals#currentBuild
            currentBuild.displayName = imageTag
        }

        stage('BUILD PROJECT') {
            // TODO: execute maven build
            // IDEA: use 'Shell Script' step, and also see README.md - how to build project
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

                sh "docker build -t $imageName:$imageTag ./docker"
                sh "docker tag $imageName:$imageTag ${AWS_ECR_REPOSITORY}/$imageName:$imageTag"
            }
        }
    }
}