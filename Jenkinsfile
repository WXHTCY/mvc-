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
            sshPublisher(
              publishers: [
                sshPublisherDesc(
                  configName: "my-server",
                  transfers: [
                    sshTransfer(
                      sourceFiles: "target/MVC.war",
                      remoteDirectory: "/root/apache-tomcat-10.1.19/webapps",
                      execCommand: """
                        # 重启服务
                        systemctl restart tomcat
                        
                        # 验证部署
                        if systemctl is-active tomcat; then
                          echo "Deployment SUCCESS"
                        else
                          echo "Deployment FAILED"
                          journalctl -u tomcat -n 50 --no-pager
                          exit 1
                        fi
                      """
                    )
                  ]
                )
              ]
            )
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
