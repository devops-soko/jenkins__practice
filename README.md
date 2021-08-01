# jenkins


## 1. Related Concept 
### 1) Issues before CI/CD
Irregular Commit -> Integration Problem -> Test Delay -> Bug Increase

### 2) What are CI and CD?
- Continuous Integration : Continuously integrate code, settings, etc.
- Continuous Delivery : Continuously test changes and release them to the public repository (ex. github) 
- Continuous Deployment : Continuously deploy changes to customers or users

### 3) Example of CI/CD Pipeline
Engineer -> Code -> Build -> Test -> Deployment -> User


## 2. Basic Concept
### 1) What is Jenkins
Jenkins is an open-source automation tool for CI/CD purposes

### 2) Features of Jenkins 
- Jenkins is written in Java 
- Jenkins has a lot of plugins (they help us to implement varius functions)

### 3) Istall Jenkins
- Jenkins installation files
  - LTS(Long Term Support) Release : Version released every 12 weeks
  - Weekly Release : Version released every week

- How to Install Jenkins?
```
#install jenkins
$ sudo yum -y update
$ sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins.io/redhat/jenkins.repo && sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key

#install java
$ sudo yum install java-1.8.0-openjdk
$ sudo alternatives --config java

#start jenkins
$ sudo service jenkins start

#check jenkins port
$ sudo  cat /etc/sysconfig/jenkins  | grep JENKINS_PORT

# you can access jenkins by http://ip:prot/
```



## 3. Jenkins Syntax (Reference : https://www.jenkins.io/doc/book/pipeline/syntax/)

### 1) pipeline 
- Form : pipeline { }
- Location : The Top level block 
- Feature : it consists of Sections, Directives, Steps, or assignment statements  

### 2) Sections
#### Section 1 : agent
- Function : select agent which runs pipeline or stages 
- Location

Agent section must be defined at the top-level inside the pipeline block -> refer to ex1
		
or it can be defined at stage-level(optional)  ->refer to ex2
                

- parameter (There are more parameters. If you want to get info about them then please refer to https://www.jenkins.io/doc/book/pipeline/syntax/#agent) 
	- any : use any available agent to execute the pipeline or stage
	- none : global agent won't be defined-> agent needs to be defined at each stages 
	- label ' ' : use agent with provided label to execute the pipeline or stage
	- node{ label ' '  } :  same with agent{label ' ' } but can use options
```
agent{
	node{
		label ''		// use agent with provided label
		customWorkspace ''	// config workspace location
	}
}
```

- Example

ex1.
```
pipeline{
	agent {

	}
	stages{
		stage{

		}
	}
}

```

ex2.
```
pipeline{
    agent none
    stages{
	stage{
	    agent{

	    }
	    steps{

	    }
	}


	stage{
	    agent{

	    }
	    steps{

	    }
	}
    }
}
```
        
#### Section 2 : stages
- Function: Define pipeline's tasks
- Feature : it consists of one or more stage directives
- Example
```
pipeline{
    agent {

    }
    stages{
	    stage('test'){
	    }
    }
}
```

#### Section 2 : Steps
- Function : Define a series of one or more steps to be executed in a given stage directive.

- Example
```
pipeline{
    agent {

    }
    stages{
	stage('test'){
	    steps{
		echo 'hello'
	    }
	}
    }
}
```

#### Section 4 : post
- Function : define one or more additional steps after the completion of pipelien or stage
- Location : after stages in pipeline block or after steps in stage block 
- Conditions
	- always{} : Run the steps in post regardless of the Pipeline or stage’s status
	- failure{} : Only run the steps in post if pipeline or stage's run status is 'failed'
	- success{} : Only run the steps in post if pipeline or stage's run status is 'success'
	- unstable{} : Only run the steps in post if pipeline or stage's run status is 'unstabe'(test or code issue)
	- changed{} : Only run the steps in post if pipeline or stage's run status is different from previous run
	- fixed{} : Only run the steps in post if pipeline or stage's run status is 'success' and previous run status is 'failed' or 'unstable'
	- regression{} : Only run the steps in post if pipeline or stage's run status is 'failed', 'unstabe', or 'aborted' and previous run status is 'success' or 'unstable' 
	- aborted{} : Only run the steps in post if pipeline or stage's run status is 'aborted' (stopped)
	- unsuccessful{} : Only run the steps in post if pipeline or stage's run status is not 'success'
	- cleanup{} : Run the steps in post after every other post condition has been evaluated, regardless of the Pipeline or stage’s status

- Example 
```
pipeline{
	agent{

	}

	stages{
		stage('test'){
			steps{
				echo 'hello'
			}
		}
	}

	post{
    		always{
			echo 'finish'
    		}
	}
}
```



### 3) Directives
#### (a) Directives located between agent and stages
##### Directives 1 : environment
- Function : Define environment variables(key-value pairs) for the pipeline
- Location : inside the pipeline block between agent and stages or within stage directives
- Example (set Username and Password Credentials as enviornment variable)
```
pipeline {
	agent any
	environment { 
		TEST_URL = 'https://test.com'
	}
	stages {
	    stage('Example Username/Password') {
		environment {
		    TEST_CREDS = credentials('test-username-password')
		}
		steps {
		    sh 'echo "User is $TEST_CREDS_USR"'
		    sh 'echo "Password is $TEST_CREDS_PSW"'
		    sh 'curl -u $TEST_CREDS $TEST_URL'
		}
	    }
	}
}
```
cf. You can get Username and Password Credentials like below 

variable_name = credentials(credential_id)

ex. TEST_CREDS = credentials('test-username-password')

and can get username and password restectively like below

username : $TEST_CREDS_USR

password : $TEST_CREDS_PSW



##### Directives 2 :options (there are more options. If you want to know more about it then please refer to https://www.jenkins.io/doc/book/pipeline/syntax/#options)
- Function : configure Pipeline-specific options
- Location : inside the pipeline block between agent and stages
- Frequently used options
	- retry :  options { retry(3) } 	-> 	retry the pipeline if it is failed
	- timeout :  options { timeout(time: 1, unit: 'HOURS') }	->	Set a timeout period for the Pipeline run

- Example
```
pipeline {
    agent any
    options {
    	retry(5)
	timeout(time: 3, unit: 'HOURS') 
    }
    stages {
	stage('Test') {
	    steps {
		
	    }
	}
    }
}
```



##### Directives 3 :parameters
- Funtion : Make the pipeline get values from user 
- Option
	- string : parameters { string(name: 'TEST_STR', defaultValue: 'this is test', description: '') }
	- text :  parameters { text(name: 'TEST_TEXT', defaultValue: 'line1\nline2\nline3\n', description: '') }
	- booleanParam :parameters { booleanParam(name: 'TEST_BOOL', defaultValue: true, description: '') }
	- choice : parameters { choice(name: 'TEST_CHOICES', choices: ['choice1', 'choice2', 'choice3'], description: '') 
	- password :  parameters { password(name: 'TEST_PASSWORD', defaultValue: 'PWD', description: '') }

- Example
```
pipeline {
    agent any
    parameters {
	string(name: 'TEST_STR', defaultValue: '', description: 'input string')

	text(name: 'TEST_TEXT', defaultValue: '', description: 'input text')

	booleanParam(name: 'TEST_BOOL', defaultValue: true, description: 'true or false')

	choice(name: 'TEST_CHOICES', choices: ['val1', 'val2', 'val3'], description: 'Pick one')

	password(name: 'TEST_PASSWORD', defaultValue: 'PWD', description: 'Enter a password')
    }
    stages {
	stage('Example') {
	    steps {
		echo "string value: ${params.TEST_STR}"

		echo "text value : ${params.TEST_TEXT}"

		echo "boolean value : ${params.TEST_BOOL}"

		echo "choice value : ${params.TEST_CHOICES}"

		echo "password value : ${params.TEST_PASSWORD}"
	    }
	}
    }
}
```




##### Directives 4 :triggers
- Function : Set build conditions so that the pipeline is automatically build
- Option -> There are more options, if you want to know about it then please refer to https://www.jenkins.io/doc/book/pipeline/syntax/#triggers
	- cron('* * * * *') : define a regular interval at which the Pipeline should be re-triggered
	- pollscm('* * * * *') : define a regular interval at which Jenkins should check for new source changes
- cron syntax : the way to set cron('* * * * *') and pollscm('* * * * *')

| Min | Hour | Dom | Month | DOW |
|:--------|:--------|:--------|:--------|:--------|
| 0-59 | 0-23 | 1-31 | 1-12 | 0-7(0,7 are Sunday)

cf1. * : all valid values

cf2. H : allow periodically scheduled tasks to produce even load on the system (it also can be used with a range)

ex1. once a day on the 1st and 15th of every month except December : triggers{ cron('H H 1,15 1-11 *') 

ex2. once every two hours at 45 minutes past the hour starting at 9:45 AM and finishing at 3:45 PM every weekday. :  triggers{ cron('45 9-16/2 * * 1-5') }

- Example 
```
pipeline {
    agent any
    triggers {
	cron('H */4 * * 1-5')
    }
    stages {
	stage('Example') {
	    steps {
		echo 'Hello World'
	    }
	}
    }
}
```


##### Directives 5 : tools
- Function : auto-install and put on the PATH
- Tools
	- maven    ''
	- jdk ''
	- gradle  ''

- Example
```
pipeline {
    agent any
    tools {
	maven 'apache-maven-3.0.1' 
    }
    stages {
	stage('Example') {
	    steps {
		sh 'mvn --version'
	    }
	}
    }
}
```        
        
#### (b) Directives located between stage and steps
##### Directives 6 :stage
- Function : Define tasks within stages section step by step
- Location : It is in stages section and includes steps section
- Features
	- Stages section must contain at least one stage
	- stage needs to be named
- Example
```
pipeline {
    agent any
    stages {
	stage('Test') {
	    steps {
		echo 'this is test'
	    }
	}
    }
}
```


##### Directives 7 :input
- Function : pause the pipeline and allow a user to input
- Options
	- message : 입력 요구시 사용자에게 출력할 메시지 (require)
	- id : dentifier for this input (default id is stage name)
	- ok : ok button's text
	- submitter : list of users or group names allowed to submit this input
	- parameters : input's form (it is same with parameters directive, so please refer to it)

- Example
```
pipeline {
    agent any
    stages {
	stage('Test') {
	    input {
		message "Do you want to continue?"
		ok "Yes, I do"
		submitter "user1,user2"
		parameters {
		    string(name: 'TEST_TASK', defaultValue: 'Task1', description: 'please input task name')
		}
	    }
	    steps {
		echo "${TEST_TASK} was completed"
	    }
	}
    }
}
```

##### Directives 8 :when 
- Function : Determine whether the stage should be executed depending on the given condition
- Options -> There are more options so if you want to know more about them please refer to https://www.jenkins.io/doc/book/pipeline/syntax/#when
	- when {branch ''} : Execute the stage when the branch being built matches the branch pattern given (ex. git)
	- when { environment name: 'DEPLOY_TO', value: 'production' } : Execute the stage when the specified environment variable is set to the given value
	- when { equals expected: 2, actual: currentBuild.number } : Execute the stage when the expected value is equal to the actual value
	- when { expression {  } }  : Execute the stage when the specified Groovy expression evaluates to true
	- when { not { branch 'master' } } : Execute the stage when the nested condition is false
	- when { allOf { branch 'master'; environment name: 'DEPLOY_TO', value: 'production' } } : Execute the stage when all of the nested conditions are true
	- when { anyOf { branch 'master'; branch 'staging' } } : Execute the stage when at least one of the nested conditions is true
	- triggeredBy : Execute the stage when the current build has been triggered by the param given
		- when { triggeredBy 'SCMTrigger' }   -> triggered by pollscm
		- when { triggeredBy 'TimerTrigger' } -> triggered by  cron
		- when { triggeredBy 'UpstreamCause' } -> triggered by  upstream
		- when { triggeredBy cause: "UserIdCause", detail: "vlinde" } -> triggered by user's requirement

- Example
```
pipeline {
    agent any
    stages {
	stage('Example Build') {
	    steps {
		echo 'Hello World'
	    }
	}
	stage('Example Deploy') {
	    when {
		expression { BRANCH_NAME ==~ /(production|staging)/ }
		anyOf {
		    environment name: 'DEPLOY_TO', value: 'production'
		    environment name: 'DEPLOY_TO', value: 'staging'
		}
	    }
	    steps {
		echo 'Deploying'
	    }
	}
    }
}
```




#### (a) Directives located in steps
##### Directives 9 : script
- Function : allow users to use groovy in steps
- Example
```
pipeline {
    agent any
    stages {
	stage('Example') {
	    steps {
		echo 'Hello World'

		script {
		    def browsers = ['chrome', 'firefox']
		    for (int i = 0; i < browsers.size(); ++i) {
			echo "Testing the ${browsers[i]} browser"
		    }
		}
	    }
	}
    }
}
```
- Exception : flow controls(if, else / try,catch, finally) do not need script block
```
node {
	stage('Example') {
	    if (env.BRANCH_NAME == 'master') {
		echo 'I only execute on the master branch'
	    } else {
		echo 'I execute elsewhere'
	    }
	}
}
```
``` 
node {
	stage('Example') {
	    try {
		sh 'exit 1'
	    }
	    catch (exc) {
		echo 'Something failed, I should sound the klaxons!'
		throw
	    }
	}
}
```

### 4) Parallel Processing 
#### (a) parallel
- Function : Run a specific task parallelly
- Location : within stage 
- Example
```
pipeline {
    agent any
    stages {
	stage('Non-Parallel Stage') {
	    steps {
		echo 'This stage will be executed first.'
	    }
	}

	stage('Parallel Stage') {
	    parallel {
		stage('Branch 1') {
		    agent {
			label "node-branch-1"
		    }
		    steps {
			echo "Branch 1"
		    }
		}
		stage('Branch 2') {
		    agent {
			label "node-branch-2"
		    }
		    steps {
			echo "Branch 2"
		    }
		}
		stage('Branch 3') {
		    agent {
			label "node-branch-3"
		    }
		    stages {
			stage('Nested 1') {
			    steps {
				echo "Nested 1 in Branch 3"
			    }
			}
			stage('Nested 2') {
			    steps {
				echo "Nested 2 in Branch 3"
			    }
			}
		    }
		}
	    }

	}

    }
}
```



#### (b) matix
- Function : Define a multi-dimensional matrix of name-value combinations to be run in parallel

- Features        
	- matrix must contain axes and stages
	- axes includes one or more axis
	- axis consists of a name and a list of values
	- total cell = multiply the number of values in each axis 
- Example -> the tasks(buil->test->deploy) will be done by 3 axis (total cells = 3*4 = 12)
```
matrix {
	axes {
	    axis {
		name 'OS'
		values 'linux', 'mac', 'windows'
	    }
	    axis {
		name 'BROWSER'
		values 'chrome', 'edge', 'firefox', 'safari'
	    }
	}
	stages {
	    stage('build') {
		// ...
	    }
	    stage('test') {
		// ...
	    }
	    stage('deploy') {
		// ...
	    }
	}
}
```





## 4. etc
### 1) How to call environments & parameters
#### (a) In Jenkins Syntax
```
environment 	->  	“${env.variable_name}“
parameters 	->  	“${params.variable_name}“
```
#### (b) In Script 
- groovy 
```
environment 	-> 	env.variable_name
parameters 	-> 	params.variable_name
```
- sh 
```
sh “ ” -> 	sh “${env.variable_name}” 
		sh “${params.변수명}”
sh ‘ ’ -> 	sh ‘${variable_name}’
```
- powershell 
```
powershell “ ” -> 	powershell “${env.variable_name}” 
			powershell “${params.variable_name}”
powershell ‘ ’ -> 	powershell ‘$env:variable_name’ (both env and params use env)
```

#### (c) Example
```
withCridentials([usernamePassword(credentialsId: ‘test1’, userVariable: ‘user1’, passwordVariable: ‘pw1’’), usernamePassword(credentialsId: ‘test2’, userVariable: ‘user2’, passwordVariable: ‘pw2’)]){
	sh ‘${user1}’     
}
```

### 2) How to change environments?
#### (a) environment variables made in environment block cannot be changed like normal variable
```
pipeline{
	agent any
	environment{
		test=’1’
	}
	stages{
		stage(‘test1’){
			steps{
				script{
					env.test = ‘2’  #env is not changed
}
}
			
}
stage(‘test2’){
	steps{
	withEnv([“test=3”]) {
		#Users can use changed env in only this block
}
}
}
	}
}
```
#### (b) environment variables made in script block can be changed like normal variable
```
pipeline{
	agent any
	stages{
		stage(‘test1’){
			steps{
				script{
					env.test = ‘1’
}
}
			
}
stage(‘test2’){
	steps{
script{
		env.test = ‘2’  #test는 2 로 변동
	}
}
}
	}
}
```
