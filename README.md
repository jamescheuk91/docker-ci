#### RAW
#### BUILD myjenkins image
docker build -t myjenkins jenkins-master/.

#### Build myjenkinsdata image
docker build -t myjenkinsdata jenkins-data/.


#### Run New Data VOLUME
docker run --name=jenkins-data myjenkinsdata

#### USING THE DATA VOLUME
docker run -p 8080:8080 -p 50000:50000 --name=jenkins-master --volumes-from=jenkins-data -d myjenkins

#### Verify by look at log
docker exec jenkins-master tail -f /var/log/jenkins/jenkins.log

#### BUILD THE NGINX IMAGE AND LINK IT TO THE JENKINS ONE
docker build -t myjenkinsnginx jenkins-nginx/.
docker run -p 80:80 --name=jenkins-nginx --link jenkins-master:jenkins-master -d myjenkinsnginx


#### JENKINS IMAGE CLEANUP
docker stop jenkins-nginx
docker stop jenkins-master
docker rm jenkins-nginx
docker rm jenkins-master
docker run -p 50000:50000 --name=jenkins-master --volumes-from=jenkins-data -d myjenkins
docker run -p 80:80 --name=jenkins-nginx --link jenkins-master:jenkins-master -d myjenkinsnginx
#### END RAW

### CLEANUP before docker-compose
docker stop jenkins-nginx
docker rm jenkins-nginx
docker stop jenkins-master
docker rm jenkins-master
docker rm jenkins-data

#### docker-compose
docker-compose build
docker-compose up -d


#### docker-compose pro
docker-compose -f docker-compose.pro.yml build
docker-compose -f docker-compose.pro.yml up -d

#### access postgres from jenkins-master
ping db
set database hostname as "db"
