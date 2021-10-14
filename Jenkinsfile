pipeline {
  agent any
  tools {
    go 'go'
  }
  environment {
    GOPROXY = 'https://goproxy.cn,direct'
  }
  stages {

    stage('Check deps tools') {
      steps {
        script {
          if (!fileExists("/usr/bin/helm")) {
            sh 'mkdir -p $HOME/.helm'
            if (!fileExists("$HOME/.helm/.helm-src")) {
              sh 'git clone https://github.com/helm/helm.git $HOME/.helm/.helm-src'
            }
            sh 'cd $HOME/.helm/.helm-src; make; cp bin/helm /usr/bin/helm'
            sh 'helm version'
          }
        }
      }
    }

    stage('Helm install apollo-service') {
      steps {
        sh 'echo "configdb:\n" > ./apollo-service.values.yaml'
        sh "echo \"  host: $MYSQL_HOST\n\" >> ./apollo-service.values.yaml"
        sh "echo \"  dbName: ApolloConfigDB\n\" >> ./apollo-service.values.yaml"
        sh "echo \"  userName: root\n\" >> ./apollo-service.values.yaml"
        sh "echo \"  connectionStringProperties: characterEncoding=utf8&useSSL=false\n\" >> ./apollo-service.values.yaml"
        sh "echo \"  service:\n\">> ./apollo-service.values.yaml"
        sh "echo \"    enabled: false\">> ./apollo-service.values.yaml"
        sh "echo \"configetc:\n\" >> ./apollo-service.values.yaml"
        sh "echo \"  consul_host: $CONSUL_HOST\n\" >> ./apollo-service.values.yaml"
        sh "echo \"  consul_port: $CONSUL_PORT\n\" >> ./apollo-service.values.yaml"
        sh "echo \"configService:\n\" >> ./apollo-service.values.yaml"
        sh "echo \"  replicaCount: $CFS_COPIES\n\" >> ./apollo-service.values.yaml"
        sh "echo \"adminService:\n\" >> ./apollo-service.values.yaml"
        sh "echo \"  replicaCount: $ADM_COPIES\n\" >> ./apollo-service.values.yaml"
        sh "echo \"\n\" >> ./apollo-service.values.yaml"
        sh "helm uninstall $SVC_NAME -n $K8S_NAMESPACE"
        sh "helm install $SVC_NAME -f ./apollo-service.values.yaml -n $K8S_NAMESPACE ./apollo-service"
      }
    }

    stage('Install apollo-portal') {
      steps {
        sh "helm uninstall $SVC_NAME -n $K8S_NAMESPACE"
        sh "helm install $SVC_NAME \
    --set configdb.host=$MYSQL_HOST \
    --set configdb.userName=root \
    --set configdb.service.enabled=false \
    --set config.envs=\"dev\\,pro\" \
    --set config.metaServers.dev=http://$SVC_NAME-apollo-configservice:8080 \
    --set config.metaServers.pro=http://$SVC_NAME-apollo-configservice:8080 \
    --set replicaCount=1 \
    -n $K8S_NAMESPACE \
    ./apollo-portal"
      }
    }

  }
}
