1. In your AWS Console, under NETWORK & SECURITY, select Key Pairs

2. Click the Create Key Pair button. Name the file tomcat-demo

3. Enter your Key pair name and click Create

    Note: If you do this from a remote computer not running your Jenkins instance you must copy it to the computer running Jenkins.

    I created mine from the computer/system running my Jenkins instance, so it downloaded to my /home/dev-acc/Download/ directory

4. Create a new directory where you'll store your .pem file.

   Note: I have Jenkins running on a *nix instance so I created a directory from /home/dev-acc using:
```
cd ~/
mkdir aws-certs
```
5. Copy your tomcat-demo.pem cert to the aws-certs directory.
```
cd ~/Download/
cp tomcat-demo.pem ~/aws-certs/
```
6. Modify the permissions of your tomcat-demo.pem file. 
```
cd ~/aws-certs/
chmod 600 tomcat-demo.pem
```
7. Modify the ownership of the aws-certs directory and all it's content for the user jenkins
```
cd ~/
sudo chown -R jenkins aws-certs
```
the directory aws-certs and all of it's content will be now owned by the jenkins account.  This account was created when I installed jenkins.  I looked for processes that could be using Jekins with the following command:

`ps axufwwww | grep 'jenkins\|java' -` (courteousy of https://stackoverflow.com/questions/17733671/how-can-i-tell-what-user-jenkins-is-running-as)

I noticed my Jenkins processes were running with a jenkins user account. So, I gave ownership of the directory and contents to jenkins.

8. Use the tomcat-demo.pem file to ssh to your AWS instances and make sure your webapps directory is owned by ec2-user:
```
sudo ssh ec2-user@<your-aws-instance-ip-address> -i tomcat-demo.pem
[ec2-user@<your-aws-instance-ip-address> ~]$ sudo su
[root@<your-aws-instance-ip-address> ~]# chown ec2-user /var/lib/tomcat8/webapps/
```
9. Make sure to adjust your scp commands in the Jenkinsfile to point to the tomcat-demo.pem file and add the option: `-o StrictHostKeyChecking=no` . 

Jenkinsfile
```
pipeline {
    agent any
 
    parameters {
         string(name: 'tomcat_dev', defaultValue: '<staging-aws-instance-ip>', description: 'Staging Server')
         string(name: 'tomcat_prod', defaultValue: '<production-aws-instance-ip>', description: 'Production Server')
    }
 
    triggers {
         pollSCM('* * * * *')
     }
 
stages{
        stage('Build'){
            steps {
                sh 'mvn clean package'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }
 
         stage ('Deployments'){
            parallel{
                stage ('Deploy to Staging'){
                    steps {
                        sh "scp -o StrictHostKeyChecking=no -i /home/dev-acc/instance-cert/tomcat-demo.pem **/target/*.war ec2-user@${params.tomcat_dev}:/var/lib/tomcat8/webapps"
                    }
                }
 
               stage ("Deploy to Production"){
                    steps {
                        sh "scp -o StrictHostKeyChecking=no -i /home/dev-acc/instance-cert/tomcat-demo.pem **/target/*.war ec2-user@${params.tomcat_prod}:/var/lib/tomcat8/webapps"
                    }
                }
            }
        }
    }
}
```
10. Once that's complete, either manually kick off your Pipeilne job or trigger a code change and wait for the deployment.



Note: the `-o StrictHostKeyChecking=no` allows Jenkins to by pass the prompt that you would normally need to confirm with user input.

