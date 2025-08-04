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
                
                // 检查 WAR 包是否生成并显示详细信息
                bat '''
                    echo "Checking WAR package existence..."
                    dir "target"
                    if not exist "target/MVC.war" (
                        echo "ERROR: WAR package not generated!"
                        exit 1
                    ) else (
                        echo "WAR package generated successfully:"
                        dir "target/MVC.war"
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
        
        stage('验证文件准备') {
            steps {
                echo "Verifying WAR file before upload..."
                script {
                    def warFile = fileExists('target/MVC.war')
                    if (!warFile) {
                        error("WAR file not found! Cannot proceed with deployment.")
                    } else {
                        echo "WAR file confirmed: target/MVC.war"
                    }
                }
            }
        }
        
        stage('部署到服务器') {
            steps {
                echo "Starting deployment process..."
                
                // 增加详细的上传日志
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
                                    echo "=== Starting deployment verification ==="
                                    echo "Current directory contents:"
                                    pwd
                                    ls -la
                                    
                                    echo "Webapps directory before deployment:"
                                    ls -la /root/apache-tomcat-10.1.19/webapps/
                                    
                                    echo "Checking uploaded WAR package..."
                                    if [ -f "/root/apache-tomcat-10.1.19/webapps/MVC.war" ]; then
                                        echo "WAR package uploaded successfully!"
                                        echo "File details:"
                                        ls -l /root/apache-tomcat-10.1.19/webapps/MVC.war
                                        
                                        echo "Stopping Tomcat service..."
                                        /root/apache-tomcat-10.1.19/bin/shutdown.sh
                                        sleep 5
                                        
                                        echo "Cleaning old deployment files..."
                                        rm -rf /root/apache-tomcat-10.1.19/webapps/MVC*
                                        
                                        echo "Starting Tomcat..."
                                        /root/apache-tomcat-10.1.19/bin/startup.sh
                                        sleep 10
                                        
                                        echo "Webapps directory after deployment:"
                                        ls -l /root/apache-tomcat-10.1.19/webapps
                                    else
                                        echo "ERROR: MVC.war not found on server after upload attempt!"
                                        exit 1
                                    fi
                                '''
                            )
                        ],
                        // 增加SSH操作的日志输出
                        verbose: true
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
