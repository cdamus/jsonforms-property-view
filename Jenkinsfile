def kubernetes_config = """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: node
    image: eclipsetheia/theia-blueprint
    tty: true
    resources:
      limits:
        memory: "2Gi"
        cpu: "1"
      requests:
        memory: "2Gi"
        cpu: "1"
    command:
    - cat
    volumeMounts:
    - mountPath: "/home/jenkins"
      name: "jenkins-home"
      readOnly: false
    - mountPath: "/.yarn"
      name: "yarn-global"
      readOnly: false
    - name: global-cache
      mountPath: /.cache     
    - name: global-npm
      mountPath: /.npm      
  volumes:
  - name: "jenkins-home"
    emptyDir: {}
  - name: "yarn-global"
    emptyDir: {}
  - name: global-cache
    emptyDir: {}
  - name: global-npm
    emptyDir: {}
"""

pipeline {
    agent {
        kubernetes {
            label 'emfcloud-agent-pod'
            yaml kubernetes_config
        }
    }
    
    options {
        buildDiscarder logRotator(numToKeepStr: '15')
    }
    
    environment {
        YARN_CACHE_FOLDER = "${env.WORKSPACE}/yarn-cache"
        SPAWN_WRAP_SHIM_ROOT = "${env.WORKSPACE}"
    }

    stages {
        stage('Build') {
            steps {
                container('node') {
                    buildInstaller()
                }
            }
        }

        stage('Deploy (master only)') {
            when { branch 'master' }
            steps {
                build job: 'deploy-jsonforms-property-view-npm', wait: false
            }
        }
    }
}

def buildInstaller() {
    int MAX_RETRY = 3

    checkout scm
    sh "printenv && yarn cache dir"
    sh "yarn cache clean"
    try {
        sh(script: 'yarn --frozen-lockfile --force')
    } catch(error) {
        retry(MAX_RETRY) {
            echo "yarn failed - Retrying"
            sh(script: 'yarn --frozen-lockfile --force')        
        }
    }
}
