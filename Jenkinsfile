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
                echo "Deploying WAR package to server Tomcat directory..."
                // 本地验证WAR包存在（Windows路径）
                bat 'dir "target\\MVC.war"'  // 保持Windows路径格式正确
                
                sshPublisher(publishers: [
                    sshPublisherDesc(
                        configName: 'my-server',
                        transfers: [
                            sshTransfer(
                                sourceFiles: 'target/MVC.war',
                                remoteDirectory: '/root/apache-tomcat-10.1.19/webapps',
                                cleanRemote: false,
                                flatten: true,
                                // 优化服务器端命令：增加错误捕获和状态校验
                                execCommand: '''
                                    # 开启命令执行日志（每步打印，类似手动操作时的实时查看）
                                    set -x
                                    
                                    # 1. 严格检查WAR包是否上传成功（不存在则立即终止）
                                    echo "=== 1. 检查WAR包是否存在 ==="
                                    if [ ! -f "/root/apache-tomcat-10.1.19/webapps/MVC.war" ]; then
                                        echo "ERROR: WAR包未上传到服务器！"
                                        exit 1  # 终止部署，与手动执行时的"不上传则不继续"一致
                                    fi
                                    
                                    # 2. 停止Tomcat服务（确保停止成功）
                                    echo "=== 2. 停止Tomcat服务 ==="
                                    if [ ! -f "/root/apache-tomcat-10.1.19/bin/shutdown.sh" ]; then
                                        echo "ERROR: Tomcat停止脚本不存在！"
                                        exit 1
                                    fi
                                    /root/apache-tomcat-10.1.19/bin/shutdown.sh
                                    shutdown_exit_code=$?  # 捕获命令返回码
                                    if [ $shutdown_exit_code -ne 0 ]; then
                                        echo "ERROR: Tomcat停止失败，返回码：$shutdown_exit_code"
                                        # 手动执行时会强制杀进程，脚本中增加容错
                                        ps -ef | grep tomcat | grep -v grep | awk '{print $2}' | xargs kill -9 2>/dev/null
                                        sleep 3
                                    fi
                                    
                                    # 3. 清理旧部署文件（确保清理成功）
                                    echo "=== 3. 清理旧文件 ==="
                                    rm -rf /root/apache-tomcat-10.1.19/webapps/MVC*
                                    rm_exit_code=$?
                                    if [ $rm_exit_code -ne 0 ]; then
                                        echo "ERROR: 旧文件清理失败，返回码：$rm_exit_code"
                                        exit 1  # 清理失败会导致部署冲突，必须终止
                                    fi
                                    
                                    # 4. 再次确认WAR包存在（防止清理时误删）
                                    echo "=== 4. 再次确认WAR包 ==="
                                    if [ ! -f "/root/apache-tomcat-10.1.19/webapps/MVC.war" ]; then
                                        echo "ERROR: WAR包丢失，部署终止！"
                                        exit 1
                                    fi
                                    
                                    # 5. 启动Tomcat服务（确保启动成功）
                                    echo "=== 5. 启动Tomcat服务 ==="
                                    if [ ! -f "/root/apache-tomcat-10.1.19/bin/startup.sh" ]; then
                                        echo "ERROR: Tomcat启动脚本不存在！"
                                        exit 1
                                    fi
                                    /root/apache-tomcat-10.1.19/bin/startup.sh
                                    startup_exit_code=$?
                                    if [ $startup_exit_code -ne 0 ]; then
                                        echo "ERROR: Tomcat启动失败，返回码：$startup_exit_code"
                                        exit 1
                                    fi
                                    
                                    # 6. 验证部署结果（类似手动执行后的检查）
                                    echo "=== 6. 部署结果验证 ==="
                                    sleep 10  # 等待Tomcat解压WAR包
                                    ls -l /root/apache-tomcat-10.1.19/webapps | grep MVC
                                    echo "Tomcat进程状态：$(ps -ef | grep tomcat | grep -v grep)"
                                    echo "部署成功！"
                                '''
                            )
                        ])
                    ])
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
