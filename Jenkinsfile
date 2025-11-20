pipeline {
    agent any

    stages {
        /*
        line 1
        line 2 
        */
        stage('Build') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                ls -la
                node --version
                npm --version
                npm ci
                npm run build
                ls -la
                '''
        }
    }
    stage ('Test'){
           agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
        steps{
        echo "Test Stage"
        sh '''
        set -e 
        echo "Running Test..."
        npm test
        echo "Test completed Successfully"
        '''
        }
    }
}
post {
    always{
        junit 'test-results/junit.xml'
    }
}
}
