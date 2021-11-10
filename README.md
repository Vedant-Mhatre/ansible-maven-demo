# Jenkins Ansible Maven Sonarqube Demo

## About
This is a demo project where a sample maven web app is built and deployed using tools like Jenkins, ansible and sonarqube

## Steps:

1. Create 2 ec2 ubuntu instances and setup master slave configuration between them - [Reference](https://sensoumya94.medium.com/configure-jenkins-master-slave-architecture-in-aws-a6ea1bda4c0c)

2. Install [Java](https://linoxide.com/install-java-ubuntu-20-04/) and [Maven](https://linuxize.com/post/how-to-install-apache-maven-on-ubuntu-20-04/)

3. Install [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) and [Sonarqube](https://blog.eldernode.com/install-and-use-sonarqube-on-ubuntu/) on slave instance

3. Integrate Sonarqube and Jenkins - [Reference](https://www.tatvasoft.com/blog/integrate-sonarqube-with-jenkins/)

4. Fork this repo.

5. In your forked git repo:

 - Replace sonarqube credentials and git repo url in [pom.xml file](https://github.com/Vedant-Mhatre/ansible-maven-demo/blob/main/pom.xml) with your credentials and git repo url.

 - Replace subnet id, key name in [ansible playbook](https://github.com/Vedant-Mhatre/ansible-maven-demo/blob/main/playbook.yaml) according to your configuration.

 - Upload your key_name.pem file on slave instance in '/home/ubuntu' directory.
 
  - Create/update file named 'ansible.cfg' inside /etc/ansible/ directory according to following config:
```
[defaults]
host_key_checking = False
remote_user = ubuntu
ask_pass = False
private_key_file = /home/ubuntu/your_key_name.pem
```

6. On Jenkins of master node, create a new pipeline project and use this pipeline script:
```
pipeline{
    agent any
    stages{
        stage('SCM Checkout'){
            steps{
               git branch: 'main', url: 'https://github.com/Vedant-Mhatre/ansible-maven-demo.git' {replace my git url with your repo's url}
            }
        }
        stage('Build and compile code'){
            steps{
                sh 'mvn clean package'  
            }
            
        }
        stage('SonarQube analysis') {
            steps{
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar'
                }
             }
        }
        
        stage('Execting Ansible'){
            steps{
                sh 'ansible-playbook playbook.yaml'
               
            }
            
        }
        
    }
}
```

7. When pipeline is executed:

 -  Jenkins will pull code from this repo.

 -  Build and compile web app code to create a jar file inside 'target' directory.

 -  Sonarqube will test web app code present in this repo

 -  Ansible Playbook invoked will then create new instance, copy jar file to new instance, install java and deploy the sample web app on port 8080. 