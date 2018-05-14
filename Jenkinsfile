pipeline {
	agent none

	environment {
		MAJOR_VERSION = 1
	}
 
	stages {
	    stage('Unit Tests') {
	      agent {
	        label 'apache'
	      }
	      
	      steps {

	         sh 'ant -f test.xml -v'
	         junit 'reports/result.xml'

	      }

	    }

	    stage('build') {
	      agent {
	        label 'apache'
	      }

	      steps {
	        sh 'ant -f build.xml -v'

	      }
	      post { 
	        success {
	          archiveArtifacts artifacts:'dist/*jar, fingerprint:true'
	        }
	      }
	    }

	    stage('deploy') {
	      agent{
	        label 'apache'
	      }
	      
	      steps {
	        sh "if [ ! -d '/var/www/html/rectangles/all/${env.BRANCH_NAME}' ]; then mkdir -p /var/www/html/rectangles/all/${env.BRANCH_NAME}/green; fi"

	        sh "cp -rp dist/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}/"
	      }
	    }

	    stage("Testing the Built Jar") {
	      agent {
	        label 'apache'
	      }
	      steps {
	        sh "wget http://localhost/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
	        sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 5 6"
		  }

		}

		stage('Promote to Green') {
                  agent {
                    label 'apache'
                  }
                  when {
                    branch 'master'
                  }
                  steps{
                    sh "if [ ! -d '/var/www/html/rectangles/all/green' ]; then mkdir -p /var/www/html/rectangles/all/green; fi"
                    sh "cp -rp /var/www/html/rectangles/all/{env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/green/"
                  }
        }

        stage('Promote Development to Master Branch'){
	        agent {
	          label 'apache'
            }
	  
            when {
	           branch 'development'
	        }
	  
            steps {
		      echo "Stashing any local changes"
		      sh 'git stash'
		      echo "Checking out Development Branch"
		      sh 'git checkout development'
              echo "Making sure the development branch is upto date"
		      sh 'git pull origin'
		      echo "Checking Out Master Branch"
		      sh 'git checkout master'
		      echo "Merging Development into Master Branch"
		      sh 'git merge development'
		      echo "Pushing to Origin Master"
		      echo "git push origin master"
            }
        }
	   

    }
    post {
        success {
            emailext(
              subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Development Promoted to Master",
              body: """<p>'${env.JOB_NAME} [${env.BUILD_NUMBER}]' Development Promoted to Master":</p>
              <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
              to: "krahulchowdary@hotmail.com"
            )
         }
    } 
}

