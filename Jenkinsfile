podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      serviceAccountName: jenkins-admin
      containers:
      - name: maven
        image: maven:3.8.1-jdk-8
        command:
        - sleep
        args:
        - 120
      - name: kaniko
        image: gcr.io/kaniko-project/executor:debug
        command:
        - sleep
        args:
        - 60
        volumeMounts:
        - name: kaniko-secret
          mountPath: /kaniko/.docker
      restartPolicy: Never
      volumes:
      - name: kaniko-secret
        secret:
            secretName: dockercred
            items:
            - key: .dockerconfigjson
              path: config.json
''') {
  node(POD_LABEL) {
    // stage('Get a  project') {      
    //   git url: 'https://github.com/anmiroshnichenko/test-app.git', branch: 'main'      
    // }
    stage('Build test-app Image') {
      container('kaniko') {
        stage('Build a my project') {
          sh '''
          pwd
          /kaniko/executor --context `pwd` --destination aleksandm/test-app:$TAG_NAME
          '''
        }
      }
    }
    stage('deploy to dev') { 
      // echo "${env.TAG_NAME}"  
      if ("${env.TAG_NAME}" != 'null')
      container ('maven') {
        stage('deploy test-app') {           
          sh '''
          curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
          chmod 700 get_helm.sh
          ./get_helm.sh                    
          helm upgrade --install test-app  ./test-app-chart -n devops-tools   --set frontend.image.tag=$TAG_NAME      
          '''
        }
      }
    }    
  }
}
