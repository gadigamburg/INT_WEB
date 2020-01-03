//@Library('Utilities') _
import groovy.json.JsonSlurper
import hudson.model.*
def BuildVersion
def Current_version
def NextVersion
def dev_rep_docker = 'gadigamburg/finalproject'
def colons = ':'
def module = 'intweb'
def underscore = '_'

pipeline {
    options {
        timeout(time: 30, unit: 'MINUTES')
    }
    agent { label 'slave' }
    stages {
         stage('Checkout') {
             steps {
                 script {
                     node('master'){
                         dir('Release') {
                             deleteDir()
                             checkout([$class: 'GitSCM', branches: [[name: 'gadi']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'GadiGamburg_GIT', url: "https://github.com/gadigamburg/Release.git"]]])
                             println("Test")
                             path_json_file = sh(script: "pwd", returnStdout: true).trim() + '/' + 'release' + '.json'
                             println("path_json_file: $path_json_file")
                             Current_version = Return_Json_From_File("$path_json_file").release.services.intweb.version
                             println("Current_version: $Current_version")
                         }
                     }
                     dir('INT_WEB') {
                         deleteDir()
                         checkout([$class: 'GitSCM', branches: [[name: 'gadi']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'GadiGamburg_GIT', url: "https://github.com/gadigamburg/INT_WEB.git"]]])
                         Commit_Id = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                         BuildVersion = Current_version + '_' + Commit_Id
                         println("BuildVersion: $BuildVersion")
                         last_digit_current_version = sh(script: "echo $Current_version | cut -d'.' -f1", returnStdout: true).trim()
                         println("Current last_digit_current_version: $last_digit_current_version")
                         NextVersion = sh(script: "echo $Current_version | cut -d. -f1", returnStdout: true).trim() + '.' + sh(script: "echo $Current_version |cut -d'.' -f2", returnStdout: true).trim() + '.' + (Integer.parseInt(last_digit_current_version) + 1)
                         println("Current the build version: $BuildVersion")
                         println("Next build version: $NextVersion")
                     }
                 }
             }
         }
         stage('UT') {
             steps {
                 println('UT will be added soon GADI')
             }
         }
         stage('Build') {
             steps {
                 script {
                     dir('INT_WEB') {
                         try {

                           docker.build("$module$colons$BuildVersion")
                           println("The build image is successfully")

                         }
                         catch (exception) {
                             println "The image build is failed"
                             currentBuild.result = 'FAILURE'
                             throw exception
                         }
                     }
                 }
             }
         }
         stage('Push image to repository'){
             steps{
                 script{
                     try{
                         withCredentials([usernamePassword(credentialsId: 'docker_hub', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                            println("DockerUser: ${DOCKER_USERNAME}")
                            println("DockerPass: ${DOCKER_PASSWORD}")
                            sh "docker login -u=${DOCKER_USERNAME} -p=${DOCKER_PASSWORD}"
                            sh "docker tag $module$colons$BuildVersion $dev_rep_docker$colons$module$underscore$BuildVersion"
                            sh "docker push $dev_rep_docker$colons$module$underscore$BuildVersion"
                         }
                     }
                     catch (exception){
                         println "Pushing image to DockerHub failed"
                         currentBuild.result = 'FAILURE'
                         throw exception
                     }
                 }
             }
         }
         stage('Push tag version to repository and update Release.json file'){
             steps{
                 script{
                     node('master'){
                         dir('Release') {
                             withCredentials([usernamePassword(credentialsId: 'GadiGamburg_GIT', passwordVariable: 'GIT_PASS', usernameVariable: 'GIT_USER')]) {
                                 sh "git checkout gadi"
                                 sh "sed -i 's/${Current_version}/${BuildVersion}/' release.json"
                                 sh "git config --global user.email 'gadi@gadi.com'"
                                 sh "git config --global user.name 'Jenkins'"
                                 sh "git add ${path_json_file}"
                                 sh "git commit -m 'CI approved ${NextVersion}'"
                                 sh "git push https://${GIT_USER}:${GIT_PASS}@github.com/gadigamburg/Release.git"
                             }
                         }
                     }
                 }
             }
         }
    }
}
def Return_Json_From_File(file_name){
    return new JsonSlurper().parse(new File(file_name))
}