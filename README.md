# install-Jenkins-with-Artifactory
In this repository you can find omplete instructions on how to install and configure jenkins, docker. In docker create an artifactory container. And how to connect all this and create a job and configure it.

## 1. Install Jenkins
* Open the console and enter the following commands:
```
$ sudo apt-get update
$ sudo apt-get install jenkins
  ```
* Using systemctl we'll start Jenkins:
```
$ sudo systemctl start jenkins
```
* Next we check the status of Jenkins:
```
$ sudo systemctl status jenkins
```
* If everything went well, the start of the output should show that the service
is active and configured to start at boot:
```
jenkins.service - LSB: Start Jenkins at boot time
  	Loaded: loaded (/etc/init.d/jenkins; bad; vendor preset: enabled)
  	Active:active (exited) since Thu 2017-04-20 16:51:13 UTC; 2min 7s ago
  	Docs: man:systemd-sysv-generator(8)
 ```
> The jenkins installation is complete.
Now we start it and we configure.
First enter the admin password.
Then jenkins suggests to install the necessary plug-ins, we choose independently or we choose, that jenkins himself installed plug-ins. When the installation is complete, we'll be prompted to set up the first administrative user.
## 2. Configure jenkins:
* We go to the Jenkins Management tab further in the Global Configuration Tool.
  install the necessary tools - such as git, maven, etc.
## 3. Create a new slave:
* We go to the Build Executor Status tab and select **New Node**.
* Enter the **Node** name.
* Set it up:
  - Enter the **Remote root directory**, for example ( _/var/lib/jenkins/slave_ ).
  - Enter **Labels**, for example ( _linux ubuntu18._ ).
  - Select the launch method: select the **__launch agent agents via Java Web Start__**.
  - Save the settings.
* To run a slave on this system:
  - Download **jar** file, which gives jenkins.
  - Open the folder that you chose as “Remote root directory” and move the jar file there.
  - Open the terminal and execute the command, which jenkins will say in my case:
  ```
  $ java -jar agent.jar -jnlpUrl http://127.0.0.1:8080/computer/EPBYBREW0019/slave-agent.jnlp -	secret  e929aa50648e0fd7c69aacd44d691a9cb1f422ec14b27d94fb1b400d6659bbbb -workDir "/	var/lib/jenkins/slave"
  ```
## 4. Connecting the project to the git repository:
* Open the **job** and go to **Configure**.
* We go to the **Source Code Management** tab and click **Git**.
* Enter **Repository URL**.
* Enter the **login** and **password**.
* Choose a branch.
* You can also set a trigger for when the project is going,  chose **Poll SCM**.
* I set ( _0 H / 6 * * *_ ) this means that jenkins will check for changes on the git repository every 6 hours.
## 5. Install Docker :
* Update the apt package index:
```
$ sudo apt-get update
```
* Install the specific version of the **Docker CE**;
```
$ sudo apt install docker-ce
```
* That's all, **Docker** installed.
## 6. Install Artifactory:
* The **Artifactory Docker** image can be pulled from Bintray by executing the corresponding Docker command below:
```
$ docker pull docker.bintray.io/jfrog/artifactory-oss:latest
```
* You can list the Docker images command:
```
$ docker images
```
* To start the container, you must execute the following command:
```
$ docker run --name artifactory -d -p 8081: 8081 docker.bintray.io/jfrog/artifactory-oss:latest
```
* That's all, **Artifactory** is installed and running, now you can go to the browser and enter address _http: // 127.0.0.1:8081/artifactory_ .
## 7. Creating a job with artifactory:
* Open the tab **Manage Jenkins** -> **Manage Plugins**, in the search enter **_Artifactory Plugin_** and install it.
* Open the **Manage Jenkins** -> **Configure System** tab:
  - Find the item **Artifactory**.
  - Click **Add Artifactory Server**.
  - Copy  **ServerId** from docker - this is the name of docker artifactory container.
  - Paste the **URL** ( _http: // 127.0.0.1:8081/artifactory_ ).
  - Enter **Username** and **Password**.
  - Click **TestConnection**, if it displays the found version of Artifactory, then everything is done correctly
* Press **New item**, enter the name job, and select **Pipline**
* Go to Jfrog company official GitHub [repository](https://github.com/jfrog) and go to the [maven-example](https://github.com/jfrog/project-examples/blob/master/jenkins-examples/pipeline-examples/maven-example/Jenkinsfile) and copy the script from there.
* Substitute the desired values and get:
```
node {
    def server = Artifactory.server '47ee5d3c2ab6033efa6527246a3ad7678485eb0f888e98059497879c58f96450'
    def rtMaven = Artifactory.newMavenBuild()
    def buildInfo
    stage ('Clone') {
        git url: 'https://github.com/evtushenko-sasha/ListOfTasks'
    }
    stage ('Artifactory configuration') {
        rtMaven.tool = '/usr/bin/mvn' // Tool name from Jenkins configuration
        rtMaven.deployer releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local', server: server
        buildInfo = Artifactory.newBuildInfo()
        buildInfo.env.capture = true
    }
    stage ('Exec Maven') {
        rtMaven.run pom: 'pom.xml', goals: 'clean install -P production', buildInfo: buildInfo
    }
    stage ('Publish build info') {
        server.publishBuildInfo buildInfo
    }
}
```
* Paste this into the **pipline** section
* You can also set **Build Trigger** to follow the change in the git repository (see above).
