pipeline {
    agent any

    environment {
        APP_SERVER  = "10.114.0.3"
        APP_USER    = "jenkins-deploy"
        APP_DIR     = "/opt/juice-shop"
        REPO_URL    = "https://github.com/azmaharramov/juice-shop.git"
        BRANCH      = "main"
        SSH_KEY     = "home/azar/.ssh/id_ed25519"
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Pull & Deploy') {
            steps {
                echo "App serverinə qoşulub kod güncəllənir..."
                sh """
                    ssh -o StrictHostKeyChecking=no \
                        -i ${SSH_KEY} \
                        ${APP_USER}@${APP_SERVER} '
                        set -e

                        if [ ! -d "${APP_DIR}/.git" ]; then
                            echo "İlk dəfə clone edilir..."
                            git clone ${REPO_URL} ${APP_DIR}
                        else
                            echo "Mövcud repo güncəllənir..."
                            cd ${APP_DIR}
                            git fetch origin
                            git reset --hard origin/${BRANCH}
                        fi

                        cd ${APP_DIR}

                        echo "npm install icra edilir..."
                        npm install --omit=dev

                        echo "Köhnə proses dayandırılır..."
                        pm2 stop juice-shop 2>/dev/null || true
                        pm2 delete juice-shop 2>/dev/null || true

                        echo "Tətbiq işə salınır..."
                        pm2 start app.js --name juice-shop
                        pm2 save
                    '
                """
            }
        }

        stage('Health Check') {
            steps {
                echo "Tətbiqin ayağa qalxması gözlənilir..."
                sh """
                    ssh -o StrictHostKeyChecking=no \
                        -i ${SSH_KEY} \
                        ${APP_USER}@${APP_SERVER} '
                        attempt=0
                        max=15
                        until curl -sf http://localhost:3000 > /dev/null; do
                            attempt=\$((attempt+1))
                            if [ \$attempt -ge \$max ]; then
                                echo "Health check uğursuz oldu!"
                                pm2 logs juice-shop --lines 20 --nostream
                                exit 1
                            fi
                            echo "Gözlənilir... (\$attempt/\$max)"
                            sleep 5
                        done
                        echo "Tətbiq uğurla işə düşdü!"
                    '
                """
            }
        }
    }

    post {
        success {
            echo "✅ Build #${BUILD_NUMBER} uğurla deploy edildi!"
        }
        failure {
            echo "❌ Pipeline uğursuz oldu! Build #${BUILD_NUMBER}"
        }
    }
}