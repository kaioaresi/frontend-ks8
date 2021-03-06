def LABEL_ID = "questcode-${UUID.randomUUID().toString()}"

podTemplate(
    containers: [
      containerTemplate(alwaysPullImage: false, args: 'cat', command: '/bin/sh -c', envVars: [], image: 'docker', livenessProbe: containerLivenessProbe(execArgs: '', failureThreshold: 0, initialDelaySeconds: 0, periodSeconds: 0, successThreshold: 0, timeoutSeconds: 0), name: 'docker-container', ports: [], privileged: false, resourceLimitCpu: '', resourceLimitMemory: '', resourceRequestCpu: '', resourceRequestMemory: '', shell: null, ttyEnabled: true, workingDir: '/home/jenkins'),
      containerTemplate(alwaysPullImage: false, args: 'cat', command: '/bin/sh -c', envVars: [], image: 'lachlanevenson/k8s-helm:v2.13.1', name: 'helm-container', ttyEnabled: true)

      //Helm cliente - lachlanevenson/k8s-helm
    ],
    label: LABEL_ID,
    name: 'devops-pipeline',
    namespace: 'devops',
    volumes: [hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')]
)
{
  def REPOS
  def TAG_IMG = "staging"
  def KUBE_NAMESPACE
  def ENVIRONMENT
  def GIT_BRANCH
  def HELM_NAME_DEPLOY
  def HELM_CHART_NAME = "questcode/frontend"
  def NODE_PORT = "30080"
  // Start Pipeline
  node(LABEL_ID){
    stage('Checkout') {
      echo 'Iniciando clone do repositorio'
      REPOS = checkout scm
      GIT_BRANCH = REPOS.GIT_BRANCH
      echo "GIT_BRANCH e '${GIT_BRANCH}'"

      // baseado na branch alterar o deploy por ambiente
      if(GIT_BRANCH.equals("master")){
          KUBE_NAMESPACE = "prod"
          ENVIRONMENT = "production"
          TAG_IMG = "latest"
          echo "Master - GIT_BRANCH - '${GIT_BRANCH}'"
      } else if(GIT_BRANCH.equals("staging")){
          KUBE_NAMESPACE = "staging"
          ENVIRONMENT = "staging"
          NODE_PORT = "30180"
          echo "Staging - GIT_BRANCH - '${GIT_BRANCH}'"
      } else {
          def error = echo "Nao existem pipeline para essa branch ${GIT_BRANCH}!"
          throw new Exception(error)
      }
      // Nome que sera definico no helm
      HELM_NAME_DEPLOY = KUBE_NAMESPACE + "-frontend"

    }
    stage('Package') {
        // Comandos que devem ser executados dentro do container 'docker-container'
        container('docker-container') {
          echo 'Iniciando Build'
          withCredentials([usernamePassword(credentialsId: '95b860bc-932a-4770-8674-8cf9b3b147cb', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USER')]) {
            sh 'docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASSWORD}'
            sh "docker build -t ${DOCKER_HUB_USER}/frontend:${TAG_IMG} . --build-arg NPM_ENV=${ENVIRONMENT}"
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
            sh "helm upgrade ${HELM_NAME_DEPLOY} ${HELM_CHART_NAME} --set image.tag=${TAG_IMG} --set service.nodePort=${NODE_PORT}"
          } catch(Exception e){
            sh "helm install --namespace=${KUBE_NAMESPACE} --name ${HELM_NAME_DEPLOY} ${HELM_CHART_NAME} --set image.tag=${TAG_IMG} --set service.nodePort=${NODE_PORT}"
          }
        } // Helm container fim
    }
  }// Node fim
}//podTemplate FIM
