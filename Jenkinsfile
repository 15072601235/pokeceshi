pipeline {
    agent any

    tools {
        nodejs 'NodeJS14.9.0'
    }

    environment {
        // 使用非root路径
        DEPLOY_PATH = '/home/jenkins/webApps/myVue3Web'
    }

    stages {
        stage('拉取代码') {
            steps {
                git credentialsId: 'ad68df2e-173e-48dc-9094-64d6a3d08150', 
                    url: 'git@github.com:15072601235/pokeceshi.git'
                echo '--------------代码拉取成功!--------------'
            }
        }

        stage('重置环境') {
            steps {
                script {
                    // 检查SSH配置是否存在
                    def sshConfig = Jenkins.instance.getDescriptor('jenkins.plugins.publish_over_ssh.BapSshPublisherPlugin')
                    if(!sshConfig.getConfigurationByName('DockerServer-test')) {
                        error("SSH配置 DockerServer-test 不存在，请先在系统设置中配置")
                    }

                    sshPublisher(publishers: [
                        sshPublisherDesc(
                            configName: 'DockerServer-test', 
                            transfers: [
                                sshTransfer(
                                    execCommand: """
                                    mkdir -p ${DEPLOY_PATH}
                                    cd ${DEPLOY_PATH}
                                    docker stop my-vue3-web || true
                                    docker rm -f my-vue3-web || true
                                    docker rmi my-vue3-web || true
                                    rm -rf dist Dockerfile
                                    """
                                )
                            ]
                        )
                    ])
                }
                echo '--------------环境重置完成!--------------'
            }
        }

        stage('构建代码') {
            steps {
                sh """
                npm install
                npm run build  # 修正此处
                """
                echo '--------------构建完成!--------------'
            }
        }

        stage('部署服务') {
            steps {
                script {
                    // 确保Dockerfile存在
                    if(!fileExists('Dockerfile')) {
                        writeFile file: 'Dockerfile', text: """
                        FROM nginx:alpine
                        COPY dist/ /usr/share/nginx/html
                        COPY nginx-conf/nginx.conf /etc/nginx/conf.d/default.conf
                        """
                    }

                    sshPublisher(publishers: [
                        sshPublisherDesc(
                            configName: 'DockerServer-test',
                            transfers: [
                                // 传输dist文件
                                sshTransfer(
                                    sourceFiles: 'dist/**',
                                    remoteDirectory: "${DEPLOY_PATH}",
                                    execCommand: ""
                                ),
                                // 传输Dockerfile并部署
                                sshTransfer(
                                    sourceFiles: 'Dockerfile,nginx-conf/nginx.conf',
                                    remoteDirectory: "${DEPLOY_PATH}",
                                    execCommand: """
                                    cd ${DEPLOY_PATH}
                                    docker build -t my-vue3-web .
                                    docker run -d --name my-vue3-web -p 80:80 \\
                                        -v ${DEPLOY_PATH}/nginx-conf/nginx.conf:/etc/nginx/conf.d/default.conf \\
                                        my-vue3-web
                                    """
                                )
                            ]
                        )
                    ])
                }
                echo '--------------服务部署完成!--------------'
            }
        }
    }
}
