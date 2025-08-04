pipeline {
    agent any
    tools {
        maven 'ceshi1'
        jdk 'JDK'
    }
    stages {
        stage('拉取代码') {
            steps {
                echo "Pulling code from GitHub main branch..."
                git url: 'https://github.com/msg-555/mvc-.git', branch: 'main'
            }
        }
        
        stage('构建项目') {
            steps {
                echo "Building WAR package with Maven..."
                bat 'mvn clean package -Dmaven.test.skip=true'
                // 检查 WAR 包是否生成（英文提示，避免乱码）
                bat '''
                    if not exist "target/MVC.war" (
                        echo "ERROR: WAR package not generated!"
                        exit 1
                    ) else (
                        echo "WAR package generated successfully: target/MVC.war"
                    )
                '''
            }
        }
        
        stage('运行测试') {
            steps {
                echo "Running unit tests..."
                bat 'mvn test'
            }
        }
        
        stage('部署到服务器') {
            steps {
                echo "Deploying from Windows to Linux server..."
                
                // 1. 本地验证WAR包（Windows路径）
                bat 'if not exist "target\\MVC.war" ( echo "WAR包不存在！" && exit 1 )'
                
                // 2. 传输文件并执行简化的Linux命令（避免换行符问题）
                sshPublisher(publishers: [
                    sshPublisherDesc(
                        configName: 'my-server',
                        transfers: [
                            sshTransfer(
                                sourceFiles: 'target/MVC.war',  // 插件会自动处理Windows到Linux的路径转换
                                remoteDirectory: '/root/apache-tomcat-10.1.19/webapps',
                                flatten: true,
                                // 关键：使用单行命令，避免Windows换行符影响Linux解析
                                execCommand: 'echo "=== 服务器信息 ==="; whoami; pwd; echo "=== 检查WAR包 ==="; ls -l /root/apache-tomcat-10.1.19/webapps/MVC.war; echo "=== 检查Tomcat路径 ==="; ls -l /root/apache-tomcat-10.1.19/bin/shutdown.sh'
                            )
                        ],
                        verbose: true,  // 输出详细传输日志
                        timeout: 60000
                    )
                ])
            }
        }
    }
    
    post {
        success {
            echo "=============================================="
            echo "🎉 Build and deployment completed successfully!"
            echo "Access URL: http://111.230.94.55:8080/MVC"
            echo "=============================================="
        }
        failure {
            echo "=============================================="
            echo "❌ Build or deployment failed. Check console logs for details."
            echo "=============================================="
        }
    }
}
