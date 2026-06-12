pipeline {
    agent any

    tools {
        jdk 'JDK21'
        maven 'M3'
    }

    parameters {
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: '테스트 실행 여부')
        choice(name: 'DEPLOY_ENV', choices: ['staging', 'production'], description: '배포 환경')
    }

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 20, unit: 'MINUTES')
    }

    environment {
        APP_NAME = 'cicd-demo'
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Build') {
            steps { sh 'mvn -B clean compile' }
        }
        stage('Test') {
            when { expression { return params.RUN_TESTS } }
            steps { sh 'mvn -B test' }
            post {
                always { junit 'target/surefire-reports/*.xml' }
            }
        }
        stage('Package') {
            steps {
                sh 'mvn -B -DskipTests package'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
        stage('Quality Gate') {
            parallel {
                stage('Static Analysis') {
                    steps { sh 'echo "정적 분석 단계 (예: checkstyle)"' }
                }
                stage('Dependency Check') {
                    steps { sh 'echo "의존성 취약점 점검 단계"' }
                }
            }
        }
        stage('Approval') {
            when { expression { params.DEPLOY_ENV == 'production' } }
            steps {
                input message: '프로덕션 배포를 승인하시겠습니까?', ok: '배포 진행'
            }
        }
        stage('Deploy (모의)') {
            steps {
                echo "${params.DEPLOY_ENV} 환경으로 ${env.APP_NAME} 배포 시뮬레이션"
            }
        }
    }

    post {
        success { echo "OK: ${env.APP_NAME} 파이프라인 성공 (빌드 #${env.BUILD_NUMBER})" }
        failure { echo "FAIL: 파이프라인 실패 - 콘솔 로그를 확인하세요" }
        always  { echo "결과: ${currentBuild.currentResult}" }
    }
}
