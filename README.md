- 👋 Hi, I’m @xuehongle2
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
xuehongle2/xuehongle2 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
node('testhan') {
    stage('Clone') {
        echo "1.Clone Stage"
        git url: "https://github.com/luckylucky421/jenkins-sample.git"
        script {
            build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
        }
    }
    stage('Test') {
      echo "2.Test Stage"

    }
    stage('Build') {
        echo "3.Build Docker Image Stage"
        sh "docker build -t xuehongle2/jenkins-demo:${build_tag} ."
    }
    stage('Push') {
        echo "4.Push Docker Image Stage"
        withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
            sh "docker login -u ${dockerHubUser} -p ${dockerHubPassword}"
            sh "docker push xuehongle2/jenkins-demo:${build_tag}"
        }
    }
    stage('Deploy to dev') {
        echo "5. Deploy DEV"
		sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s-dev.yaml"
        sh "sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' k8s-dev.yaml"
//        sh "bash running-devlopment.sh"
        sh "kubectl apply -f k8s-dev.yaml  --validate=false"
	}	
	stage('Promote to qa') {	
		def userInput = input(
            id: 'userInput',

            message: 'Promote to qa?',
            parameters: [
                [
                    $class: 'ChoiceParameterDefinition',
                    choices: "YES\nNO",
                    name: 'Env'
                ]
            ]
        )
        echo "This is a deploy step to ${userInput}"
        if (userInput == "YES") {
            sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s-qa.yaml"
            sh "sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' k8s-qa.yaml"
//            sh "bash running-qa.sh"
            sh "kubectl apply -f k8s-qa.yaml --validate=false"
            sh "sleep 6"
            sh "kubectl get pods -n qatest"
        } else {
            //exit
        }
    }
	stage('Promote to pro') {	
		def userInput = input(

            id: 'userInput',
            message: 'Promote to pro?',
            parameters: [
                [
                    $class: 'ChoiceParameterDefinition',
                    choices: "YES\nNO",
                    name: 'Env'
                ]
            ]
        )
        echo "This is a deploy step to ${userInput}"
        if (userInput == "YES") {
            sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s-prod.yaml"
            sh "sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' k8s-prod.yaml"
//            sh "bash running-production.sh"
            sh "cat k8s-prod.yaml"
            sh "kubectl apply -f k8s-prod.yaml --record --validate=false"
        }
    }
}
