pipeline {
  agent any

  environment {
    NETLIFY_SITE_ID   = '26cea4a9-c8b7-45e9-ac86-cea792c200e6'
    NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    CI_ENVIRONMENT_URL = 'https://verdant-creponne-a7f9cc.netlify.app/'
  }

  stages {
    /*
    line 1
    line 2*/
    stage('Build') {
      agent {
        docker {
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

    stage('Tests') {
      parallel {

        stage('Unit Test') {
          agent {
            docker {
              image 'node:18-alpine'
              reuseNode true
            }
          }
          steps {
            echo "Test Stage"
            sh '''
            set -e
            echo "Running Test..."
            echo "small changes"
            npm test
            echo "Test completed Successfully"
            '''
          }
          post {
            always {
              junit 'jest-results/junit.xml'
            }
          }
        }

        stage('E2E') {
          agent {
            docker {
              image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
              reuseNode true
            }
          }
          steps {
            sh '''
            set -e
            npm install serve
            node_modules/.bin/serve -s build &
            sleep 10
            npx playwright test --reporter=html
            '''
          }
          post {
            always {
              publishHTML([
                allowMissing: false,
                alwaysLinkToLastBuild: false,
                keepAll: false,
                reportDir: 'playwright-report',
                reportFiles: 'index.html',
                reportName: 'Playwright local',
                reportTitles: ''
              ])
            }
          }
        }

      }
    }

    stage('Deploy') {
      agent {
        docker {
          image 'node:18-alpine'
          reuseNode true
        }
      }
      steps {
        sh '''
        node --version
        npm --version

        # Install Netlify CLI locally
        npm install netlify-cli

        echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"

        # Deploy pre-built assets, skip Netlify build & plugins (no Lighthouse/Puppeteer)
        npx netlify deploy \
          --dir=build \
          --prod \
          --site="$NETLIFY_SITE_ID" \
          --no-build
        '''
      }
    }

  stage('Prod E2E') {
        agent {
          docker {
            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
            reuseNode true
          }
        }

        environment {
        NETLIFY_SITE_ID   = '26cea4a9-c8b7-45e9-ac86-cea792c200e6'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        CI_ENVIRONMENT_URL = 'https://verdant-creponne-a7f9cc.netlify.app'
        }
        steps {
          sh '''
          npx playwright test --reporter=html
          '''
        }
        post {
          always {
            publishHTML([
              allowMissing: false,
              alwaysLinkToLastBuild: false,
              keepAll: false,
              reportDir: 'playwright-report',
              reportFiles: 'index.html',
              reportName: 'Playwright E2E ',
              reportTitles: ''
            ])
          }
        }
      }
  }
}
