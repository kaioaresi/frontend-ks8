podTemplate(
    containers: [
      containerTemplate(alwaysPullImage: false, args: 'cat', command: '/bin/sh -c', envVars: [], image: 'docker', livenessProbe: containerLivenessProbe(execArgs: '', failureThreshold: 0, initialDelaySeconds: 0, periodSeconds: 0, successThreshold: 0, timeoutSeconds: 0), name: 'docker-container', ports: [], privileged: false, resourceLimitCpu: '', resourceLimitMemory: '', resourceRequestCpu: '', resourceRequestMemory: '', shell: null, ttyEnabled: true, workingDir: '/home/jenkins'),
      containerTemplate(alwaysPullImage: false, args: 'cat', command: '/bin/sh -c', envVars: [], image: 'lachlanevenson/k8s-helm:v2.13.1', name: 'helm-container', ttyEnabled: true)

      //Helm cliente - lachlanevenson/k8s-helm
    ],
    label: 'questcode',
    name: 'devops-pipeline',
    namespace: 'devops',
    volumes: [hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')]
)
{
  // Pipeline
  node('questcode'){
    def REPOS
    def TAG_IMG = "staging"
    def KUBE_NAMESPACE
    def GIT_BRANCH
    stage('Checkout') {
      echo 'Iniciando clone do repositorio'
      // REPOS = git credentialsId: '48488f72-08cd-40ff-b88e-702dfc31276c', url: 'https://github.com/kaioaresi/frontend-ks8.git'
      REPOS = checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '48488f72-08cd-40ff-b88e-702dfc31276c', url: 'https://github.com/kaioaresi/frontend-ks8.git']]])
      GIT_BRANCH = REPOS.GIT_BRANCH
      // baseado na branch alterar o deploy por ambiente
      if(GIT_BRANCH.equals("master")){
        KUBE_NAMESPACE = "prod"
      } else if(GIT_BRANCH.equals("staging")){
        KUBE_NAMESPACE = "staging"
      } else {
        echo "Nao existem pipeline para essa branch ${GIT_BRANCH}!"
        exit 0
      }

    }
    stage('Package') {
        // Comandos que devem ser executados dentro do container 'docker-container'
        container('docker-container') {
          echo 'Iniciando Build'
          withCredentials([usernamePassword(credentialsId: '95b860bc-932a-4770-8674-8cf9b3b147cb', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USER')]) {
            sh 'docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASSWORD}'
            sh "docker build -t ${DOCKER_HUB_USER}/frontend:${TAG_IMG} . --build-arg NPM_ENV='${TAG_IMG}'"
            sh "docker image push ${DOCKER_HUB_USER}/frontend:${TAG_IMG}"
          }// withCredentials FIM
        }
    }
    stage('Deploy') {
        container('helm-container'){
          echo 'Iniciando deploy com helm'
          sh 'ls -lh'
          sh 'helm init --client-only'
          sh 'helm repo add questcode http://helm-chartmuseum:8080'
          sh 'helm repo list'
          sh 'helm repo update'
          sh 'helm search questcode'
          try {
            sh "helm upgrade --namespace=${KUBE_NAMESPACE} --name frontend questcode/frontend --set image.tag=${TAG_IMG}"
          } catch(Exception e){
            sh "helm install --namespace=${KUBE_NAMESPACE} --name frontend questcode/frontend --set image.tag=${TAG_IMG}"
          }
        } // Helm container fim
    }
  }// Node fim
}//podTemplate FIM
