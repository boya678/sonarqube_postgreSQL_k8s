This project will allow you to create a local persisted SonarQube instance that acts as your own personal sandbox
for performing static analysis. You deploy this onto your local Kubernetes cluster where you would
expose your service.

**Installed will be the latest LTS SonarQube 7.9.1-community version.**

To learn more about the 7.9 LTS features of SonarQube, please refer to the following documents
[7.9 LTS SonarQube Features](https://www.sonarqube.org/sonarqube-7-9-lts/)

---
#### Explanation of each Kubernetes config file
In order to persist changes within SonarQube, a persistent volume such as Postgres is required.

##### Postgres related
- `sonar-pv-postgres.yml`

    This is the Postgres **p**ersistent **v**olume that abstracts away the persistent storage.

    They have a lifecycle that is independent of the **Pod** and/or **Deployment** resources and are thus not impacted by their health.

    More information can be found here [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes)


- `sonar-pvc-postgres.yml`

    **P**ersistent **V**olume **C**laim.

    This is the request for storage by a user. A PVC consumes PV resources.

    More information can be found here [Persistent Volume Claims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)


- `sonar-postgres-deployment.yml`

    This is where the container specification for the Postgres Pod resource lives.

    The config file also connects the Postgres claim to the Pod resource via volume mounts.

    More information can be found here [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)


- `sonar-postgres-service.yml`

    This is where the Postgres service gets exposed (as a network service) to other resources (like the SonarQube pod resource).

    More information can be found here [Services](https://kubernetes.io/docs/concepts/services-networking/service/)

##### SonarQube related
- `sonarqube-deployment.yml`

    This is where the container specification for the 7.9.1-community version of SonarQube lives.

    The config file states that 2GB of memory is required.

    More information can be found here [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)


- `sonarqube-service.yml`

    This is where the SonarQube service gets exposed (as a network service) to other resources.

    It is with this config file that we are allowed to hit the SonarQube service from a browser.

    More information can be found here [Services](https://kubernetes.io/docs/concepts/services-networking/service/)

---
#### Install and run SonarQube locally

##### Install Docker Desktop and configure Kubernetes
1. Follow the instructions here [Docker Desktop](https://docs.docker.com/docker-for-mac/install/) to install Docker desktop
2. Once Docker Desktop is installed, in your menu bar, click on the Docker icon
    1. Open `Preferences`
    2. Go to the `Advanced` tab
    3. Drag the `Memory` slider to `4.0 GiB`
    4. Go to the `Kubernetes` tab
    5. Click the checkbox `Enable Kubernetes`
    6. Click `Apply` and wait for Docker and Kubernetes to restart
3. Ensure that the `docker-desktop` Context is set 
    1. Click on Docker icon in menu bar
    2. Click on `docker-desktop` in the `Kubernetes` section

#####  Install Sonar scanner
1. `brew install sonar-scanner`

#####  Deploy your instance of SonarQube (will take a few minutes for pods to fully warm up and load SonarQube)
1. `kubectl create secret generic postgres-pwd --from-literal=password={some made up password}`
2. `kubectl create -f sonar-pv-postgres.yml`
3. `kubectl create -f sonar-pvc-postgres.yml`
4. `kubectl create -f sonar-postgres-deployment.yml`
5. `kubectl create -f sonarqube-deployment.yml`
6. `kubectl create -f sonarqube-service.yml`
7. `kubectl create -f sonar-postgres-service.yml`

##### Access your local SonarQube
1. Find the `nodePort` of your `sonar` service
    1. `kubectl get service sonar --output yaml`
    2. Copy the number next to `nodePort` IE: `31564`
2. Open a new Chrome/Firefox tab/browser
    1. Go to `localhost:{nodePort you just copied}`

##### To delete the Kubernetes deployment is as follows
1. `kubectl delete deployment sonar-postgres`
2. `kubectl delete deployment sonarqube`

---
#### Perform static analysis

##### Java
1. In the root of your Java project, add an empty `sonar-project.properties` file. The `sonar-scanner` service will be 
looking for this file when performing static analysis.

2. Paste the following into the newly created `sonar-project.properties` file:
```
sonar.projectKey={name of project}
sonar.host.url=http://localhost:##### (nodePort of sonar service)
sonar.login=${env.SONAR_TOKEN}
sonar.java.binaries=build/classes
sonar.sources=src/main/java
``` 

An example config would look like the following
```
sonar.projectKey=notificationemailproc
sonar.host.url=http://localhost:31828
sonar.login=${env.SONAR_TOKEN}
sonar.java.binaries=build/classes
sonar.sources=src/main/java
```

3. Create and copy a new SonarQube token by going to the SonarQube instance in the browser and navigating to 
My Account -> Security tab -> Enter Any Token Name You Want -> Generate -> Copy token generated

4. In your .bashrc or .zshrc file, add the following line
`export SONAR_TOKEN={SonarQube token that was just copied}`

5. Reload your rc file
`source ~/.bashrc` or `source ~/.zshrc`

6. Run `sonar-scanner` in the project root directory. Once static analysis is finished, you can view the results in your 
SonarQube instance in the browser.

---
#### Useful SonarQube plugins
- FindBugs
- Checkstyle
- Mutation Analysis