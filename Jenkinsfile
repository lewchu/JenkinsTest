pipeline {
    agent any
    
    // 指定 Jenkins 全局工具（需提前配置 Maven 和 JDK）
    tools {
        maven 'maven-3'      // 对应 Jenkins 中配置的 Maven 工具名称
        jdk   'jdk-17'        // 对应 Jenkins 中配置的 JDK 工具名称
    }
    
    environment {
        // 项目克隆目录名（与 git clone 目标一致）
        PROJECT_DIR = 'JenkinsTest'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checkout code from GitHub...'
                // 清理旧目录，确保每次都是最新代码
                sh 'rm -rf ${PROJECT_DIR}'
                // 克隆代码（使用 HTTPS 避免 SSH 密钥配置）
                sh 'git clone git@github.com:lewchu/JenkinsTest.git ${PROJECT_DIR}'
            }
        }
        stage('Build') {
            steps {
                dir(env.PROJECT_DIR) {
                    echo 'Compiling the project...'
                    // 清理并编译
                    sh 'mvn -f demo/pom.xml clean compile'
                }
            }
        }
        stage('Test') {
            steps {
                dir(env.PROJECT_DIR) {
                    echo 'Running unit tests...'
                    sh 'mvn -f demo/pom.xml test'
                }
            }
        }
        stage('Package') {
            steps {
                dir(env.PROJECT_DIR) {
                    echo 'Packaging the project...'
                    sh 'mvn -f demo/pom.xml package'
                }
            }
        }
        stage('Cleanup') {
            steps {
                dir(env.PROJECT_DIR) {
                    sh '''
                        echo "Cleaning up port 8081..."
                        lsof -ti:8081 2>/dev/null | xargs -r kill -9
                        echo "Done"
                    '''
                }
            }
        }
        stage('Run') {
            steps {
                dir(env.PROJECT_DIR) {
                    echo 'Running the application...'
                    // 使用 withEnv 语句正确设置环境变量
                    withEnv(['JENKINS_NODE_COOKIE=dontkillme']) {
                        sh 'nohup java -jar demo/target/demo-0.0.1-SNAPSHOT.jar &'
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline finished.'
            // 可在此添加清理工作空间的命令（如删除临时文件）
        }
        success {
            echo 'Pipeline succeeded!'
            // 归档构建产物（JAR 包）
        }
        failure {
            echo 'Pipeline failed!'
            // 可添加通知（如邮件）
        }
    }
}
