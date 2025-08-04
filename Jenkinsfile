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
                echo "Deploying WAR package to server..."
                bat 'dir "target\\MVC.war"'
                
                sshPublisher(publishers: [
                    sshPublisherDesc(
                        configName: 'my-server',
                        transfers: [
                            sshTransfer(
                                sourceFiles: 'target/MVC.war',
                                remoteDirectory: '/root/apache-tomcat-10.1.19/webapps',
                                cleanRemote: false,
                                flatten: true,
                                // 关键：用单引号包裹命令，避免Windows换行符干扰，强制输出
                                execCommand: '/bin/bash -c \'
                                    # 强制创建日志文件（即使后续失败，也能确认执行到这一步）
                                    LOG_FILE="/tmp/jenkins_deploy.log"
                                    > "$LOG_FILE"  # 清空并创建文件
                                    
                                    # 输出基础信息到日志和控制台
                                    echo "=== 脚本开始执行: $(date) ===" | tee -a "$LOG_FILE"
                                    echo "当前用户: $(whoami)" | tee -a "$LOG_FILE"
                                    echo "当前目录: $(pwd)" | tee -a "$LOG_FILE"
                                    echo "Tomcat目录检查: /root/apache-tomcat-10.1.19" | tee -a "$LOG_FILE"
                                    
                                    # 验证Tomcat目录是否存在
                                    if [ -d "/root/apache-tomcat-10.1.19" ]; then
                                        echo "Tomcat目录存在" | tee -a "$LOG_FILE"
                                    else
                                        echo "ERROR: Tomcat目录不存在！" | tee -a "$LOG_FILE"
                                        exit 1
                                    fi
                                    
                                    # 验证WAR包是否上传成功
                                    WAR_PATH="/root/apache-tomcat-10.1.19/webapps/MVC.war"
                                    if [ -f "$WAR_PATH" ]; then
                                        echo "WAR包存在: $WAR_PATH" | tee -a "$LOG_FILE"
                                    else
                                        echo "ERROR: WAR包未找到！" | tee -a "$LOG_FILE"
                                        exit 1
                                    fi
                                    
                                    # 测试基础命令执行（确保脚本可正常运行）
                                    echo "=== 测试命令执行成功 ===" | tee -a "$LOG_FILE"
                                \''  # 注意：此处用单引号+反斜杠转义，避免格式错误
                            )
                        ],
                        verbose: true,
                        timeout: 180000
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
