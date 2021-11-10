# Jenkins Ansible Maven Sonarqube Demo

## About
This is a demo project where a sample maven web app is built and deployed using tools like Jenkins, ansible and sonarqube

## Steps:

1. Create 2 ec2 ubuntu instances and setup master slave configuration between them - [Reference](https://sensoumya94.medium.com/configure-jenkins-master-slave-architecture-in-aws-a6ea1bda4c0c)

2. Install [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) and [Sonarqube](https://docs.sonarqube.org/latest/setup/install-server/) on slave instance

3. Integrate Sonarqube and Jenkins - [Reference](https://www.tatvasoft.com/blog/integrate-sonarqube-with-jenkins/)

4. SSH into slave instance, clone this repo and cd into it:
```bash
git clone https://github.com/Vedant-Mhatre/ansible-maven-demo.git
```
```
cd ansible-maven-demo
```

5. Install [Java](https://linoxide.com/install-java-ubuntu-20-04/) and [Maven](https://linuxize.com/post/how-to-install-apache-maven-on-ubuntu-20-04/)

6. Build and compile code:
```
mvn clean package
```
This is will create jar file inside directory named 'target'

7. On Jenkins of master node, create a new pipeline project and use this pipeline script:
```
pipeline{
    agent any
    stages{
        stage('SCM Checkout'){
            steps{
               git branch: 'main', url: 'https://github.com/Vedant-Mhatre/ansible-maven-demo.git'
            }
        }
        stage('Execting Ansible'){
            steps{
                sh 'ansible-playbook playbook.yaml'
               
            }
            
        }
        stage('SonarQube analysis') {
            steps{
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar'
                }
             }
        }
    }
}
```

8. Sonarqube will test web app code present in this repo.

9. Ansible Playbook invoked by jenkins pipeline will create new instance, copy jar file to new instance, install java and deploy the sample web app on port 8080.

## To Do:

- [ ] Automate building and compiling part of sample web app in pipeline script