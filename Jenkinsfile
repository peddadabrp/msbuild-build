#!/usr/bin/env groovy

node {
    
    def mvnHome
    mvnHome = tool 'M2'
    def anthome
    def gradlehome
    checkout scm
    if (fileExists("${WORKSPACE}/gradlew")) {
        sh "echo ###-------GRADLE BUILD-------###"
        Gradle()
    } else if (fileExists("${WORKSPACE}/build.xml")) {
        sh "echo ###-------ANT BUILD-------###"
        Ant()
    } else if (fileExists("${WORKSPACE}/Calculator.sln")) {
        sh "echo ###-------MSBUILD------###"
        Msbuild()
    } else if (fileExists("${WORKSPACE}/test4.txt")) {
        sh "cat $WORKSPACE/test4.txt"
        sh "rm -rf $WORKSPACE/test4.txt"
    } else if (fileExists("${WORKSPACE}/test.txt")) {
        sh "cat $WORKSPACE/test.txt"
        sh "rm -rf $WORKSPACE/test.txt"
    } else {
        sh "echo 'no files found'"
    }
}


def Gradle() {
    stage ('Preparation') {PreparationGradleEnv() }
    stage ('Checkout') {Checkout() }
    stage ('Gradle build') {GradleBuild() }
    stage ('Unit Test') {Sleeping() }
    stage ('Static Code Analysis') {CodeTest() }
    stage ('Publish to Artifactory') {UploadArtifact() }
    stage ('Download Artifact') {Downloadartifact() }
    stage ('Deploy QA') {DeployQA() }
}

def Ant() {

    stage ('Preparation') {PreparationANTEnv() }
    stage ('Checkout') {Checkout() }
    stage ('Ant build') {AntBuild() }
    stage ('Unit Test') {Sleeping() }
    stage ('Static Code Analysis') {CodeTest() }
    stage ('Publish to Artifactory') {UploadArtifact() }
    stage ('Download Artifact') {Downloadartifact() }
    stage ('Deploy QA') {DeployQA() }
}

def Msbuild() {
    stage ('Preparation') {PreparationMsbuildEnv() }
    stage ('Checkout') {Checkout() }
    stage ('MSbuild') {MsBuild() }
    stage ('Unit Test') {Sleeping() }
    stage ('Static Code Analysis') {CodeTest() }
    stage ('Publish to Artifactory') {UploadArtifact() }
    stage ('Download Artifact') {Downloadartifact() }
    stage ('Deploy QA') {DeployQA() }
}

def PreparationGradleEnv() {
    
    deleteDir()
    
    URL_SOURCE = 'https://github.com/geb/geb-example-gradle.git'

    SCM_BRANCH = '*/master'

    // Automation Dashboard URL
    URL_DASHBOARD = "${env.BUILD_URL}artifact/dashboard.htm"

    // Credentials configurations with respect to Jenkins
    CREDENTIALS = '6152ec42-b301-4820-a1ea-582595560005'

    //Artifactory server details
    SERVER_ID = 'AP5P5hu1iVpC7PnPLwoSg6YCs3k'
    ARTIFACTORY_PATTERN = "${WORKSPACE}/**.tar.gz"
    ARTIFACTORY_TARGET = "libs-snapshot-local/${JOB_NAME}/${BUILD_NUMBER}/"

    // Release Details
    PROJECT_NAME = 'sample-project'
    RELEASE_VERSION = '1.0'
    ARTIFACT_NAME = "$PROJECT_NAME-${JOB_NAME}-$RELEASE_VERSION.${BUILD_NUMBER}.tar.gz"

}

def PreparationANTEnv() {

    deleteDir()
    
    URL_SOURCE = 'https://github.com/peddadabrp/ant-build.git'

    SCM_BRANCH = '*/master'

    // Automation Dashboard URL
    URL_DASHBOARD = "${env.BUILD_URL}artifact/dashboard.htm"

    // Credentials configurations with respect to Jenkins
    CREDENTIALS = '6152ec42-b301-4820-a1ea-582595560005'

    //Artifactory server details
    SERVER_ID = 'AP5P5hu1iVpC7PnPLwoSg6YCs3k'
    ARTIFACTORY_PATTERN = "${WORKSPACE}/build/jar/**.jar"
    ARTIFACTORY_TARGET = "libs-snapshot-local/${JOB_NAME}/${BUILD_NUMBER}/"

    // Release Details
    PROJECT_NAME = 'sample-project'
    RELEASE_VERSION = '1.0'
    ARTIFACT_NAME = "$PROJECT_NAME-${JOB_NAME}-$RELEASE_VERSION.${BUILD_NUMBER}.jar"

}

def PreparationMsbuildEnv() {

    deleteDir()
    
    URL_SOURCE = 'https://github.com/peddadabrp/msbuild-build.git'

    SCM_BRANCH = '*/master'

    // Automation Dashboard URL
    URL_DASHBOARD = "${env.BUILD_URL}artifact/dashboard.htm"

    // Credentials configurations with respect to Jenkins
    CREDENTIALS = '6152ec42-b301-4820-a1ea-582595560005'

    //Artifactory server details
    SERVER_ID = 'AP5P5hu1iVpC7PnPLwoSg6YCs3k'
    ARTIFACTORY_PATTERN = "${WORKSPACE}/**.sln"
    ARTIFACTORY_TARGET = "libs-snapshot-local/${JOB_NAME}/${BUILD_NUMBER}/"

    // Release Details
    PROJECT_NAME = 'sample-project'
    RELEASE_VERSION = '1.0'
    ARTIFACT_NAME = "$PROJECT_NAME-${JOB_NAME}-$RELEASE_VERSION.${BUILD_NUMBER}.sln"

}

def Checkout() {
    checkout([$class: 'GitSCM',
        branches: [[name: "$SCM_BRANCH"]],
        doGenerateSubmoduleConfigurations: false,
        extensions: [], submoduleCfg: [],
        userRemoteConfigs: [[url: "$URL_SOURCE"]]
    ])
    gradlehome = tool 'G3'
}

def GradleBuild() {
    
    try {
        gradlehome = tool 'G3'
        sh "./gradlew chromeTest"
    } catch (Exception e) {
        sh "echo Build Failed at chromeTest"
    } 
}

def AntBuild() {
    try {
        anthome = tool 'ANT'
        sh "'${anthome}/bin/ant' main"
    } catch (Exception e) {
        sh "echo Ant Build Failed"
    }

}

def MsBuild() {

        try {
        sh "sleep 15"
    } catch (Exception e) {
        sh "echo MS Build Failed"
    }
}

def CreateArtifact() {
    try {
        sh "tar -czvf $ARTIFACT_NAME ${WORKSPACE} --exclude=${WORKSPACE}/readme.md --exclude=${WORKSPACE}/.*"
    } catch (Exception e) {
        sh "ls -lart ${WORKSPACE}"
    }
}

def UploadArtifact() {

    echo 'printing Build Stamp on Artifact'
    fingerprint "${WORKSPACE}/$ARTIFACT_NAME"
    def server = Artifactory.server "$SERVER_ID"
    def buildInfo = Artifactory.newBuildInfo()
    buildInfo.env.capture = true
    buildInfo.env.collect()

    def uploadSpec = """{
      "files": [
        {
          "pattern": "$ARTIFACTORY_PATTERN",
          "target": "$ARTIFACTORY_TARGET"
        }
      ]
    }"""
    // Upload to Artifactory.
    server.upload spec: uploadSpec, buildInfo: buildInfo

    buildInfo.retention maxBuilds: 10, maxDays: 7, deleteBuildArtifacts: true
    // Publish build info.
    server.publishBuildInfo buildInfo
    stash includes: "$ARTIFACT_NAME", name: 'STASH-1'
}

def Downloadartifact() { 
    def server = Artifactory.server "$SERVER_ID"
    def downloadSpec = """{
        "files": [
         {
            "pattern": "libs-snapshot-local/${JOB_NAME}/${BUILD_NUMBER}/$ARTIFACT_NAME",
            "target": "$WORKSPACE/"
           }
    ]
  }"""
  server.download(downloadSpec)
} 

def PublishHTMLReport() {
    publishHTML([allowMissing: false,
    alwaysLinkToLastBuild: false,
    keepAll: false,
    reportDir: "${WORKSPACE}/build/reports/chromeTest/tests",
    reportFiles: 'index.html', reportName: 'HTML Report'])
}

def Sleeping() {
    sleep 5
}
def CodeTest() {
    parallel(longerTests: {
        runTests(5)
        CreateArtifact()
        
    }, quickerTests: {
        runTests(5)
    })
}
def runTests(duration) {
    node {
        sh "sleep ${duration}"
        }
}
def DeployQA() {
    input message: "Ready 2 Deploy 2 QA"
    parallel(longerTests: {
        QA1()
        
    }, quickerTests: {
        QA2()
    })
}
def QA1() {

    sh "mkdir QA1"
    dir('QA1') {
        sh "ls -lart" 
        unstash 'STASH-1'
        pwd()
        sh "ls -lart" 
    }

}
def QA2() {

    sh "mkdir QA2"
    dir('QA2') {
        sh "ls -lart" 
        unstash 'STASH-1'
        pwd()
        sh "ls -lart" 
    }

}
