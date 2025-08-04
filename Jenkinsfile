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
        
        stage('部署到服务器') {
            steps {
                echo "Deploying WAR package to server Tomcat directory..."
                bat 'dir "target\\MVC.war"'  // 确认本地WAR包存在
                
                sshPublisher(publishers: [
                    sshPublisherDesc(
                        configName: 'my-server',
                        transfers: [
                            sshTransfer(
                                sourceFiles: 'target/MVC.war',
                                remoteDirectory: '/root/apache-tomcat-10.1.19/webapps',
                                cleanRemote: false,
                                flatten: true,
                                execCommand: '''
                                    # 输出详细日志，定位错误步骤
                                    set -x  # 开启命令执行日志（每步命令都会打印）
                                    
                                    echo "=== 1. 检查服务器webapps目录 ==="
                                    ls -la /root/apache-tomcat-10.1.19/webapps/ || { echo "ERROR: webapps目录不存在"; exit 1; }
                                    
                                    echo "=== 2. 检查WAR包是否上传成功 ==="
                                    if [ -f "/root/apache-tomcat-10.1.19/webapps/MVC.war" ]; then
                                        echo "WAR包已上传: $(ls -l /root/apache-tomcat-10.1.19/webapps/MVC.war)"
                                    else
                                        echo "ERROR: WAR包未找到，上传失败"
                                        exit 1
                                    fi
                                    
                                    echo "=== 3. 停止Tomcat服务 ==="
                                    /root/apache-tomcat-10.1.19/bin/shutdown.sh || { echo "ERROR: Tomcat停止失败"; exit 1; }
                                    sleep 5
                                    # 强制杀死残留进程（可选）
                                    ps -ef | grep tomcat | grep -v grep | awk '{print $2}' | xargs kill -9 2>/dev/null
                                    
                                    echo "=== 4. 清理旧部署文件 ==="
                                    rm -rf /root/apache-tomcat-10.1.19/webapps/MVC* || { echo "ERROR: 清理旧文件失败"; exit 1; }
                                    
                                    echo "=== 5. 启动Tomcat服务 ==="
                                    /root/apache-tomcat-10.1.19/bin/startup.sh || { echo "ERROR: Tomcat启动失败"; exit 1; }
                                    sleep 10
                                    
                                    echo "=== 6. 验证部署结果 ==="
                                    ls -la /root/apache-tomcat-10.1.19/webapps/ | grep MVC
                                    echo "Tomcat进程状态: $(ps -ef | grep tomcat | grep -v grep)"
                                '''
                            )
                        ],
                        verbose: true  // 输出SSH详细日志
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
