@Library('jenkins_libs') _

def DOCKER_REGISTRY_USER_CREDENTIONALS_ID  = 'gitlab.sb' //getParameterValue(buildEnv, 'DOCKER_REGISTRY_USER_CREDENTIONALS_ID') 
def DOCKER_REGISTRY_URL = 'https://registry.silverbulleters.org' // getParameterValue(buildEnv, 'DOCKER_REGISTRY_URL')
                            

pipeline {

    agent { label 'docker' }

    stages {

        stage('Тестирование') {
            parallel {
                stage('Статический анализ') {
                    steps {
                        script {

                            docker.withRegistry(DOCKER_REGISTRY_URL, DOCKER_REGISTRY_USER_CREDENTIONALS_ID) {
                                def sonarCommand = sonarqubeScan()
                                cmdRun("echo sonarCommand:  ${sonarCommand}")
                                withDockerContainer(args: '', image: 'registry.silverbulleters.org/landscape/ops/isasacode/silverbulleters/sonarqube-scanner:latest') {
                                    cmdRun(sonarCommand)
                                }
                            }
                        }
                    }

                }
                stage('ADD Test') {
                    steps {
                        script{
                            docker.withRegistry(DOCKER_REGISTRY_URL, DOCKER_REGISTRY_USER_CREDENTIONALS_ID) {
                                withDockerContainer(args: '-u root:root', image: 'registry.silverbulleters.org/landscape/ops/isasacode/vanessa-runner:8.3.15.1489-latest') {
                                    cmdRun("set -xe && xstart && novnc && runxfce4 && echo >> /opt/1C/v8.3/x86_64/conf/nethasp.ini && echo NH_SERVER_ADDR = 10.77.1.141 >> /opt/1C/v8.3/x86_64/conf/nethasp.ini && bash")
                                    cmdRun("rm -rf /usr/share/oscript/lib/opm/oscript_modules/fs")
                                    cmdRun("opm run init file")
                                    cmdRun("opm build .")
                                    archiveArtifacts 'add-*.zip'
                                    archiveArtifacts 'add-*.ospx'
                                    cmdRun("vrunner xunit tests/smoke --settings tools/JSON/vrunner.json --reportsxunit \"ГенераторОтчетаJUnitXML{build/junit-smoke/junit.xml};ГенераторОтчетаAllureXMLВерсия2{build/allure/allure.xml}\"")
                                }
                            }
                        }
                    }
                }
            }
        }
    }

}

def sonarqubeScan() {

    def projectVersion = ""
    
    def configurationText = readFile encoding: 'UTF-8', file: 'epf/bddRunner/bddRunner/Ext/ObjectModule.bsl'
    def configurationVersion = (configurationText =~ /Версия = "(.*)";/)[0][1]
    projectVersion = "-Dsonar.projectVersion=${configurationVersion}"

    
    def sonarCommand = "sonar-scanner -X ${projectVersion}"

    withCredentials([string(credentialsId: env.OpenSonarOAuthCredentianalID, variable: 'SonarOAuth')]) {
        sonarCommand = sonarCommand + " -Dsonar.host.url=${env.sonarQubeURL} -Dsonar.login=${SonarOAuth}"
    }

    def makeAnalyzis = true
    // if (env.BRANCH_NAME == "master") {
    //     echo 'Analysing master branch'
    // } else if (env.BRANCH_NAME == "develop") {
    //     echo 'Analysing develop branch'
    //     // sonarCommand = sonarCommand + " -Dsonar.branch.name=${BRANCH_NAME}"
    // } else if (env.BRANCH_NAME.startsWith("release/") || env.BRANCH_NAME.startsWith("feature/")) {
    //     // sonarCommand = sonarCommand + " -Dsonar.branch.name=${BRANCH_NAME}"
    // } else if (env.BRANCH_NAME.startsWith("PR-")) {
    //     // Report PR issues
    //     def PRNumber = env.BRANCH_NAME.tokenize("PR-")[0]
    //     def gitURLcommand = 'git config --local remote.origin.url'
    //     def gitURL = ""
    //     if (isUnix) {
    //         gitURL = sh(returnStdout: true, script: gitURLcommand).trim()
    //     } else {
    //         gitURL = bat(returnStdout: true, script: gitURLcommand).trim()
    //     }
    //     def repository = gitURL.tokenize("/")[2] + "/" + gitURL.tokenize("/")[3]
    //     repository = repository.tokenize(".")[0]
    //     // withCredentials([string(credentialsId: env.GithubOAuthCredentianalID, variable: 'githubOAuth')]) {
    //     //     sonarCommand = sonarCommand + " -Dsonar.analysis.mode=issues -Dsonar.github.pullRequest=${PRNumber} -Dsonar.github.repository=${repository} -Dsonar.github.oauth=${githubOAuth}"
    //     // }
    // } else {
    //     echo "Анализ SonarQube не выполнен. Ветка ${env.BRANCH_NAME} не подходит по условию проверки веток!"
    //     makeAnalyzis = false
    // }

    return sonarCommand
                
}