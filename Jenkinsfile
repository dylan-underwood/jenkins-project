node('TESTLINUX') {

    stage('Build') {
        deleteDir()

        checkout changelog: false, poll: false, scm: [$class: 'GitSCM', branches: [[name: '*/${gitTargetBranch}']], browser: [$class: 'GitLab', repoUrl: '${GitLab_URL}', version: '8.9'], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'who'],[$class: 'CloneOption', depth: 0, honorRefspec: true, noTags: true, reference: '', shallow: true, timeout: 20], [$class: 'LocalBranch', localBranch: '${gitTargetBranch}']], submoduleCfg: [], userRemoteConfigs: [[url: '${REPO_URL}']]]

        env.JAVA_HOME="${tool 'Java8'}"
        env.PATH="${env.JAVA_HOME}/bin:${env.PATH}"
        
        withEnv(["GRADLE_USER_HOME=${env.WORKSPACE}"]){  
            
            echo "GRADLE_USER_HOME:$GRADLE_USER_HOME"
            
            def server = Artifactory.newServer url: SERVER_URL, credentialsId: CREDENTIALS
            def rtGradle = Artifactory.newGradleBuild()

            rtGradle.tool = GRADLE_TOOL
            rtGradle.deployer repo: 'rncs-sandbox-local', server: server
            rtGradle.resolver repo: 'remote-repos', server: server

            withEnv(['DONT_COLLECT=FOO']){
                def buildInfo = Artifactory.newBuildInfo()
                buildInfo.env.capture = true
                buildInfo.env.filter.addInclude("*")
                buildInfo.env.filter.addExclude("DONT_COLLECT*")

                rtGradle.usesPlugin = true

                rtGradle.run rootDir: "who/", buildFile: 'build.gradle', tasks: 'runCI', buildInfo: buildInfo

                server.publishBuildInfo buildInfo
            }
        }            
    } 
  
}

