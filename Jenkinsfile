pipeline {
    agent any  // 使用任意可用的 Jenkins 节点
    tools {
        maven 'ceshi1'  // 引用 Jenkins 中配置的 Maven 名称（需与全局工具配置一致）
        jdk 'JDK'       // 引用 Jenkins 中配置的 JDK 名称（需与全局工具配置一致）
    }
    stages {   // 流水线阶段定义
        stage('拉取代码') {  // 阶段 1：从 Git 仓库拉取代码
            steps {
                echo "从 GitHub 拉取 main 分支代码..."
                git url: 'https://github.com/msg-555/mvc-.git', branch: 'main'  // 替换为你的仓库地址
            }
        }
        
        stage('构建项目') {  // 阶段 2：编译并打包项目
            steps {
                echo "使用 Maven 构建 WAR 包..."
                bat 'mvn clean package -Dmaven.test.skip=true'  // Windows 环境用 bat 命令，跳过测试加速构建
                // 若为 Linux 环境，替换为：sh 'mvn clean package -Dmaven.test.skip=true'
            }
        }
        
        stage('运行测试') {  // 阶段 3：执行单元测试（可选，根据项目需求启用）
            steps {
                echo "执行单元测试..."
                bat 'mvn test'  // Windows 用 bat，Linux 用 sh
            }
        }
        
        stage('部署到服务器') {  // 阶段 4：部署到服务器并重启 Tomcat
            steps {
                echo "部署 WAR 包到服务器 Tomcat 目录..."
                sshPublisher(publishers: [
                    sshPublisherDesc(
                        configName: 'my-server',  // 必须与 Jenkins 中配置的 SSH 服务器名称一致
                        transfers: [
                            sshTransfer(
                                sourceFiles: 'target/MVC.war',  // 本地构建好的 WAR 包路径（根据实际文件名调整）
                                remoteDirectory: '/root/apache-tomcat-10.1.19/webapps',  // 服务器 Tomcat 部署目录
                                cleanRemote: false,  // 禁用清空服务器目录，避免删除其他文件
                                flatten: true,       // 仅上传 WAR 包，不保留本地目录结构
                                execCommand: '''  // 部署后执行服务器命令（重启 Tomcat）
                                    echo "停止 Tomcat 服务..."
                                    /root/apache-tomcat-10.1.19/bin/shutdown.sh
                                    sleep 5  // 等待 5 秒确保进程终止
                                    
                                    echo "清理旧部署文件（可选）..."
                                    rm -rf /root/apache-tomcat-10.1.19/webapps/MVC*  // 删除旧版本项目目录和 WAR 包
                                    
                                    echo "启动 Tomcat 服务..."
                                    /root/apache-tomcat-10.1.19/bin/startup.sh
                                '''
                            )
                        ]
                    )
                ])
            }
        }
    }
    
    post {  // 构建完成后的操作
        success {
            echo "=============================================="
            echo "🎉 构建部署成功！"
            echo "访问地址：http://111.230.94.55:8080/MVC"  // 假设 WAR 包名为 MVC.war
            echo "=============================================="
        }
        failure {
            echo "=============================================="
            echo "❌ 构建或部署失败，请查看控制台日志排查问题"
            echo "=============================================="
        }
    }
}
