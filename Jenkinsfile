pipeline {
    agent any

    tools {
        nodejs 'NodeJS14.9.0'  // 指定 Jenkins 全局工具配置中的 Node.js 版本
    }
    stages {
        stage('拉取代码') {
            steps {
                // 使用流水线语法生成
                git credentialsId: 'ad68df2e-173e-48dc-9094-64d6a3d08150', url: 'git@github.com:15072601235/pokeceshi.git'
                echo '--------------代码拉取成功!--------------'
            }
        }

        stage('重置环境') {
            steps {
                // 需要使用流水线语法生成
                sshPublisher(publishers: [sshPublisherDesc(configName: 'DockerServer-test', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand:
                '''cd webApps/myVue3Web
                    rm -rf dist
                    rm -rf Dockerfile
                    docker stop my-vue3-web
                    docker rm -f my-vue3-web
                    docker rmi my-vue3-web''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
                echo '--------------环境重置完成!--------------'
            }
        }


        stage('构建代码') {
            steps {
                sh """
                npm install
                npm build
                """
                echo '--------------构建完成!--------------'
            }
        }

        stage('部署服务') {
                steps {
                    sshPublisher(publishers: [
                        sshPublisherDesc(
                            configName: 'DockerServer-test',
                            transfers: [
                                sshTransfer(
                                    execCommand: """
                                    # 确保目录可写
                                    mkdir -p /home/jenkins/webApps/myVue3Web/dist
                                    chown -R jenkins:docker /home/jenkins/webApps
                                    chmod -R 775 /home/jenkins/webApps
                                    """
                                ),
                                sshTransfer(
                                    sourceFiles: 'dist/**',
                                    remoteDirectory: 'webApps/myVue3Web',
                                    //仅传输不执行命令
                                    execCommand: ""  
                                ),
                                sshTransfer(
                                    sourceFiles: 'Dockerfile',
                                    remoteDirectory: 'webApps/myVue3Web',
                                    execCommand: """
                                    # 使用完整路径执行
                                    cd /home/jenkins/webApps/myVue3Web && \
                            /usr/bin/docker build . -t my-vue3-web && \
                            /usr/bin/docker run -d -p 80:80 my-vue3-web
                                    """
                                )
                            ],
                            //启用伪终端
                            usePty: true  
                        )
                    ])
                }
        }
    }

  post {
    always {
        sh '''
            echo "=== 部署验证 ==="
            ssh jenkins@目标服务器 "
                echo '最后部署状态:';
                docker ps -a | grep my-vue3-web;
                echo '目录内容:';
                ls -la /home/jenkins/webApps/myVue3Web;
                echo 'Docker日志:';
                docker logs my-vue3-web || echo '容器不存在'
            "
        '''
    }
}
}
