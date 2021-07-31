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



#### Section 3 : post
- Function : define one or more additional steps after the completion of pipelien or stage
- Location : after stages in pipeline block or after steps in stage block 
- Conditions
	- always{} : 파이프라인 또는 stages의 작업결과와 상관없이 post 의 작업 실행
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
		stage{
			steps{
			
			}
			post{
			
			}
		}
	}

	post{
    		always{

    		}
	}
}
```



### 3) Directives
#### (a) Directives located between agent and stages
##### Directives 1 : environment
    1. 기능 : 전역변수 설정

    2. 활용 : credentials()를 자주 사용
        ex.Username and Password Credentials 활용
            pipeline {
                agent any
                stages {
                    stage('Example Username/Password') {
                        environment {
                            SERVICE_CREDS = credentials('my-predefined-username-password')
                        }
                        steps {
                            sh 'echo "Service user is $SERVICE_CREDS_USR"'
                            sh 'echo "Service password is $SERVICE_CREDS_PSW"'
                            sh 'curl -u $SERVICE_CREDS https://myservice.example.com'
                        }
                    }
                    stage('Example SSH Username with private key') {
                        environment {
                            SSH_CREDS = credentials('my-predefined-ssh-creds')
                        }
                        steps {
                            sh 'echo "SSH private key is located at $SSH_CREDS"'
                            sh 'echo "SSH user is $SSH_CREDS_USR"'
                            sh 'echo "SSH passphrase is $SSH_CREDS_PSW"'
                        }
                    }
                }
            }

        cf. Username and Password Credentials은  username:password로 설정되어있는 credentials임
            각 값 호출 방법
            SSH_CREDS = credentials('my-predefined-ssh-creds') 와 같이 credentials을 SSH_CREDS에 담으면,
            아이디 호출 시 : SSH_CREDS_USR
            비밀번호 호출시 : SSH_CREDS_PSW










##### Directives 2 :options -> 엄청 많은 옵션이 있는데 아직 딱히 사용할 거 같지 않아서 생략
    1. 기능 : 파이프라인 또는 stage의 특정 옵션 설정
    
    2. 자주 사용하는 옵션
        (1) retry ->  options { retry(3) }
        (2) timeout ->  options { timeout(time: 1, unit: 'HOURS') }

    3. 예시
        pipeline {
            agent any
            options {
                timeout(time: 1, unit: 'HOURS') 
            }
            stages {
                stage('Example') {
                    steps {
                        echo 'Hello World'
                    }
                }
            }
        }




##### Directives 3 :parameters
    1. 기능 : 파이프라인 빌드 시 사용자에게 입력받을 값

    2. 옵션
        string : parameters { string(name: 'DEPLOY_ENV', defaultValue: 'staging', description: '') }

        text :  parameters { text(name: 'DEPLOY_TEXT', defaultValue: 'One\nTwo\nThree\n', description: '') }

        booleanParam :parameters { booleanParam(name: 'DEBUG_BUILD', defaultValue: true, description: '') }

        choice : parameters { choice(name: 'CHOICES', choices: ['one', 'two', 'three'], description: '') }

        password :  parameters { password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'A secret password') }


    3. 예시
        pipeline {
            agent any
            parameters {
                string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')

                text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')

                booleanParam(name: 'TOGGLE', defaultValue: true, description: 'Toggle this value')

                choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')

                password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')
            }
            stages {
                stage('Example') {
                    steps {
                        echo "Hello ${params.PERSON}"

                        echo "Biography: ${params.BIOGRAPHY}"

                        echo "Toggle: ${params.TOGGLE}"

                        echo "Choice: ${params.CHOICE}"

                        echo "Password: ${params.PASSWORD}"
                    }
                }
            }
        }





##### Directives 4 :triggers
    1. 기능 : 파이프라인이 자동빌드 되도록 빌드 조건을 설정하는 지시어

    2. 옵션
        cron('* * * * *') : 특정 시간주기마다 빌드 일어나도록 하는 옵션

        pollscm('* * * * *') : 특정 시간주기마다 소스 확인해 변경사항이 있으면 실행하는 옵션

        upstream(upstreamProjects: 'job1,job2', threshold: hudson.model.Result.SUCCESS) : string이 설정한 임계값 넘으면 실행하는 옵션-> 써본적이 없어 이해 떨어짐 

    3. cron syntax : cron('* * * * *'), pollscm('* * * * *') 에서 시간 주기 설정하는 방법
        분      시간    일      달      요일
        *       *       *       *       *
        0-59    0-23    1-31    1-12    0-7(0,7은 일요일임)

        cf1. * : 모든 유효값을 설정
        cf2. H는 특정 시간을 지칭하는 것이 아닌 랜덤하게 한시간대를 설정(사용 이유 : 너무 한 시간대에 많은 파이프라인이 빌드되게 설정하면 과부하가 걸릴 수 있기 때문)

        ex1. once a day on the 1st and 15th of every month except December : triggers{ cron('H H 1,15 1-11 *') }
        ex2. once every two hours at 45 minutes past the hour starting at 9:45 AM and finishing at 3:45 PM every weekday. :  triggers{ cron('45 9-16/2 * * 1-5') }


    4. 예시 
        // Declarative //
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



##### Directives 5 :tools -> 써본적이 없어 이해 떨어짐
    1. 기능 : 자동설치 및 path설정을 위한 섹션

    2. 옵션
        maven    ''
        jdk ''
        gradle  ''

    3. 특징

    4. 예시
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
        
        
#### (b) Directives located between stage and steps
##### Directives 6 :stage
    1. 기능 : stages 섹션 안의 작업들을 나누는 지시어

    2. 특징 :   stages 안에 적어도 하나의 stage는 있어야함
                stage의 이름을 설정해줘야함

    3. 예시
        pipeline {
            agent any
            stages {
                stage('Example') {
                    steps {
                        echo 'Hello World'
                    }
                }
            }
        }



##### Directives 7 :input
    1. 기능 : 특정 stage에서 사용자에게 입력값 받기 위한 지시어

    2. 옵션
        (1) message : 입력 요구시 사용자에게 출력할 메시지 (무조건 있어야함)
        (2) id : 이 입력에 대한 식별자 (디폴트는 스테이지이름)
        (3) ok : 입력 양식이 ok버튼임
        (4) submitter : 입력값 입력 가능한 사용자 지정
        (5) parameters : 기존 paramteters 지시어와 동일하게 사용 가능

    3. 예시
        pipeline {
            agent any
            stages {
                stage('Example') {
                    input {
                        message "Should we continue?"
                        ok "Yes, we should."
                        submitter "alice,bob"
                        parameters {
                            string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
                        }
                    }
                    steps {
                        echo "Hello, ${PERSON}, nice to meet you."
                    }
                }
            }
        }


##### Directives 8 :when
    1. 기능 : stage가 실행할지 결정하는 조건문 

    2. 옵션 
        (1) when {branch ''} : git의 브랜치에 따라 실행
        (2) when { environment name: 'DEPLOY_TO', value: 'production' } : 특정 전역변수(environment)의 값에 따라 실행
        (3) when { equals expected: 2, actual: currentBuild.number } : 기대값과 실제값 동일 여부에 따라 실행
        (4) when { expression {  } } 안의 출력값이 true, false 에 따라 실행 여부 결정(if 와 값음) 
        (5) when { not { branch 'master' } } : ~조건이 아닐 시 실행
        (6) when { allOf { branch 'master'; environment name: 'DEPLOY_TO', value: 'production' } } : 안의 조건 중 모두 해당되면 실행(조건 구별 ;)
        (7) when { anyOf { branch 'master'; branch 'staging' } } : 안의 조건 중 최소 하나만 해당되면 실행(조건 구별 ;)
        (8) triggeredBy
            -   when { triggeredBy 'SCMTrigger' }   -> pollscm 이 trigger로 실행된 경우
            -    when { triggeredBy 'TimerTrigger' } -> 일정 시간주기가 trigger로 실행된 경우
            -    when { triggeredBy 'UpstreamCause' } -> upstream 이 trigger로 실행된 경우
            -    when { triggeredBy cause: "UserIdCause", detail: "vlinde" } -> 특정사용자의 요청이 trigger로 실행된 경우

    3. 예시 
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


##### Directives 9: parallel
    1. 기능 : 특정 작업을 병력적으로 실행하기 위한 지시어

    2. 예시 :
        pipeline {
            agent any
            stages {
                stage('Non-Parallel Stage') {
                    steps {
                        echo 'This stage will be executed first.'
                    }
                }

                stage('Parallel Stage') {
                    when {
                        branch 'master'
                    }
                    failFast true

                    parallel {
                        stage('Branch A') {
                            agent {
                                label "for-branch-a"
                            }
                            steps {
                                echo "On Branch A"
                            }
                        }
                        stage('Branch B') {
                            agent {
                                label "for-branch-b"
                            }
                            steps {
                                echo "On Branch B"
                            }
                        }
                        stage('Branch C') {
                            agent {
                                label "for-branch-c"
                            }
                            stages {
                                stage('Nested 1') {
                                    steps {
                                        echo "In stage Nested 1 within Branch C"
                                    }
                                }
                                stage('Nested 2') {
                                    steps {
                                        echo "In stage Nested 2 within Branch C"
                                    }
                                }
                            }
                        }
                    }

                }

            }
        }



#### (a) Directives located in steps
##### Directives 10 : script
    1. 기능 : steps 내 작업 입력 시 스크립트(groovy) 사용을 위해 사용하는 지시어

    2. 예시
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
        
    3. 예외
        flow control(if, else / try,catch, finally)와 관련된 groovy 명령어는 steps와 script블럭이 필요없이 사용 가능
        ex1.
            node {
                stage('Example') {
                    if (env.BRANCH_NAME == 'master') {
                        echo 'I only execute on the master branch'
                    } else {
                        echo 'I execute elsewhere'
                    }
                }
            }

        ex2. 
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



#### (a) Directives located in an unusal place
##### Directives 11 : matix
    1. 기능 :   parallel와 같이 병렬 실행을 위한 지시어임
                그러나 parallel의 단점 개선
                parallel의 단점 : 병렬로 진행하고 싶은 작업을 반복적으로 기입해야하기 때문에 동시에 많은 작업을 수행하게 만들 때 코드가 길어짐
                matrix의 개선점 : cell의 개념을 통해 적은 수의 코드 중복을 피함

    2. 구조 
        - matrix 안에는 axes와 stages가 무조건 있어야함
        - axes 안에는 여러개의 axis로 구성
        - axis는 name 과 values를 입력받음
        - 각axis의 values 수의 곱한 만큼의 cell이 생김

    3. 예시
        ex1.buil->test->deploy의 작업을 axis의 3개의 cell이 각각 실행
            matrix {
                axes {
                    axis {
                        name 'PLATFORM'
                        values 'linux', 'mac', 'windows'
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

        ex2. build-and-test 라는 작업을 12개의 cell이 실행
            matrix {
                axes {
                    axis {
                        name 'PLATFORM'
                        values 'linux', 'mac', 'windows'
                    }
                    axis {
                        name 'BROWSER'
                        values 'chrome', 'edge', 'firefox', 'safari'
                    }
                }
                stages {
                    stage('build-and-test') {
                        // ...
                    }
                }
            }


### 4) Steps
    1. 기능 : stage 가 수해할 작업을 정의하는 블록

    2. 예시
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



## 4. etc
### 1) How to call environments & parameters
#### (a) Jenkins일때
		environment ->  “${env.변수명}“
    parameters ->  “${params.변수명}“

#### (b) Script 일때
- groovy 
	environment -> env.변수명
parameters -> params.변수명

- sh 
sh “ ” 시 -> 	sh “${env.변수명}” 
sh “${params.변수명}”
sh ‘ ’ 시 -> 	sh ‘${변수명}’

- powershell ->
			powershell “ ” 시 -> 	powershell “${env.변수명}” 
powershell “${params.변수명}”
			powershell ‘ ’ 시 -> 	powershell ‘$env:변수명’ (env, params 모두 env로)

withCridentials([usernamePassword(credentialsId: ‘test1’, userVariable: ‘user1’, passwordVariable: ‘pw1’’), usernamePassword(credentialsId: ‘test2’, userVariable: ‘user2’, passwordVariable: ‘pw2’)]){
	sh ‘${user1}’     -> 이런식으로 호출해서 사용 가능
}


### 2) How to change environments 
environment 블록에서 만든 environment -> 일반 변수처럼 변경 불가
ex.
pipeline{
	agent any
	environment{
		test=’1’
	}
	stages{
		stage(‘test1’){
			steps{
				script{
					env.test = ‘2’  #변경되지 않음
}
}
			
}
stage(‘test2’){
	steps{
	withEnv([“test=3”]) {
		#이 블럭 안에서만 해당 env값 변경
}
}
}
	}
}

script 블록 안에서 만든 environment -> 일반 변수처럼 변경 가능
ex.
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
