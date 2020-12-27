def BIN_CATALOG = ''
def ACC_PROPERTIES = ''
def ACC_BASE = ''
def ACC_USER = ''
def BSL_LS_PROPERTIES = ''
def CURRENT_CATALOG = ''
def TEMP_CATALOG = ''
def PROJECT_NAME_EDT = ''
def PLATFORM_1C = ''
def PROJECT_KEY
def EDT_VALIDATION_RESULT = ''
def GENERIC_ISSUE_JSON = ''
def SRC = ''
def PROJECT_URL = ''
def NIGHT_BUILD_CATALOG = ''

pipeline {

    parameters {
        string(defaultValue: "${env.PROJECT_NAME}", description: '* Имя проекта. Одинаковое для EDT, проекта в АПК и в сонаре. Обычно совпадает с именем конфигурации.', name: 'PROJECT_NAME')
        string(defaultValue: "${env.git_repo_url}", description: '* URL к гит-репозиторию, который необходимо проверить.', name: 'git_repo_url')
        string(defaultValue: "${env.git_repo_branch}", description: 'Ветка репозитория, которую необходимо проверить. По умолчанию master', name: 'git_repo_branch')
        string(defaultValue: "${env.sonar_catalog}", description: 'Каталог сонара, в котором лежит все, что нужно. По умолчанию C:/Sonar/', name: 'sonar_catalog')
        string(defaultValue: "${env.PROPERTIES_CATALOG}", description: 'Каталог с настройками acc.properties, bsl-language-server.conf и sonar-project.properties. По умолчанию ./Sonar', name: 'PROPERTIES_CATALOG')
        booleanParam(defaultValue: env.ACC_check== null ? true : env.ACC_check, description: 'Выполнять ли проверку АПК. Если нет, то будут получены существующие результаты. По умолчанию: true', name: 'ACC_check')
        booleanParam(defaultValue: env.ACC_recreateProject== null ? false : env.ACC_recreateProject, description: 'Пересоздать проект в АПК. Все данные о проекте будут собраны заново. По умолчанию: false', name: 'ACC_recreateProject')
        string(defaultValue: "${env.STEBI_SETTINGS}", description: 'Файл настроек для переопределения замечаний. Для файла из репо проекта должен начинатся с папки Repo, например .Repo/Sonar/settings.json. По умолчанию ./Sonar/settings.json', name: 'STEBI_SETTINGS')
        string(defaultValue: "${env.jenkinsAgent}", description: 'Нода дженкинса, на которой запускать пайплайн. По умолчанию master', name: 'jenkinsAgent')
        string(defaultValue: "${env.EDT_VERSION}", description: 'Используемая версия EDT. По умолчанию 1.13.0', name: 'EDT_VERSION')
        string(defaultValue: "${env.platfor1c}", description: 'Используемая версия платформы 1С. По умолчанию 8.3.17.1851', name: 'platfor1c')
        string(defaultValue: "${env.perf_catalog}", description: 'Путь к каталогу с замерами производительности, на основе которых будет рассчитано покрытие. Если пусто - покрытие не считается.', name: 'perf_catalog')
        string(defaultValue: "${env.git_credentials_Id}", description: 'ID Credentials для получения изменений из гит-репозитория', name: 'git_credentials_Id')
        string(defaultValue: "${env.telegram_channel}", description: 'Канал в рокет-чате для отправки уведомлений', name: 'telegram_channel')
    }
    agent {
        label "${(env.jenkinsAgent == null || env.jenkinsAgent == 'null') ? "master" : env.jenkinsAgent}"
    }
    options {
        timeout(time: 8, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }
    stages {
        stage("Инициализация переменных") {
            steps {
                script {

                    telegram_channel = telegram_channel == null || telegram_channel == 'null' ? '' : telegram_channel
                    platfor1c = platfor1c.isEmpty() ? "8.3.17.1851" : platfor1c

                    CURRENT_CATALOG = pwd()
                    TEMP_CATALOG = "${CURRENT_CATALOG}\\temp"
                    EDT_VALIDATION_RESULT = "${TEMP_CATALOG}\\edt-result.csv"
                    PROJECT_XML_CATALOG = "${CURRENT_CATALOG}\\XML"
                    PROJECT_IB_CATALOG = "${CURRENT_CATALOG}\\IB"
                    CURRENT_CATALOG = "${CURRENT_CATALOG}\\Repo"
                    PROJECT_NAME_EDT = "${CURRENT_CATALOG}\\${PROJECT_NAME}"
                    PLATFORM_1C = "C:\\Program Files\\1cv8\\${platfor1c}\\bin\\1cv8.exe"
                    NIGHT_BUILD_CATALOG = "O:\\Отдел 1С\\Управление карьером\\ПОСТАВКИ\\ГДП ОУ\\_NightBuild\\1Cv8.cf"

                    if (!telegram_channel.isEmpty() ) {
                        telegramSend(message: "Jenkins build started: [${env.JOB_NAME} ${env.BUILD_NUMBER}](${env.JOB_URL})", chatId: -1001247906636)
                    } else {telegramSend(message: "Jenkins build started 2: [${env.JOB_NAME} ${env.BUILD_NUMBER}](${env.JOB_URL})", chatId: -1001247906636)}

                    dir('IB') {
                        deleteDir()
                    }
                    dir('XML') {
                        deleteDir()
                    } 
                }
            }
        }
        stage('Checkout') {
            steps {
                script {
                    dir('Repo') {
                        checkout([$class: 'GitSCM',
                        branches: [[name: "*/${git_repo_branch}"]],
                        doGenerateSubmoduleConfigurations: false,
                        extensions: [[$class: 'CheckoutOption', timeout: 60], [$class: 'CloneOption', depth: 0, noTags: true, reference: '', shallow: false, timeout: 60]],
                        submoduleCfg: [],
                        userRemoteConfigs: [[credentialsId: git_credentials_Id, url: git_repo_url]]])
                    }
                }
            }
        }
        stage('EDT') {
            steps {
                script {
                    if (fileExists("${EDT_VALIDATION_RESULT}")) {
                        cmd("@DEL \"${EDT_VALIDATION_RESULT}\"")
                    }
                    /*cmd("""
                    @set RING_OPTS=-Dfile.encoding=UTF-8 -Dosgi.nl=ru
                    ring edt@${EDT_VERSION} workspace validate --workspace-location \"${TEMP_CATALOG}\" --file \"${EDT_VALIDATION_RESULT}\" --project-list \"${PROJECT_NAME_EDT}\"
                    """)*/
                }
            }
        }
        stage('EDT to XML') {
            steps {
                script {
                    cmd("""
                    @set RING_OPTS=-Dfile.encoding=UTF-8 -Dosgi.nl=ru
                    ring edt@${EDT_VERSION} workspace export --workspace-location \"${TEMP_CATALOG}\" --configuration-files \"${PROJECT_XML_CATALOG}\" --project \"${PROJECT_NAME_EDT}\"
                    """)
                }
            }
        }

        stage('Создание файла поставки') {
            steps {
                script {
                    cmd("\"${PLATFORM_1C}\" CREATEINFOBASE File=\"${PROJECT_IB_CATALOG}\" /DumpResult \"${TEMP_CATALOG}\\log.txt\"")
                    cmd("\"${PLATFORM_1C}\" DESIGNER /F \"${PROJECT_IB_CATALOG}\" /LoadConfigFromFiles \"${PROJECT_XML_CATALOG}\" /UpdateDBCfg /DisableStartupDialogs /DumpResult \"${TEMP_CATALOG}\\log.txt\"")
                    cmd("\"${PLATFORM_1C}\" DESIGNER /F \"${PROJECT_IB_CATALOG}\" /CreateDistributionFiles -cffile \"${NIGHT_BUILD_CATALOG}\" /DisableStartupDialogs /DumpResult \"${TEMP_CATALOG}\\log.txt\"")
                

                    if (!telegram_channel.isEmpty() ) {
                        telegramSend(message: "Jenkins build started: [${env.JOB_NAME} ${env.BUILD_NUMBER}](${env.JOB_URL}) STATUS: [${currentBuild.result}]", chatId: -1001247906636)
                    }
                }
            }
        }
    }
    post {
        always {
            script {
                dir ('temp') {
                    deleteDir()
                }
            }
        }
    }
}

def cmd(command) {
    // при запуске Jenkins не в режиме UTF-8 нужно написать chcp 1251 вместо chcp 65001
    if (isUnix()) { sh "${command}" } else { bat "chcp 65001\n${command}" }
}