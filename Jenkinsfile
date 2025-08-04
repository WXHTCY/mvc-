pipeline {
    agent any
    
    environment {
        // 项目配置
        PROJECT_NAME = "MVC"
        WAR_FILE = "target/${PROJECT_NAME}.war"
        
        // 服务器配置
        DEPLOY_USER = "root"
        TOMCAT_HOME = "/root/apache-tomcat-10.1.19"
        WEBAPPS_DIR = "${TOMCAT_HOME}/webapps"
        
        // 构建工具配置
        MAVEN_HOME = tool 'M3'
        JAVA_HOME = tool 'JDK17'
        PATH = "${MAVEN_HOME}/bin:${JAVA_HOME}/bin:${env.PATH}"
    }
    
    stages {
        stage('拉取代码') {
            steps {
                echo "从GitHub拉取主分支代码..."
                git url: 'https://github.com/msg-555/mvc-.git', branch: 'main'
            }
        }
        
        stage('构建项目') {
            steps {
                echo "使用Maven构建WAR包..."
                script {
                    // 使用--no-transfer-progress减少日志输出
                    sh "mvn clean package -Dmaven.test.skip=true --no-transfer-progress"
                    
                    // 验证WAR文件生成
                    if (!fileExists(WAR_FILE)) {
                        error "❌ WAR文件未生成: ${WAR_FILE}"
                    } else {
                        def warSize = sh(script: "du -h ${WAR_FILE} | cut -f1", returnStdout: true).trim()
                        echo "✅ WAR包生成成功: ${WAR_FILE} (大小: ${warSize})"
                    }
                }
            }
        }
        
        stage('运行测试') {
            steps {
                echo "运行单元测试..."
                sh "mvn test --no-transfer-progress"
            }
        }
        
        stage('部署到服务器') {
            steps {
                echo "部署WAR包到Tomcat服务器..."
                script {
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: "my-server",
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: WAR_FILE,
                                        remoteDirectory: "apache-tomcat-10.1.19/webapps", // 关键修复：使用相对路径
                                        removePrefix: "target",
                                        execCommand: """
                                            # 部署验证脚本
                                            echo '===== 部署验证 ====='
                                            echo "服务器时间: $(date)"
                                            echo "Tomcat目录: ${TOMCAT_HOME}"
                                            
                                            # 1. 验证文件位置
                                            DEPLOYED_WAR="${TOMCAT_HOME}/webapps/${PROJECT_NAME}.war"
                                            if [ ! -f "\$DEPLOYED_WAR" ]; then
                                                echo "❌ 错误: WAR文件未找到!"
                                                echo "当前目录内容:"
                                                pwd && ls -l
                                                echo "目标目录内容:"
                                                ls -l ${TOMCAT_HOME}/webapps/
                                                exit 1
                                            fi
                                            
                                            # 2. 检查文件大小
                                            FILE_SIZE=\$(du -h "\$DEPLOYED_WAR" | cut -f1)
                                            echo "✅ WAR文件位置正确: \$DEPLOYED_WAR (大小: \$FILE_SIZE)"
                                            
                                            # 3. 重启Tomcat服务
                                            echo "重启Tomcat服务..."
                                            systemctl restart tomcat
                                            sleep 5  # 等待服务启动
                                            
                                            # 4. 检查服务状态
                                            SERVICE_STATUS=\$(systemctl is-active tomcat)
                                            if [ "\$SERVICE_STATUS" != "active" ]; then
                                                echo "❌ Tomcat服务未运行! 状态: \$SERVICE_STATUS"
                                                journalctl -u tomcat -n 20 --no-pager
                                                exit 1
                                            else
                                                echo "✅ Tomcat服务状态: \$SERVICE_STATUS"
                                            fi
                                            
                                            # 5. 验证应用目录
                                            if [ ! -d "${TOMCAT_HOME}/webapps/${PROJECT_NAME}" ]; then
                                                echo "❌ 应用未解压部署!"
                                                echo "请检查Tomcat日志: ${TOMCAT_HOME}/logs/catalina.out"
                                                exit 1
                                            else
                                                echo "✅ 应用已成功部署: ${TOMCAT_HOME}/webapps/${PROJECT_NAME}"
                                            fi
                                            
                                            # 6. 可选：简单HTTP检查
                                            echo "应用访问测试..."
                                            curl -Is http://localhost:8080/${PROJECT_NAME} | head -n 1 || echo "⚠️ 应用访问测试失败，请手动验证"
                                            
                                            echo "===== 部署成功! ====="
                                        """
                                    )
                                ],
                                usePromotionTimestamp: false,
                                useWorkspaceInPromotion: false,
                                verbose: true  // 启用详细日志
                            )
                        ]
                    )
                }
            }
        }
    }
    
    post {
        always {
            echo "构建状态: ${currentBuild.currentResult}"
            // 清理工作空间（可选）
            cleanWs()
        }
        success {
            echo "🎉 部署成功! 应用URL: http://111.230.94.55:8080/${PROJECT_NAME}"
            // 可以添加成功通知（邮件、钉钉等）
        }
        failure {
            echo "❌ 构建失败! 请检查日志"
            // 可以添加失败通知
        }
    }
}
