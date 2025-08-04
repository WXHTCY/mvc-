pipeline {
    agent any
    tools {
        maven 'ceshi1'
        jdk 'JDK'
    }
    // 增加环境变量配置，优化网络稳定性
    environment {
        // Git 网络优化配置，减少拉取失败概率
        GIT_NETWORK_OPTS = 'git config --global http.postBuffer 524288000 && git config --global http.sslVerify false'
    }
    stages {
        stage('拉取代码') {
            steps {
                echo "从 GitHub 拉取 main 分支代码..."
                script {
                    // 预配置 Git 网络参数，解决 HTTPS 连接问题
                    bat "${env.GIT_NETWORK_OPTS}"
                    // 使用存储的凭据拉取代码，避免认证失败
                    git url: 'https://github.com/msg-555/mvc-.git', 
                        branch: 'main',
                        credentialsId: 'b22d5859-a10f-4cfb-bf76-9460f4bf46a3'  // 替换为你的凭据ID
                }
            }
        }
        
        stage('构建项目') {
            steps {
                echo "使用 Maven 构建 WAR 包..."
                bat 'mvn clean package -Dmaven.test.skip=true'
                
                // 增加 WAR 包存在性检查，提前发现构建问题
                bat '''
                    if not exist "target/MVC.war" (
                        echo "ERROR: WAR 包未生成！请检查 Maven 配置或代码错误"
                        exit 1
                    ) else (
                        echo "WAR 包生成成功：target/MVC.war"
                        dir "target\\MVC.war"  // 显示 WAR 包详细信息
                    )
                '''
            }
        }
        
        stage('运行测试') {
            steps {
                echo "执行单元测试..."
                bat 'mvn test'
            }
        }
        
        stage('部署到服务器') {
            steps {
                echo "部署 WAR 包到服务器 Tomcat 目录..."
                sshPublisher(publishers: [
                    sshPublisherDesc(
                        configName: 'my-server',  // 确保与 Jenkins SSH 配置名称一致
                        transfers: [
                            sshTransfer(
                                sourceFiles: 'target/MVC.war',
                                remoteDirectory: '/root/apache-tomcat-10.1.19/webapps',
                                cleanRemote: false,
                                flatten: true,
                                // 关键优化：使用 sh -c 包裹命令 + 详细日志 + 容错处理
                                execCommand: '''sh -c '
                                    # 开启命令追踪模式，输出每一步执行的命令
                                    set -x
                                    
                                    # 1. 服务器环境检查（核心排查点）
                                    echo "=== 服务器基础信息 ==="
                                    whoami  # 确认执行用户（应为 root 或有足够权限的用户）
                                    pwd     # 确认当前目录
                                    df -h   # 检查磁盘空间是否充足
                                    
                                    # 2. 验证 Tomcat 路径正确性
                                    echo "=== Tomcat 目录检查 ==="
                                    if [ ! -d "/root/apache-tomcat-10.1.19" ]; then
                                        echo "ERROR: Tomcat 目录不存在！请检查路径是否正确"
                                        exit 1
                                    fi
                                    ls -ld /root/apache-tomcat-10.1.19/webapps  # 确认部署目录存在
                                    
                                    # 3. 验证 WAR 包上传结果
                                    echo "=== WAR 包上传检查 ==="
                                    if [ ! -f "/root/apache-tomcat-10.1.19/webapps/MVC.war" ]; then
                                        echo "ERROR: WAR 包未上传成功！检查传输配置或目录权限"
                                        exit 1
                                    fi
                                    ls -l /root/apache-tomcat-10.1.19/webapps/MVC.war  # 确认文件大小正常
                                    
                                    # 4. 停止 Tomcat 服务（增强容错）
                                    echo "=== 停止 Tomcat 服务 ==="
                                    if [ -f "/root/apache-tomcat-10.1.19/bin/shutdown.sh" ]; then
                                        /root/apache-tomcat-10.1.19/bin/shutdown.sh
                                        shutdown_exit=$?
                                        # 若正常停止失败，强制终止进程
                                        if [ $shutdown_exit -ne 0 ]; then
                                            echo "WARNING: Tomcat 正常停止失败，尝试强制终止"
                                            ps -ef | grep tomcat | grep -v grep | awk '{print $2}' | xargs kill -9 2>/dev/null
                                        fi
                                    else
                                        echo "ERROR: Tomcat 停止脚本不存在！"
                                        exit 1
                                    fi
                                    sleep 5  # 等待进程终止
                                    
                                    # 5. 清理旧部署文件
                                    echo "=== 清理旧部署文件 ==="
                                    rm -rf /root/apache-tomcat-10.1.19/webapps/MVC*
                                    if [ $? -ne 0 ]; then
                                        echo "ERROR: 清理旧文件失败！可能文件被占用或权限不足"
                                        exit 1
                                    fi
                                    
                                    # 6. 再次确认 WAR 包存在（防止清理时误删）
                                    if [ ! -f "/root/apache-tomcat-10.1.19/webapps/MVC.war" ]; then
                                        echo "ERROR: WAR 包丢失！清理步骤可能误删文件"
                                        exit 1
                                    fi
                                    
                                    # 7. 启动 Tomcat 服务
                                    echo "=== 启动 Tomcat 服务 ==="
                                    if [ -f "/root/apache-tomcat-10.1.19/bin/startup.sh" ]; then
                                        /root/apache-tomcat-10.1.19/bin/startup.sh
                                        if [ $? -ne 0 ]; then
                                            echo "ERROR: Tomcat 启动脚本执行失败！"
                                            exit 1
                                        fi
                                    else
                                        echo "ERROR: Tomcat 启动脚本不存在！"
                                        exit 1
                                    fi
                                    
                                    # 8. 验证部署结果
                                    echo "=== 部署结果验证 ==="
                                    sleep 10  # 等待 Tomcat 初始化
                                    ps -ef | grep tomcat | grep -v grep  # 确认 Tomcat 进程存在
                                    ls -l /root/apache-tomcat-10.1.19/webapps  # 确认应用目录已解压
                                ' '''
                            )
                        ],
                        verbose: true,  // 输出 SSH 传输详细日志
                        timeout: 180000  // 延长超时时间至 3 分钟，适应慢网络
                    )
                ])
            }
        }
    }
    
    post {
        success {
            echo "=============================================="
            echo "🎉 构建部署成功！"
            echo "访问地址：http://111.230.94.55:8080/MVC"
            echo "=============================================="
        }
        failure {
            echo "=============================================="
            echo "❌ 构建或部署失败，请重点查看以下日志："
            echo "1. 服务器环境检查步骤的输出（用户/路径/权限）"
            echo "2. 标记为 ERROR 的关键错误信息"
            echo "=============================================="
        }
    }
}
