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
                echo "Deploying to Linux Tomcat..."
                bat 'dir "target\\MVC.war"'  // 本地验证WAR包
                
                sshPublisher(publishers: [
                    sshPublisherDesc(
                        configName: 'my-server',
                        transfers: [
                            sshTransfer(
                                sourceFiles: 'target/MVC.war',
                                remoteDirectory: '/root/apache-tomcat-10.1.19/webapps',
                                flatten: true,
                                // 执行修复后的启停命令，增加日志输出
                                execCommand: 'sh -c \'
                                    echo "=== 检查 WAR 包 ===";
                                    ls -l /root/apache-tomcat-10.1.19/webapps/MVC.war || { echo "WAR包不存在！"; exit 1; };
                                    
                                    echo "=== 停止 Tomcat ===";
                                    /root/apache-tomcat-10.1.19/bin/shutdown.sh;
                                    sleep 5;
                                    
                                    echo "=== 清理旧文件 ===";
                                    rm -rf /root/apache-tomcat-10.1.19/webapps/MVC*;
                                    
                                    echo "=== 确认 WAR 包存在 ===";
                                    ls -l /root/apache-tomcat-10.1.19/webapps/MVC.war || { echo "WAR包丢失！"; exit 1; };
                                    
                                    echo "=== 启动 Tomcat ===";
                                    /root/apache-tomcat-10.1.19/bin/startup.sh;
                                    sleep 5;
                                    
                                    echo "=== 部署完成，Tomcat 进程 ===";
                                    ps -ef | grep tomcat;
                                \''
                            )
                        ],
                        verbose: true,
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
