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
        sh 'cd apollo-service'
        sh 'echo "configdb:\n" > ./apollo-service.values.yaml'
        sh "echo \"  host: $MYSQL_HOST\n\" >> ./apollo-service.values.yaml"
        sh "echo \"  dbName: ApolloConfigDB\n\" >> ./apollo-service.values.yaml"
        sh "echo \"  userName: root\n\" >> ./apollo-service.values.yaml"
        sh "echo \"  password: $MYSQL_ROOT_PASSWORD\n\" >> ./apollo-service.values.yaml"
        sh "echo \"  connectionStringProperties: characterEncoding=utf8&useSSL=false\n\" >> ./apollo-service.values.yaml"
        sh "echo \"  service:\n\">> ./apollo-service.values.yaml"
        sh "echo \"    enabled: false\">> ./apollo-service.values.yaml"
        sh "echo \"configetc:\n\" >> ./apollo-service.values.yaml"
        sh "echo \"  consul_host: $CONSUL_HOST\n\" >> ./apollo-service.values.yaml"
        sh "echo \"  consul_port: $CONSUL_PORT\n\" >> ./apollo-service.values.yaml"
        sh "echo \"configService:\n\" >> ./apollo-service.values.yaml"
        sh "echo \"  replicaCount: $CFS_COPIES\n\" >> ./apollo-service.values.yaml"
        sh "#echo \"  config:\n\" >> ./apollo-service.values.yaml"
        sh "#echo \"    configServiceUrlOverride: \"http://$LAN_IP:8080\"\n\" >> ./apollo-service.values.yaml"
        sh "#echo \"    adminServiceUrlOverride: \"http://$LAN_IP:8090\"\n\" >> ./apollo-service.values.yaml"
        sh "echo \"adminService:\n\" >> ./apollo-service.values.yaml"
        sh "echo \"  replicaCount: $ADM_COPIES\n\" >> ./apollo-service.values.yaml"
        sh "echo \"\n\" >> ./apollo-service.values.yaml"
        sh "helm install $SVC_NAME_PREFIX -f ./apollo-service.values.yaml -n $K8S_NAMESPACE ."
      }
    }

    stage('Install apollo-portal') {
      steps {
        sh 'helm repo add apollo https://www.apolloconfig.com/charts'
        sh "helm install $SVC_NAME_PREFIX \
    --set configdb.host=$MYSQL_HOST \
    --set configdb.userName=root \
    --set configdb.password=$MYSQL_ROOT_PASSWORD \
    --set configdb.service.enabled=false \
    --set config.envs=\"dev,pro\" \
    --set config.metaServers.dev=http://$SVC_NAME_PREFIX-apollo-configservice:8080 \
    --set config.metaServers.pro=http://$SVC_NAME_PREFIX-apollo-configservice:8080 \
    --set replicaCount=1 \
    -n $K8S_NAMESPACE \
    apollo/apollo-portal"
      }
    }

  }
}
