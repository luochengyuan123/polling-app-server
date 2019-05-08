// 测试环境Jenkinsfile
def envKey = env.JOB_NAME.substring(0, 1)


def projectName, targetDir, mvnArgs = ""
if (params.subProject != null) {
    projectName = params.subProject.trim()
    targetDir = "${projectName}/target"
    mvnArgs = "-pl ${projectName} -am -amd"
} else {
    projectName = env.JOB_NAME.substring(2, env.JOB_NAME.length())
    targetDir = "target"
}

def gitUrl = "https://github.com/luochengyuan123/${env.JOB_NAME.substring(2, env.JOB_NAME.length())}.git"
def gitCredential = "gitlabjenkins"
def gitBranch = params.BRANCH.trim()
def projectName = env.JOB_NAME.substring(2, env.JOB_NAME.length())
def imageTag = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
def registryUrl = "harbor.haimaxy.com"
def imageEndpoint = "payeco/${projectName}"
def image = "${registryUrl}/${imageEndpoint}"


def registryCredential = "harbor"


node('jenkins-jnlp') {
    echo "git clone gitlab"
    def mvnHome
    stage('Preparation') {
        git branch: gitBranch, credentialsId: gitCredential, url: gitUrl
        mvnHome = tool 'M3'
    }

    stage('Maven build') {
	    echo "mvn jar"
        sh "'${mvnHome}/bin/mvn' clean package -Dmaven.test.skip=true"
        archive 'target/*.jar'

    }


    stage('Test') {
      echo "2.Test Stage"
      echo "${projectName}"
      echo "${gitBranch}"
    }
    stage('Build & Push Image') {
        echo "4.Push Docker Image Stage"
        dir("/home/jenkins/workspace/${jobName}") {
           docker.withRegistry("https://${registryUrl}", "${registryCredential}") {
                def image = docker.build("${registryUrl}/payeco/${image}", ".")
                image.push()
            }
        }
    }

    stage('Deploy to k8s') {
        echo "5. Deploy Stage"
        if (env.BRANCH_NAME == 'master') {
            input "确认要部署线上环境吗？"
        }
        sh "sed -i 's/<IMAGE>/${image}/' k8s.yaml"
        sh "sed -i 's/<IMAGE_TAG>/${imageTag}/' k8s.yaml"
        sh "kubectl apply -f k8s.yaml --record"
    }

}
