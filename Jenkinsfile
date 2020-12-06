pipeline {
  agent any
  stages {
     stage('Verify Branch') {
       steps {
         echo "$GIT_BRANCH"
       }
     }
    stage('Pull Changes') {
      steps {
        powershell(script: "git pull")
      }
    }
    stage('Run Unit Tests') {
      steps {
        powershell(script: """ 
          cd Server
          dotnet test
          cd ..
        """)
      }
    }
	stage('Docker Build Development') {
       when { branch 'development' }
      steps {
        powershell(script: 'docker-compose build')
        powershell(script: 'docker build -t sekul/carrentalsystem-user-client-development --build-arg configuration=development ./Client')
        powershell(script: 'docker images -a')
      }
    }
     stage('Docker Build Production') {
       when { branch 'main' }
      steps {
        powershell(script: 'docker-compose build')
        powershell(script: 'docker build -t sekul/carrentalsystem-user-client-production --build-arg configuration=production ./Client')
        powershell(script: 'docker images -a')
      }
    }
    stage('Run Test Application') {
      steps {
        powershell(script: 'docker-compose up -d')    
      }
    }
    stage('Run Integration Tests') {
      steps {
        powershell(script: './Tests/ContainerTests.ps1') 
      }
    }
    stage('Stop Test Application') {
      steps {
        powershell(script: 'docker-compose down') 
        // powershell(script: 'docker volumes prune -f')   		
      }
      post {
	    success {
	      echo "Build successfull! You should deploy! :)"
	    }
	    failure {
	      echo "Build failed! You should receive an e-mail! :("
	    }
      }
    }
    stage('Push Images') {
      when { branch 'main' }  
      steps {
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'Docker Hub') {
            def image = docker.image("sekul/carrentalsystem-identity-service")
            image.push("1.0.${env.BUILD_ID}")
            image.push('latest')
          }
		  docker.withRegistry('https://index.docker.io/v1/', 'Docker Hub') {
            def image = docker.image("sekul/carrentalsystem-dealers-service")
            image.push("1.0.${env.BUILD_ID}")
            image.push('latest')
          }
		  docker.withRegistry('https://index.docker.io/v1/', 'Docker Hub') {
            def image = docker.image("sekul/carrentalsystem-statistics-service")
            image.push("1.0.${env.BUILD_ID}")
            image.push('latest')
          }
		  docker.withRegistry('https://index.docker.io/v1/', 'Docker Hub') {
            def image = docker.image("sekul/carrentalsystem-notifications-service")
            image.push("1.0.${env.BUILD_ID}")
            image.push('latest')
          }
		  docker.withRegistry('https://index.docker.io/v1/', 'Docker Hub') {
            def image = docker.image("sekul/carrentalsystem-user-client")
            image.push("1.0.${env.BUILD_ID}")
            image.push('latest')
          }
		  docker.withRegistry('https://index.docker.io/v1/', 'Docker Hub') {
            def image = docker.image("sekul/carrentalsystem-admin-client")
            image.push("1.0.${env.BUILD_ID}")
            image.push('latest')
          }
		 docker.withRegistry('https://index.docker.io/v1/', 'Docker Hub') {
            def image = docker.image("sekul/carrentalsystem-watchdog-service")
            image.push("1.0.${env.BUILD_ID}")
            image.push('latest')
          }
        }
      }
    } 
    stage('Deploy Development') {
      when { branch 'development' }
      steps {
        withKubeConfig([credentialsId: 'DevelopmentServer', serverUrl: 'https://34.72.169.70']) {
		       powershell(script: 'kubectl apply -f ./.k8s/.environment/development.yml') 
		       powershell(script: 'kubectl apply -f ./.k8s/databases') 
		       powershell(script: 'kubectl apply -f ./.k8s/event-bus') 
		       powershell(script: 'kubectl apply -f ./.k8s/web-services') 
           powershell(script: 'kubectl apply -f ./.k8s/clients') 
           powershell(script: 'kubectl set image deployments/user-client user-client=sekul/carrentalsystem-user-client-development:latest')
        }
      }
    }
  }
}
