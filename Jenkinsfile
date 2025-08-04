pipeline {
    agent any  // 使用任意可用的 Jenkins 节点
    tools {
        maven 'ceshi1'  // Jenkins 中配置的 Maven 名称（需与全局工具配置一致）
        jdk 'JDK'       // Jenkins 中配置的 JDK 名称（需与全局工具配置一致）
    }
    environment {
        // 项目相关配置（根据实际情况修改）
        PROJECT_NAME = 'MVC'
        WAR_FILE = "target/${PROJECT_NAME}.war"
        REMOTE_TOMCAT_WEBAPPS = '/root/apache-tomcat-10.1.19/webapps'
        REMOTE_DEPLOY_SCRIPT = '/root/deploy_mvc.sh'  // 远程服务器部署脚本路径
    }
    stages {
        stage('la qu dai ma') {
            steps {
                echo "从 GitHub 拉取 main 分支代码..."
                script {
                    // 优化 Git 网络配置，避免拉取失败
                    bat '''
                        git config --global http.postBuffer 524288000
                        git config --global http.sslVerify false
                    '''
                    // 使用凭据拉取代码（替换为你的凭据 ID）
                    git url: 'https://github.com/msg-555/mvc-.git',
                        branch: 'main',
                        credentialsId: 'b22d5859-a10f-4cfb-bf76-9460f4bf46a3'
                }
            }
        }

        stage('gou jian xiang mu') {
            steps {
                echo "使用 Maven 构建 WAR 包..."
                bat 'mvn clean package -Dmaven.test.skip=true'  // Windows 用 bat，Linux 用 sh

                // 验证 WAR 包是否生成
                bat """
                    if not exist "${env.WAR_FILE}" (
                        echo "ERROR: WAR 包未生成！路径：${env.WAR_FILE}"
                        exit 1
                    ) else (
                        echo "WAR 包生成成功：${env.WAR_FILE}"
                        dir "${env.WAR_FILE}"  // 显示 WAR 包详细信息
                    )
                """
            }
        }

        stage('yun xin ce shi') {
            steps {
                echo "执行单元测试..."
                bat 'mvn test'  // Windows 用 bat，Linux 用 sh
            }
        }

        stage('bu shu dao fu wu qi') {
            steps {
                echo "kai shi bu shu ${env.PROJECT_NAME} 到服务器..."
                script {
                    // 远程服务器配置（替换为你的服务器信息）
                    def remote = [
                        name: 'my-server',  // Jenkins 中配置的 SSH 服务器名称
                        host: '111.230.94.55',  // 服务器 IP 地址
                        user: 'root',  // 登录用户名
                        credentialsId: 'b22d5859-a10f-4cfb-bf76-9460f4bf46a3',  // SSH 凭据 ID
                        port: 22  // SSH 端口，默认 22
                    ]

                    // 步骤 1：上传 WAR 包到服务器 Tomcat 的 webapps 目录
                    echo "shang chuang WAR bao 到服务器..."
                    sshPut remote: remote,
                        from: env.WAR_FILE,
                        into: env.REMOTE_TOMCAT_WEBAPPS,
                        override: true  // 覆盖已存在的文件

                    // 步骤 2：执行远程部署脚本（停止 Tomcat、备份、启动等）
                    echo "zhi xin jiao ben 远程部署脚本..."
                    def deployOutput = sshCommand remote: remote,
                        command: "/bin/bash ${env.REMOTE_DEPLOY_SCRIPT}",  // 确保脚本有执行权限
                        returnStdout: true

                    // 输出部署脚本执行结果
                    echo "部署脚本输出："
                    echo deployOutput
                }
            }
        }
    }

    post {
        success {
            echo "=============================================="
            echo "🎉 构建部署成功！"
            echo "访问地址：http://111.230.94.55:8080/${env.PROJECT_NAME}"  // 替换为你的服务器 IP
            echo "=============================================="
        }
        failure {
            echo "=============================================="
            echo "❌ 构建或部署失败，请查看控制台日志排查问题"
            echo "=============================================="
        }
    }
}
