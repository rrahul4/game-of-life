#!/usr/bin/env groovy
import hudson.model.*
import hudson.EnvVars
import groovy.json.JsonSlurperClassic
import groovy.json.JsonBuilder
import groovy.json.JsonOutput
import java.net.URL

node{

properties([
    buildDiscarder(logRotator(artifactDaysToKeepStr: '7', artifactNumToKeepStr: '10', daysToKeepStr: '7', numToKeepStr: '10')),
    parameters([
        booleanParam(defaultValue: true, description: 'Application Compile ', name: 'Compile'),
        booleanParam(defaultValue: true, description: 'Application Review  ', name: 'Review'),
        booleanParam(defaultValue: true, description: 'Application Test    ', name: 'Test'),
        booleanParam(defaultValue: true, description: 'Application Metrics ', name: 'Metrics'),
        booleanParam(defaultValue: true, description: 'Application Package ', name: 'Package'),
        booleanParam(defaultValue: false, description: 'Nexus Push          ', name: 'NexusPush'),
        booleanParam(defaultValue: false, description: 'Ansible Deploy      ', name: 'AnsibleDeploy'),
        booleanParam(defaultValue: false, description: 'Docker Image Build  ', name: 'DockerImageBuild'),
        booleanParam(defaultValue: false, description: 'Docker Image Push   ', name: 'DockerImagePush')
        ])
    ])

    stage('Checkout'){
        cleanWs()
        git 'https://github.com/rrahul4/game-of-life'
    }        
    
    if (params.Compile) {
        stage('Compile'){
            withMaven(maven:'MyMaven'){
            sh 'mvn compile'
            }
        }        
    }

    if (params.Review) {
        stage('Review'){
            withMaven(maven:'MyMaven'){
            sh 'mvn pmd:pmd'
            pmd canComputeNew: false, defaultEncoding: '', healthy: '', pattern: 'gameoflife-web/target/pmd.xml', unHealthy: ''
            }
        }        
    }

    if (params.Test) {
        stage('Test')  {
            withMaven(maven:'MyMaven'){
            sh 'mvn test'
            junit 'gameoflife-web/target/surefire-reports/*.xml'
            }
        }        
    }

    if (params.Metrics) {
        stage('Metrics'){
            withMaven(maven:'MyMaven'){
            sh 'mvn cobertura:cobertura -Dcobertura.report.format=xml'
            cobertura autoUpdateHealth: false, autoUpdateStability: false, coberturaReportFile: 'gameoflife-web/target/site/cobertura/coverage.xml', conditionalCoverageTargets: '70, 0, 0', failUnhealthy: false, failUnstable: false, lineCoverageTargets: '80, 0, 0', maxNumberOfBuilds: 0, methodCoverageTargets: '80, 0, 0', onlyStable: false, sourceEncoding: 'ASCII', zoomCoverageChart: false
            }
        }        
    }

    if (params.Package) {
        stage('Package'){
           withMaven(maven:'MyMaven'){
            sh 'mvn package'
            }
        }        
    }
    
    if (params.NexusPush) {
        stage('Nexus Push'){
            nexusPublisher nexusInstanceId: 'MyNexus', nexusRepositoryId: 'Rahul_Nexus', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: 'gameoflife-web/target/gameoflife.war']], mavenCoordinate: [artifactId: '$BUILD_TIMESTAMP', groupId: 'GameOfLife', packaging: 'war', version: '$BUILD_ID']]]
        }
    }
   if (params.AnsibleDeploy) {
        stage('Ansible Deploy'){
            ansiblePlaybook installation: 'MyAnsible', playbook: 'tomcatinstall.yml'
        }
    }
    
    if (params.DockerImageBuild) {
        stage('Docker Image Build') {
            sh 'sudo docker build . -t rrahul4/gameoflife:$BUILD_NUMBER'
        }
    }
    
    if (params.DockerImagePush) {
        stage('Docker Image Push') {
            sh 'sudo docker push rrahul4/gameoflife:$BUILD_NUMBER'
        }
    }
}
