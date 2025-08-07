pipeline {
    agent any

    tools {
        nodejs 'NodeJS 14.9.0'  // 指定 Jenkins 全局工具配置中的 Node.js 版本
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
                pnpm install
                pnpm build
                """
                echo '--------------构建完成!--------------'
            }
        }

        stage('部署服务') {
            steps {
                // 需要使用流水线语法生成 先发送dist文件，再发送Dockerfile文件，并且执行命令
                sshPublisher(publishers: [sshPublisherDesc(configName: 'DockerServer-test', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: 'webApps/myVue3Web', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'dist/**/*'), sshTransfer(cleanRemote: false, excludes: '', execCommand:
                    '''cd webApps/myVue3Web
                    docker build -t my-vue3-web .
                    docker run -d --name my-vue3-web -p 80:80 -v /root/webApps/myVue3Web/nginx-conf/nginx.conf:/etc/nginx/conf.d/default.conf my-vue3-web''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: 'webApps/myVue3Web', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'Dockerfile')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
                echo '--------------服务部署完成!--------------'
            }
        }
    }
}

