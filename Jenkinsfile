pipeline {
  // Где будет выполняться пайплайн. 'any' — любой Jenkins-агент.
  agent any

  // Общие опции конвейера
  options {
    // Добавляет метки времени в лог сборки
    timestamps()
    // Обрезает историю сборок, хранит только последние 20
    buildDiscarder(logRotator(numToKeepStr: '20'))
    // Запрещает параллельные сборки одного job (полезно для локального демо)
    disableConcurrentBuilds()
  }

  // Переменные окружения для стадий
  environment {
    // Опции Maven: падать при ошибках тестов
    MAVEN_OPTS = '-Dmaven.test.failure.ignore=false'
  }

  // Триггеры запуска конвейера
  triggers {
    // Периодически проверяет SCM (репозиторий) на изменения каждые ~2 минуты
    pollSCM('H/2 * * * *')
    // Для продакшена лучше настраивать webhooks, но локально pollSCM удобнее.
  }

  stages {
//Вставка
stage('Fix Git Ownership') {
      steps {
        bat 'git config --global --add safe.directory E:/repos/java-maven-ci-demo.git'
      }
    }
//Вставка конец

    stage('Checkout') {
      steps {
        // Извлечение кода из Git. Для локального демо используем file:// путь к bare-репозиторию.
        // Замените путь на ваш локальный (Windows/Linux).
        checkout([$class: 'GitSCM',
          branches: [[name: '*/${BRANCH_NAME ?: "develop"}']],
          userRemoteConfigs: [[url: 'file:///E:/repos/java-maven-ci-demo.git']]
        ])
        // Примечание: BRANCH_NAME Jenkins подставляет для мультиветочных пайплайнов.
        // Для простого pipeline можно явно указать '*/main' или '*/develop'.
      }
    }

    stage('Build') {
      steps {
        // Проверка версии Maven
        bat 'mvn -v'
        // Компиляция проекта. Параметры:
        // -B: batch mode (без интерактивных вопросов)
        // -U: принудительно обновить снапшоты (на случай зависимостей)
        bat 'mvn -B -U clean compile'
      }
    }

    stage('Test') {
      steps {
        // Запуск модульных тестов (JUnit 5) и сбор покрытия JaCoCo
        bat 'mvn -B test'
      }
      post {
        // Действия после стадии Test
        always {
          // Публикация результатов JUnit (Surefire создаёт XML-отчёты в target/surefire-reports)
          junit '**/target/surefire-reports/*.xml'

          // Публикация HTML-отчёта JaCoCo (генерируется в target/site/jacoco/index.html)
          // Требуется плагин HTML Publibater в Jenkins.
          publibatHTML([reportDir: 'target/site/jacoco',
                       reportFiles: 'index.html',
                       reportName: 'JaCoCo Coverage'])
        }
      }
    }

    stage('Package') {
      steps {
        // Сборка исполняемого JAR. Артефакт будет в target/java-maven-ci-demo-1.0.0.jar
        bat 'mvn -B package'
      }
      post {
        success {
          // Архивирование собранных артефактов в Jenkins (видны в UI job)
          archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
      }
    }

    stage('Quality gates') {
      // Стадия качества включается только для веток develop и main
      when {
        anyOf {
          branch 'develop'
          branch 'main'
        }
      }
      steps {
        // Здесь можно подключить статический анализ: SpotBugs, Checkstyle, SonarQube.
        echo 'Quality checks placeholder: линтеры, статанализ, SonarQube и т.д.'
      }
    }

    stage('Deploy (local)') {
      // Условный «деплой»: выполняем только для main
      when {
        branch 'main'
      }
      steps {
        // Запуск приложения в фоне (nohup) и вывод логов в app.log
        // В учебных целях это имитация CD (continuous deployment).
        bat 'nohup java -jar target/java-maven-ci-demo-1.0.0.jar > app.log 2>&1 &'
      }
    }
  }

  // Глобальные действия после конвейера
  post {
    success {
      echo "Build successful for ${env.BRANCH_NAME}"
    }
    failure {
      echo "Build failed for ${env.BRANCH_NAME}"
    }
  }
} 
