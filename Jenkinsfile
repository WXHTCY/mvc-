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
                bat 'dir "target\\MVC.war"'  // 本地验证WAR包存在
                
                sshPublisher(publishers: [
                    sshPublisherDesc(
                        configName: 'my-server',  // 确保与Jenkins SSH配置名称一致
                        transfers: [
                            sshTransfer(
                                sourceFiles: 'target/MVC.war',
                                remoteDirectory: '/root/apache-tomcat-10.1.19/webapps',
                                cleanRemote: false,
                                flatten: true,
                                // 核心优化：详细日志+分步错误检查
                                execCommand: '''sh -c '
                                    # 开启命令追踪，输出每一步执行的命令
                                    set -x
                                    
                                    # 1. 检查服务器环境与权限
                                    echo "=== 服务器基础信息 ==="
                                    whoami  # 确认执行用户（需为root或有足够权限）
                                    pwd     # 确认当前目录
                                    ls -ld /root  # 检查/root目录权限（需r-x权限）
                                    ls -ld /root/apache-tomcat-10.1.19/webapps  # 确认部署目录权限
                                    
                                    # 2. 验证WAR包上传结果
                                    echo "=== 验证WAR包上传 ==="
                                    if [ ! -f "/root/apache-tomcat-10.1.19/webapps/MVC.war" ]; then
                                        echo "ERROR: WAR包未上传成功！"
                                        exit 1
                                    fi
                                    ls -l /root/apache-tomcat-10.1.19/webapps/MVC.war  # 确认文件大小正常
                                    
                                    # 3. 停止Tomcat服务（增强容错）
                                    echo "=== 停止Tomcat服务 ==="
                                    TOMCAT_SHUTDOWN="/root/apache-tomcat-10.1.19/bin/shutdown.sh"
                                    if [ ! -f "$TOMCAT_SHUTDOWN" ]; then
                                        echo "ERROR: 停止脚本不存在！路径：$TOMCAT_SHUTDOWN"
                                        exit 1
                                    fi
                                    # 执行停止命令并检查返回码
                                    "$TOMCAT_SHUTDOWN"
                                    shutdown_exit=$?
                                    if [ $shutdown_exit -ne 0 ]; then
                                        echo "WARNING: Tomcat正常停止失败，尝试强制终止进程"
                                        # 强制杀死Tomcat进程（避免端口占用）
                                        ps -ef | grep tomcat | grep -v grep | awk '{print $2}' | xargs kill -9 2>/dev/null
                                        sleep 3
                                    fi
                                    # 确认Tomcat进程已终止
                                    if ps -ef | grep tomcat | grep -v grep >/dev/null; then
                                        echo "ERROR: Tomcat进程强制终止失败！"
                                        exit 1
                                    fi
                                    
                                    # 4. 清理旧部署文件（增加容错）
                                    echo "=== 清理旧文件 ==="
                                    rm -rf /root/apache-tomcat-10.1.19/webapps/MVC*
                                    rm_exit=$?
                                    if [ $rm_exit -ne 0 ]; then
                                        echo "ERROR: 清理旧文件失败！可能文件被占用或权限不足"
                                        exit 1
                                    fi
                                    
                                    # 5. 再次确认WAR包存在（防止清理时误删）
                                    if [ ! -f "/root/apache-tomcat-10.1.19/webapps/MVC.war" ]; then
                                        echo "ERROR: WAR包丢失！清理步骤可能误删文件"
                                        exit 1
                                    fi
                                    
                                    # 6. 修复Tomcat脚本权限和换行符（关键步骤）
                                    echo "=== 修复Tomcat脚本 ==="
                                    # 为所有脚本添加执行权限
                                    chmod +x /root/apache-tomcat-10.1.19/bin/*.sh
                                    # 转换换行符为Linux格式（解决直接解压的脚本问题）
                                    dos2unix /root/apache-tomcat-10.1.19/bin/*.sh 2>/dev/null || true
                                    
                                    # 7. 启动Tomcat服务
                                    echo "=== 启动Tomcat服务 ==="
                                    TOMCAT_STARTUP="/root/apache-tomcat-10.1.19/bin/startup.sh"
                                    if [ ! -f "$TOMCAT_STARTUP" ]; then
                                        echo "ERROR: 启动脚本不存在！路径：$TOMCAT_STARTUP"
                                        exit 1
                                    fi
                                    # 执行启动命令并检查返回码
                                    "$TOMCAT_STARTUP"
                                    startup_exit=$?
                                    if [ $startup_exit -ne 0 ]; then
                                        echo "ERROR: Tomcat启动脚本执行失败！"
                                        exit 1
                                    fi
                                    
                                    # 8. 验证部署结果
                                    echo "=== 部署结果验证 ==="
                                    sleep 10  # 等待Tomcat初始化
                                    ps -ef | grep tomcat | grep -v grep  # 确认Tomcat进程存在
                                    ls -l /root/apache-tomcat-10.1.19/webapps  # 确认应用已解压
                                ' '''
                            )
                        ],
                        verbose: true,  // 输出SSH详细日志
                        timeout: 180000  // 延长超时时间至3分钟
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
