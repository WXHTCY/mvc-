pipeline {
    agent any
    tools {
        maven 'ceshi1'  // 与Jenkins中配置的Maven名称一致
        jdk 'JDK'       // 与Jenkins中配置的JDK名称一致
    }
    environment {
        // 项目核心配置
        PROJECT_NAME = 'MVC'
        WAR_FILE = "target/${PROJECT_NAME}.war"
        REMOTE_TOMCAT_WEBAPPS = '/root/apache-tomcat-10.1.19/webapps'
    }
    stages {
        stage('拉取代码（SSH）') {
            steps {
                echo "通过SSH协议拉取GitHub代码..."
                git(
                    url: 'git@github.com:msg-555/mvc-.git',  // SSH地址
                    branch: 'main',
                    credentialsId: 'github-ssh-credentials'  // SSH凭据ID
                )
            }
        }

        stage('构建项目') {
            steps {
                echo "使用Maven构建WAR包..."
                bat 'mvn clean package -Dmaven.test.skip=true'  // Windows环境用bat

                // 验证WAR包是否生成
                bat """
                    if not exist "${env.WAR_FILE}" (
                        echo "ERROR: WAR包未生成！路径：${env.WAR_FILE}"
                        exit 1
                    ) else (
                        echo "WAR包生成成功：${env.WAR_FILE}"
                        dir "${env.WAR_FILE}"
                    )
                """
            }
        }

        stage('运行测试') {
            steps {
                echo "执行单元测试..."
                bat 'mvn test'  // Windows环境用bat
            }
        }

        stage('部署到服务器') {
            steps {
                echo "部署${env.PROJECT_NAME}到远程服务器..."
                script {
                    // 验证本地WAR包存在
                    bat "dir \"${env.WAR_FILE}\""

                    // 配置远程服务器信息
                    sshPublisher(publishers: [
                        sshPublisherDesc(
                            configName: 'my-server',  // Jenkins中配置的SSH服务器名称
                            transfers: [
                                sshTransfer(
                                    sourceFiles: env.WAR_FILE,
                                    remoteDirectory: env.REMOTE_TOMCAT_WEBAPPS,
                                    cleanRemote: false,
                                    flatten: true,
                                    // 服务器端部署命令（包含完整生命周期管理）
                                    execCommand: '''bash -c '
                                        # 定义变量
                                        TOMCAT_PATH="/root/apache-tomcat-10.1.19"
                                        WAR_NAME="MVC.war"
                                        
                                        echo "=== 开始部署: $(date) ==="
                                        
                                        # 停止Tomcat服务
                                        echo "停止Tomcat服务..."
                                        ${TOMCAT_PATH}/bin/shutdown.sh
                                        sleep 5
                                        
                                        # 强制杀死残留进程
                                        TOMCAT_PID=$(ps -ef | grep ${TOMCAT_PATH} | grep -v grep | awk '{print $2}')
                                        if [ -n "${TOMCAT_PID}" ]; then
                                            echo "强制杀死Tomcat进程: ${TOMCAT_PID}"
                                            kill -9 ${TOMCAT_PID}
                                            sleep 3
                                        fi
                                        
                                        # 清理旧部署文件
                                        echo "清理旧文件..."
                                        rm -rf ${TOMCAT_PATH}/webapps/${WAR_NAME}
                                        rm -rf ${TOMCAT_PATH}/webapps/${WAR_NAME%.war}  # 清理解压目录
                                        rm -rf ${TOMCAT_PATH}/work/*  # 清理工作目录
                                        
                                        # 验证新WAR包是否上传成功
                                        if [ -f "${TOMCAT_PATH}/webapps/${WAR_NAME}" ]; then
                                            echo "WAR包上传成功，启动Tomcat..."
                                            ${TOMCAT_PATH}/bin/startup.sh
                                            sleep 10
                                            
                                            # 验证部署结果
                                            if ps -ef | grep ${TOMCAT_PATH} | grep -v grep >/dev/null; then
                                                echo "=== Tomcat启动成功 ==="
                                                echo "部署完成，Webapps目录内容："
                                                ls -l ${TOMCAT_PATH}/webapps
                                            else
                                                echo "ERROR: Tomcat启动失败！"
                                                exit 1
                                            fi
                                        else
                                            echo "ERROR: WAR包未找到，部署失败！"
                                            exit 1
                                        fi
                                    ' '''
                                )
                            ],
                            verbose: true,    // 输出详细部署日志
                            timeout: 180000   // 超时时间3分钟（180000毫秒）
                        )
                    ])
                }
            }
        }
    }

    post {
        success {
            echo "=============================================="
            echo "🎉 构建部署成功！"
            echo "访问地址: http://111.230.94.55:8080/${env.PROJECT_NAME}"
            echo "=============================================="
        }
        failure {
            echo "=============================================="
            echo "❌ 构建或部署失败，请查看控制台日志"
            echo "=============================================="
        }
    }
}
