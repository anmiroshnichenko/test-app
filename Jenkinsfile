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
        - 60
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
    stage('Get a Maven project') {
      git url: 'https://github.com/anmiroshnichenko/test-app.git', branch: 'main'
      container('maven') {
        stage('Build a Maven project') {
          sh '''
          echo pwd
          '''
        }
      }
    }

    stage('Build test-app Image') {
      container('kaniko') {
        stage('Build a my project') {
          sh '''
            /kaniko/executor --context `pwd` --destination aleksandm/test-app:$TAG
          '''
        }
      }
    }
    stage('deploy to dev') {
      when { tag "v1.0.0" }  
      container ('maven') {
        stage('deploy test-app') {
          sh '''
          curl -LO https://dl.k8s.io/release/v1.26.11/bin/linux/amd64/kubectl
          chmod +x ./kubectl 
          mv ./kubectl /usr/local/bin/kubectl
          kubectl apply -f deployment.yaml -n devops-tools
          kubectl rollout restart deployment test-app -n devops-tools
          '''
        }
      }
    }    
  }
}
