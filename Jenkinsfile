pipeline {
    agent any

    environment {
        // NETLIFY_SITE_ID = 'd9022715-e6c5-4569-938a-52d806a99677'
        // NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.${BUILD_ID}"
        APP_NAME = 'myjenkinsapp'
        AWS_DEFAULT_REGION = 'ap-south-1'
        AWS_ECS_CLUSTER = 'LearnJenkinsApp-Cluster-Prod'
        AWS_ECS_SERVICE = 'LearnJenkinsApp-Service-Prod'
        AWS_ECS_TASK_DEFINITION = 'LearnJenkinsApp-TaskDefinition-Prod'
    }

    stages {
        // stage("Docker") {
        //     steps {
        //         sh 'docker build -t my-playwright .'
        //     }
        // }
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
        stage("Build Docker image") {
            agent {
                docker {
                    // image 'amazon/aws-cli'
                    image 'my-aws-cli'
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                    reuseNode true
                }
            }
            steps {
                sh '''
                    # amazon-linux-extras install docker
                    docker build -t ${APP_NAME}:${REACT_APP_VERSION} .
                '''

                
            }
        }
        stage('Deploy to AWS') {
            agent {
                docker {
                    // image 'amazon/aws-cli'
                    image 'my-aws-cli'
                    args "--entrypoint=''"
                    reuseNode true
                }
            }
            // environment {
            //     AWS_S3_BUCKET = 'learn-jenkins-202502071003'
            // }
            steps {
                withCredentials([usernamePassword(credentialsId: 'karan-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                     sh '''
                        aws --version
                        # yum install jq -y
                        # echo "Hello S3!" > index.html
                        # aws s3 cp index.html s3://${AWS_S3_BUCKET}/index.html
                        # aws s3 sync build s3://${AWS_S3_BUCKET}
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json "file://aws/task-definition-prod.json" | jq '.taskDefinition.revision')
                        echo ${LATEST_TD_REVISION}
                        aws ecs update-service --cluster ${AWS_ECS_CLUSTER} --service ${AWS_ECS_SERVICE} --task-definition ${AWS_ECS_TASK_DEFINITION}:${LATEST_TD_REVISION}
                        aws ecs wait services-stable --cluster ${AWS_ECS_CLUSTER} --services ${AWS_ECS_SERVICE}
                    '''
                }
            }
        }

        
        // stage('Tests') {
        //     parallel {

        //         stage('Unit Tests') {
        //             agent {
        //                 docker {
        //                     image 'node:18-alpine'
        //                     reuseNode true
        //                 }
        //             }
        //             steps {
        //                 sh '''
        //                     echo "Test stage"
        //                     test -f build/index.html
        //                     npm test
        //                 '''
        //             }
        //             post {
        //                 always {
        //                     junit 'jest-results/junit.xml'
        //                 }
        //             }
        //         }

        //         stage('Local E2E') {
        //             agent {
        //                 docker {
        //                     // image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
        //                     image 'my-playwright'
        //                     reuseNode true
        //                 }
        //             }
        //             steps {
        //                 // E2E tests are being run on local deployment
        //                 sh '''
        //                     # npm install serve
        //                     # node_modules/.bin/serve -s build &
        //                     serve -s build &
        //                     sleep 10
        //                     npx playwright test --reporter=html
        //                 '''
        //             }
        //             post {
        //                 always {
        //                     publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Local E2E', reportTitles: '', useWrapperFileDirectly: true])
        //                 }
        //             }
        //          }
        //     }
        // }

        // stage('Deploy staging') {
        //     agent {
        //         docker {
        //             // image 'node:18-alpine'
        //             image 'my-playwright'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //             # npm install netlify-cli node-jq
        //             # node_modules/.bin/netlify --version
        //             netlify --version
        //             echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
        //             #node_modules/.bin/netlify status
        //             netlify status
        //             #node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
        //             netlify deploy --dir=build --json > deploy-output.json
        //         '''
        //         script {
        //             // env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
        //             // env.STAGING_URL = sh(script: "node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
        //             env.STAGING_URL = sh(script: "jq -r '.deploy_url' deploy-output.json", returnStdout: true)
        //         }
        //     }
        // }
        //  stage('Staging E2E') {
        //     agent {
        //         docker {
        //             // image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
        //             image 'my-playwright'
        //             reuseNode true
        //         }
        //     }
        //     environment {
        //         CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
        //     }
        //     steps {
        //         // E2E tests being run on prod deployment which is hoted by netlify
        //         sh '''
        //             npx playwright test --reporter=html
        //         '''
        //     }
        //     post {
        //         always {
        //             publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
        //         }
        //     }
        // }
        // stage('Approval') {
        //     steps {
        //         timeout(time: 15, unit: 'MINUTES') {
        //             input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
        //         }
        //     }
        // }
        // stage('Deploy Prod') {
        //     agent {
        //         docker {
        //             // image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
        //             image 'my-playwright'
        //             reuseNode true
        //         }
        //     }
        //     environment {
        //         CI_ENVIRONMENT_URL = 'https://beamish-genie-a9d6be.netlify.app'
        //     }
        //     steps {
        //         // E2E tests being run on prod deployment which is hoted by netlify
        //         sh '''
        //             node --version
        //             # npm install netlify-cli
        //             # node_modules/.bin/netlify --version
        //             netlify --version
        //             echo "Deploying to prod. Site ID: $NETLIFY_SITE_ID"
        //             # node_modules/.bin/netlify status
        //             # node_modules/.bin/netlify deploy --dir=build --prod
        //             netlify status
        //             netlify deploy --dir=build --prod
        //             sleep 10
        //             npx playwright test --reporter=html
        //         '''
        //     }
        //     post {
        //         always {
        //             publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
        //         }
        //     }
        // }

    }
}
