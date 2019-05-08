// 测试环境Jenkinsfile
def envKey = env.JOB_NAME.substring(0,1)
def domainName, namespace, portPrefix, springConfigLabel, serviceAddr
switch (envKey) {
    case "d":
       domainName = "d.haimaxy.com"
       namespace = "oliver-dev"
       break
    default:
        println("No matching case found!!")

}
def registryUrl = "harbor.haimaxy.com"
def registryCredential = "harbor"
def projectName = env.JOB_NAME.substring(2, env.JOB_NAME.length())
def jobName = env.JOB_NAME.trim()
def gitBranch = params.BRANCH.trim()

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
    stage('Prepare') {
        echo "1.Prepare Stage"
        checkout scm
        script {
            if (${gitBranch} != 'master') {
                imageTag = "${gitBranch}-${imageTag}"
            }
			sh "'${mvnHome}/bin/mvn' clean package -Dmaven.test.skip=true"
            archive 'target/*.jar'
        }
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
                def images = docker.build("${image}:${imageTag}", ".")
                images.push()
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
