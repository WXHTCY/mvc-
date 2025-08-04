pipeline {
    agent any
    tools {
        maven 'ceshi1'
        jdk 'JDK'
    }
    // 增加环境变量配置，提高网络稳定性
    environment {
        // 配置Git超时和缓存，减少网络问题影响
        GIT_CONFIG = '''
            git config --global http.postBuffer 524288000
            git config --global http.sslVerify false
            git config --global core.compression 0
        '''
    }
    stages {
        stage('拉取代码') {
            steps {
                echo "Pulling code from GitHub main branch..."
                // 增加网络稳定性配置，使用凭据避免认证问题
                script {
                    // 预配置Git参数，提高网络兼容性
                    bat "${env.GIT_CONFIG}"
                    // 使用存储的凭据拉取代码，避免重复认证
                    git url: 'https://github.com/msg-555/mvc-.git', 
                        branch: 'main',
                        credentialsId: 'b22d5859-a10f-4cfb-bf76-9460f4bf46a3'
                }
            }
        }
        
        stage('构建项目') {
            steps {
                echo "Building WAR package with Maven..."
                bat 'mvn clean package -Dmaven.test.skip=true'
                // 检查WAR包是否生成
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
                                // 使用sh -c包裹命令，确保Linux正确解析换行符
                                execCommand: '''sh -c '
                                    echo "=== Server deployment verification ==="
                                    echo "Current user: $(whoami)"
                                    echo "Current directory: $(pwd)"
                                    
                                    echo "Checking WAR package in webapps directory..."
                                    ls -l /root/apache-tomcat-10.1.19/webapps/MVC.war || { echo "WAR package upload failed!"; exit 1; }
                                    
                                    echo "Stopping Tomcat service..."
                                    /root/apache-tomcat-10.1.19/bin/shutdown.sh
                                    sleep 5
                                    
                                    echo "Cleaning old deployment files..."
                                    rm -rf /root/apache-tomcat-10.1.19/webapps/MVC*
                                    
                                    echo "Verifying WAR package exists after cleanup..."
                                    if [ ! -f "/root/apache-tomcat-10.1.19/webapps/MVC.war" ]; then
                                        echo "ERROR: MVC.war missing after cleanup!"
                                        exit 1
                                    fi
                                    
                                    echo "Starting Tomcat..."
                                    /root/apache-tomcat-10.1.19/bin/startup.sh
                                    sleep 10
                                    
                                    echo "Tomcat process status:"
                                    ps -ef | grep tomcat | grep -v grep
                                    
                                    echo "Webapps directory after deployment:"
                                    ls -l /root/apache-tomcat-10.1.19/webapps
                                ' '''
                            )
                        ],
                        // 开启详细日志，便于排查问题
                        verbose: true,
                        // 延长超时时间，适应网络较慢的情况
                        timeout: 120000
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
