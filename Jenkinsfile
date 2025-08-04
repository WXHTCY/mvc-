pipeline {
    agent any
    tools {
        maven 'ceshi1'  // 确保与 Jenkins 中配置的 Maven 名称一致
        jdk 'JDK'       // 确保与 Jenkins 中配置的 JDK 名称一致
    }
    stages {
        stage('拉取代码') {
            steps {
                echo "Pulling code from GitHub main branch..."
                git url: 'https://github.com/msg-555/mvc-.git', 
                    branch: 'main',
                    credentialsId: 'b22d5859-a10f-4cfb-bf76-9460f4bf46a3'  // 替换为你的 GitHub 凭据 ID
            }
        }
        
        stage('构建项目') {
            steps {
                echo "Building WAR package with Maven..."
                bat 'mvn clean package -Dmaven.test.skip=true'  // Windows 用 bat，Linux 用 sh
                // 检查 WAR 包是否生成
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
                bat 'mvn test'  // Windows 用 bat，Linux 用 sh
            }
        }
        
        stage('部署到服务器') {
            steps {
                echo "Deploying WAR package to server Tomcat directory..."
                bat 'dir "target\\MVC.war"'  // 验证本地 WAR 包存在
                
                sshPublisher(publishers: [
                    sshPublisherDesc(
                        configName: 'my-server',  // 确保与 Jenkins 中配置的 SSH 服务器名称一致
                        transfers: [
                            sshTransfer(
                                sourceFiles: 'target/MVC.war',
                                remoteDirectory: '/root/apache-tomcat-10.1.19/webapps',
                                cleanRemote: false,
                                flatten: true,
                                // 修正：用 bash -c 包裹命令，添加进程清理逻辑
                                execCommand: '''bash -c '
                                    echo "=== Server deployment verification ==="
                                    echo "Checking WAR package in webapps directory..."
                                    ls -l /root/apache-tomcat-10.1.19/webapps/MVC.war || echo "WAR package upload failed!"
                                    
                                    echo "Stopping Tomcat service..."
                                    /root/apache-tomcat-10.1.19/bin/shutdown.sh
                                    sleep 5
                                    
                                    # 强制杀死未停止的 Tomcat 进程
                                    TOMCAT_PID=$(ps -ef | grep /root/apache-tomcat-10.1.19 | grep -v grep | awk '{print $2}')
                                    if [ -n "$TOMCAT_PID" ]; then
                                        echo "Force killing Tomcat PID: $TOMCAT_PID"
                                        kill -9 $TOMCAT_PID
                                        sleep 3
                                    fi
                                    
                                    echo "Cleaning old deployment files..."
                                    rm -rf /root/apache-tomcat-10.1.19/webapps/MVC*
                                    
                                    echo "Starting Tomcat after confirming WAR exists..."
                                    if [ -f "/root/apache-tomcat-10.1.19/webapps/MVC.war" ]; then
                                        /root/apache-tomcat-10.1.19/bin/startup.sh
                                        sleep 10
                                        echo "Webapps directory after deployment:"
                                        ls -l /root/apache-tomcat-10.1.19/webapps
                                    else
                                        echo "ERROR: MVC.war not found on server, deployment aborted!"
                                        exit 1
                                    fi
                                ' '''
                            )
                        ],
                        verbose: true,    // 输出详细日志
                        timeout: 180000   // 超时时间 3 分钟
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
