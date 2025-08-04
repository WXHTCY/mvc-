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
                
                bat '''
                    if not exist "target\\MVC.war" (
                        echo "ERROR: WAR package not generated!"
                        exit 1
                    ) else (
                        echo "WAR package generated: target\\MVC.war"
                        dir /s/b target\\MVC.war
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
                echo "Deploying WAR package to Tomcat..."
                
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
                                    #!/bin/bash
                                    echo "=== 服务器部署验证 ==="
                                    DEPLOY_PATH="/root/apache-tomcat-10.1.19/webapps"
                                    WAR_FILE="$DEPLOY_PATH/MVC.war"
                                    
                                    # 1. 验证文件是否存在
                                    if [ ! -f "$WAR_FILE" ]; then
                                        echo "❌ 错误: WAR文件未找到: $WAR_FILE"
                                        exit 1
                                    fi
                                    echo "✅ 找到WAR文件: $(ls -lh $WAR_FILE)"
                                    
                                    # 2. 重启Tomcat服务
                                    echo "停止Tomcat服务..."
                                    if sudo systemctl stop tomcat; then
                                        echo "Tomcat已停止"
                                        sleep 3
                                    else
                                        echo "⚠️ 警告: 停止Tomcat失败 (可能未运行)"
                                    fi
                                    
                                    # 3. 清理旧部署
                                    echo "清理旧应用: $DEPLOY_PATH/MVC*"
                                    sudo rm -rf $DEPLOY_PATH/MVC*
                                    
                                    # 4. 启动Tomcat
                                    echo "启动Tomcat..."
                                    if sudo systemctl start tomcat; then
                                        echo "✅ Tomcat启动成功"
                                        echo "等待应用部署..."
                                        sleep 15
                                        echo "当前webapps内容:"
                                        ls -l $DEPLOY_PATH
                                    else
                                        echo "❌ 错误: 启动Tomcat失败"
                                        exit 1
                                    fi
                                '''
                            )
                        ]
                    )
                ])
            }
        }
    }
    post {
        success {
            echo "========================================"
            echo "🎉 构建部署成功!"
            echo "访问地址: http://111.230.94.55:8080/MVC"
            echo "========================================"
        }
        failure {
            echo "========================================"
            echo "❌ 构建或部署失败，请检查日志"
            echo "========================================"
        }
    }
}
