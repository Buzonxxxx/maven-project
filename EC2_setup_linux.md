#EC2 Steup

#### EC2 Server:
 1. ssh ec2-user@52.79.235.30 -i tomcat-demo.pem (stage)
 2. ssh ec2-user@54.180.103.165 -i tomcat-demo.pem (production)
#### 1. Steup tomcat server
  - By package
  ```
  sudo yum install tomcat7
  sudo service tomcat7 start
  sudo su
  chown ec2-user /var/log/tomcat7/
  chown ec2-user /var/lib/tomcat7/webapps/
  ```

  - By docker
  ```
  # setup docker environment
  sudo yum install -y docker
  service docker start
  docker run hello-world 

  # start process
  docker run -p 9090:8080 tomcat:latest
  (docker run -p host-port:container-port tomcat:latest)

  # start process in background
  docker run -d -p 9090:8080 tomcat:latest

  # enter container
  docker exec -it [container ID] /bin/bash

  # check all process
  docker ps -a

  # kill process
  docker kill [container ID]
  # kill all process
  docker kill $(docker ps -q)
  
  # stop process
  docker stop [container ID]
  # stop all process
  docker stop $(docker ps -q)
  
  # list container
  docker container ls
  # remove all container
  docker rm $(docker container ls -aq)

  # remove img
  docker rmi -f tomcat
  # remove all image
  docker rmi $(docker images -a -q)
  ```