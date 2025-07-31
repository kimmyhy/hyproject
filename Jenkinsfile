pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'mcr.microsoft.com/playwright:v1.53.2-jammy'
        GIT_CREDENTIALS_ID = '72abb696-26ae-4bc6-8886-1ce6709a0b62'
        GIT_REPO_URL = 'https://github.com/kimmyhy/hyproject.git'
        GIT_BRANCH = 'main'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                deleteDir()
                echo "env : ${params.env}"
            }
        }

        stage('Checkout') {
            steps {
                git branch: "${env.GIT_BRANCH}",
                    url: "${env.GIT_REPO_URL}",
                    credentialsId: "${env.GIT_CREDENTIALS_ID}"
            }
        }

        stage('Run tests in Docker') {
            steps {
                script {
                    docker.image(DOCKER_IMAGE).inside('-u root') {
                        echo "${params.workers}"
                        sh 'npm ci'
                        sh 'npx playwright install'
                        sh "npx playwright test --workers=${params.workers}"
                    }
                }
            }
        }
    }

    post {
        always {
            // HTML 리포트 파일들을 아티팩트로 저장
            archiveArtifacts artifacts: '**/playwright-report/**, **/monocart-report/**', 
                           fingerprint: true, // 파일 변경 사항 확인
                           allowEmptyArchive: true // 파일이 없을때 에러 방지

            // HTML Publisher로 리포트 게시
            publishHTML([
                allowMissing: false,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'playwright-report',
                reportFiles: 'index.html',
                reportName: 'Playwright HTML Report',
                reportTitles: ''
            ])
            
            publishHTML([
                allowMissing: false,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'monocart-report',
                reportFiles: 'index.html',
                reportName: 'Monocart Test Report',
                reportTitles: ''
            ])
            
            echo 'Cleaning up...'
            // HTML 리포트 게시 후에 cleanup
            cleanWs() // 컨테이너에서 만든 파일들은 삭제
        }
        failure {
            echo 'Build failed!'
        }
        success {
            echo 'Build succeeded!'
        }
    }
}