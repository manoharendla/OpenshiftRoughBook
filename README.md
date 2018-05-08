# 08/05/2018
## Using template:
oc create -f https://raw.githubusercontent.com/siamaksade/jenkins-s2i-example/master/jenkins-master-s2i-template.yaml
oc new-app jenkins-master-s2i
'''
Cloning "https://github.com/siamaksade/jenkins-s2i-example.git " ...
	Commit:	29d91a30887a295e54583e7b9157a6c2532ada63 (added metadata to Dockerfile)
	Author:	Siamak Sadeghianfar <ssadeghi@redhat.com>
	Date:	Thu May 26 13:46:32 2016 +0200
Pulling image "registry.access.redhat.com/openshift3/jenkins-1-rhel7@sha256:ccb6f09b52e2f8633b65e1bfde18bab62fa7b01f6f0c1e406f6869c461384502" ...
ERROR: Error writing header for "scripts": io: read/write on closed pipe
ERROR: Error writing tar: io: read/write on closed pipe
error: build error: Error response from daemon: {"message":"create 285619fe21e31b15088ba552e95797092cb79c8d276c8f54f76813b6f738ff0e: error while creating volume path '/var/lib/docker/volumes/285619fe21e31b15088ba552e95797092cb79c8d276c8f54f76813b6f738ff0e/_data': mkdir /var/lib/docker/volumes/285619fe21e31b15088ba552e95797092cb79c8d276c8f54f76813b6f738ff0e: permission denied"}
'''


# 02/05/2018
## OpenIssues
1. Cannot install JDK 1.7 that is needed for some of the jobs.
  *Approaches Tried:*
  * Using ManageJenkins -> GlobalToolConfiguration-> Install JDK from SUN(Install failed with connection refused while downloading     packages from Sun site) 
  * Using package installation apporach : Downloaded the package from SUN to mac. Using rsync copy the installable to pod, unzip and untar the file and provide the command details in  ManageJenkins -> GlobalToolConfiguration. Again it failed to execute the Java 7
 
2. RPM-BUILD: Some jobs are getting failed due to rpmbuild package missing on the pod, to install this package we need sudo permissions
3. SMTP ERROR : Need smtp server details to configure
4. DEFAULT Setting : While doing build, build is looking for settings.xml file and failing with DEFAULT SETTINGS.XML FILE not found error. Can be automated using openshift config maps. 

Note: As part of migration, numerous plugins has been installed on the pod using oc rsync 
  


# 26/04/2018 
## Prerequisite :
1. Create config map for saxon and maven settings.xml 


## Using Jenkkins default template
1. oc project s2i
2. oc new-build https://github.com/manoharendla/AutomateJenkins.git --context-dir=. --strategy=docker


## Using a custom docker image
1. git clone https://github.com/manoharendla/AutomateJenkins.git
2. docker build -t jenkins-cicd .
3. docker tag & docker push to docker/openshift registry
4. oc project s2i
5. oc import-image s2i --from=docker.io/docmaster/jenkins-cicd --confirm-all
6. oc new-app jenkins-cicd-ephernam -p NAMESPACE=s2i -p JENKINS_IMAGE_STREAM_TAG=jenkins-cicd:latest


## Using  template
1. Build a custom image as shown above
2. oc create -f https://raw.githubusercontent.com/manoharendla/AutomateJenkins/master/jenkins-master-s2i.yml
3. oc new-app jenkins-master-s2i -p JENKINS_IMAGE=docker.io/dockmaster/jenkins-cicd:latest


## Using persistent template
1. oc create -f https://raw.githubusercontent.com/manoharendla/AutomateJenkins/master/jenkins-persistent.yml
2. oc new-app <app-name>





# 25/04/2018

1. Created a cusotmized docker image having  customized config.xml file, configured with creds etc
2. with docker run Jenkins is up and running
3. Pushed the image to openshift registry
4. From openshift tried to deploy an new applicaiton using deploy image , deployment is getting succeeded but routes are not getting created. Pod is running good and from logs Jenkins is fully up and runnning 
5. Manually tried to create a route but still it didn't help


# RoughBook

Why we created DSL when jenkins file exist

There are some options in ahop-job in inova instance like recursive or snapshot dependencies. There are not pre defined api in Jenkins file

So DSL is another approach with which we can create a script for our Jenkins job.


# Issues observed:

Once we increased our Ram size from 512MB to 1GB, sample job was able to progress but failed at nabu step that is failing to find a file that should be part of Repository

Cloudfees nabu folder is not getting cloned so we manually cloned the reposiotry to get the build succeed.


# Created a Docker file

Create a Docker file that will create a custom image with Checkmarx plugin installed in it.
Also make modfiyications to create an image  which contains jobs in it 

For sample, created some config test files in config folder 


1. Create the image with custom defined plugins and Jobs 
2. use docker build -t "custom-jenkins" .
3. that creats a docker image
4. Make this image available in our Open shift image registry
5. Creat a deployment yaml that makes use of this image .
6. Automate the container deployment through a shell script 




# 4/9/2018:

Jenkins persistent images is predifined with volume 1 Gb for /var/lib/jenkins i.e, Jenkins home.

There is no option to modify the volume size from webconsole 

So I see two options:

1. See if we can  update the volume from command line using oc command line tools . We need to redploy jenkins after update to the volume configuration
2. Or create a new mount point with defined sizeand  with the help of environment variables using config maps, use the new mount point as Jenkins home.



## Commands to list volumes: (Donot perform oc rsh <podname>)
1. Logon on to project `oc project <project-name>`
2. `oc get pv` (lists of all available persistent volumes configured)
3. To claim a suitable persistent volume and mount it using 
   `oc volume dc/nginx --add --claim-size 512M --mount-path /usr/share/nginx/html --name downloads` 
4. `oc get pv` will list the new persistent volume claim attached to one of the peristent volumes available 
5. `oc get pvc` to list all persistent volume claims
6. `oc volume dc --all` to which applications the pvc are attached to.
7. `oc volume dc/nginx --remove --name downloads` to deattach the volumes . This will not remove the pvc from the list. The command will deattach/unmount the volume from nginx application. So that the conatiner cannot use the mount point
8. `oc volume dc --all`
9.  `oc volume dc/nginx --add --type pvc --claim-name pvc-rkly7 --mount-path /usr/share/nginx/html --name downloads` remount the same pvc into the applicaiton.
10. `oc volume dc/nginx --remove --name downloads` and `oc delete pvc/pvc-rkly7` to detach and delete the pvc completely from pvc list. If so the complete data will be lost.
11. `oc describe pod <podname>`  to get the pod details on the console
12. `oc get pod <podname> -o yaml` to get the details in yaml output
13. `oc rsh <podname>`
14. `mount`   
15. `df -h`
16. `cd /var/lib/jenkins`   
17. `du -h --max-depth=2`   
   
  
  
  
### Note: dont run steps 3,7,9,10 . And for the remaining commands please send the output


# 4/11/2018:

1. `oc get pvc` to list all persistent volume claims in the name space. This will o/p the pvc name, size and mount path. 
2.  Only jenkins pvc is avilable in the name space and is having 1Gi capacity.
3. *First option:* Tried resizing the pvc using  `oc volume dc/jenkins --add --overwrite --type pvc --claim-name jenkins -- claim-size 2Gi` . But oc is not allwoing it to resize the volume with forbidden error.
4.  *Second option:* `oc get pvc jenkins -o yaml > pvc.yaml` . Get the pvc configuration in yaml format and redirect it to pvc.yaml.
5. Modify the capacity key from 1Gi to 2Gi in pvc.ymal file
6. `oc create -f pvc.yaml`. using oc create tried to create a pvc object, but this also failed with similar error.
7. *Third option:* `oc edit pvc jenkins`. Tried to modify the capacity from  pvc object, but even this failed with error. The error says once the pvc is created it is immutable. SO we cannot resize the pvc.
8. So as all the options are tried out, the only option I see right now is to recreate a new deployment configuration with defined size.
9. Also check with the team if there is any option to  resize the existing jenkins pvc.


# 4/12/2018:
1. Create a new mount to store the .m2 configuration. As of now by default the .m2 is getting stored at /var/lib/jenkins.
2. The steps would be to create a new mount named maven and mount it to /maven/
  i. `oc get pvc` 
 ii. Create a new pvc object for maven
     ```yaml
     apiVersion: "v1"
     kind: "PersistentVolumeClaim"
     metadata:
       name: "maven"
     spec:
       accessModes:
         - "ReadWriteOnce"
     resources:
       requests:
         storage: "5Gi"
     volumeName: "pv0001"
     ```
iii. Using this claim as volume in the pods
     ```yaml
     apiVersion: "v1"
     kind: "Pod"
     metadata:
       name: "mypod"
     labels:
      name: "frontendhttp"
     spec:
      containers:
        -
          name: "myfrontend"
          image: "nginx"
          ports:
        -
          containerPort: 80
          name: "http-server"
      volumeMounts:
        -
          mountPath: "/var/www/html"
          name: "maven"
     volumes:
      -
        name: "maven"
        persistentVolumeClaim:
        claimName: "maven" 
     ```
3. Modify the maven configuration to store the repositories in the new mount point.
4. Usually maven installed through Jenkins will be placed at <JENKINS_HOME>/tools. 




*references:*
https://docs.openshift.com/enterprise/3.1/dev_guide/persistent_volumes.html


## oc rsync: 
1. To copy a single file from the container to the local machine, the form of the command you need to run is:

`oc rsync <pod-name>:/remote/dir/filename ./local/dir`

2. Uploading files from a conatiner:

`oc rsync ./local/dir pod-name:/remote/dir`

we can use --exclude=* and --include=filename and also --delete to have the both folders in sync and also --no-perms to forget about permissions
*ref:* https://blog.openshift.com/transferring-files-in-and-out-of-containers-in-openshift-part-1-manually-copying-files/

## Jenkins Credentails Automation:

There are two options to automate the credentials addition in Jenkins.

*First option:* To store password in plain text in credentials.xml, copy it over to jenkins machine after installing and starting the service. Then jenkins will encrypt it with it's new secret  and it will work
*Second option:*  A second option is to install jenkins, start it and then copy the credentials.xml with encrypted passwords together with secrets directory and secret.xml from previous instllation. This will copy both encryption master key and the encrypted credentials that have been created using this master key.


Copy the credentials.xml file, secrets directory and secrets.xml from working pod to local using rsync.

# How to copy the above files to pod as part of automatic deployment.

1. The files should be mounted once the /var/lib/jenkins mount point is available.
2. Store the files in a private git repository in github internal
3. Using pod life cycle hooks, clone the repo on to the pod and also execute a command to copy that to /var/lib/jenkins.
  *Option 1:* Directly update the dc  using `oc edit dc jenkins` and update the spec with post 
   ```yaml
   spec:
    ...
    ....
    strategy:
     ....
    .....
    rollingParams:
     post:
       execNewPod:
         command:
         - /bin/sh
         - -c
         - hostname && sleep 10 && git clone $giturl && cp repo/* /var/lib/jenkins && sleep 60
         containerName: jenkins
       failurePolicy: ignore
     ....
     ....```
 
  Deploy the changes using `oc rollout latest jenkins` 
  
  *Option 2:* Using oc patch from client.
  create run.sh script and store it in a repo. run.sh contains the below code
  ```sh
  oc patch dc/jenkins --patch '{"spec":{"strategy":{"rollingParams":{"post":{"failurePolicy": "ignore","execNewPod":{"containerName":"jenkins","command":["/bin/sh","-c","hostname && sleep 10 && git clone $giturl && cp repo/* /var/lib/jenkins && sleep 60"]}}}}}}'
  ```
  `source run.sh` to run the script to perform a patch operation 
  `oc rollout latest jenkins` to deploy the latest dc 
  
4. or Use rsync to copy the files as it is time process
5. Can we make use of config maps? Investigate further.. 

*ref:* https://blog.openshift.com/using-post-hook-to-initialize-a-database/


# 04/16/2018

*Failures:*
1. Jenkins persistent is having issues and is going down frequently . So opted for ephermal jenkins image with a volume mount of 100Gi
2. Created a config map for saxon lic and added that as part of the deployment config yaml file,upon rollout deployment was getting failed. Tried troublesshooting but couldn't trace out the issue. The same worked with Persistent image
3. Java/maven is not configured and also no DSL plugin is installed.
4. The idea to install plugins is to create a DSL plugin binary DSL.hpi file from source code and then use jenkins cli upload it to plugins.
5. When tried to install maven using configrue jenkins, maven is not getting installed. So had to manaullay install the maven using oc rsync to copy the binary from local machine to pod and extract it, set maven home and path settings
6. Even java is not configured properly. Need to setup this




# 04/17/2018

Identify if the openshift installed is using  Docker registry, or an OpenShift image stream.


Notes: 

Most of the times, not able to launch the jenkins url and after sometime it automatically gets up . We don't see any pod/service/route status as down . Find out how to fix this issue

and also 
 need to create a new mount point to store  .m2 folder.

 dont have access to get the volumes `oc get pv`   
 
 Which volume should be used to create a new mount. How to get the list of volumes and which one we should opt for.
 
 
# 04/18/2018

## How to install s2i
    Headover to https://github.com/openshift/source-to-image/releases , download and extract the suitable package.


## Error cloning from private git repository 
./s2i github-url .   (In the folder where docker file is located)

Failed with error to clone the code from github.

*Steps to allow openshift to clone the code from github private repositories*

https://blog.openshift.com/private-git-repositories-part-2a-repository-ssh-keys/


## S2i commmands:

1. `s2i create <name-of-the-future-builder-image> <s2i-folder-name>` .  This will generate a new folder structure for building a new customized builder image with name  <name-of-the-future-builder-image> as shown below. *EX: s2i create lighttpd-centos7 s2i-lighttpd* .
```
s2i-lighttpd/

Dockerfile – This is a standard Dockerfile where we’ll define the builder image
Makefile – a helper script for building and testing the builder image
test/
  run – test script, testing if the builder image works correctly
  test-app/ – directory for your test application
.s2i/bin/
  assemble – script responsible for building the application
  run – script responsible for running the application
  save-artifacts – script responsible for incremental builds, covered in a future article
  usage – script responsible for printing the usage of the builder image


```
2. `s2i build test/test-app/ <name-of-the-future-builder-image> <Output-image-name>`. Run this from test/test-app folder(test this on local) where your application source code is placed locally, in this case index.html is placed in this location. If stored on github provide the url as shown at setp 4. 
3. `docker run -p 8080:8080  <Output-image-name>`.
4. `s2i build application-github-url builder-image <Output-image-name>` This will create a new image named <Output-image-name>
5. `docker run -P <Output-image-name>` to run the docker image 



*ref:*
https://blog.openshift.com/create-s2i-builder-image/
https://github.com/openshift/source-to-image/blob/master/examples/nginx-centos7/README.md


### Role of assemble script and Run script:
Assemble:
 1. Converts source code into runnable application
 2. Downloads dependenice
 3. Injects configuration 

Run: 
 1. Responsible for starting the application in final image
 2. S2I will set this as a RUN command in final image

Save-extract-scripts:
1. Extracts resuable dependncy information from previos applicaiton image


pending
https://github.com/openshift/jenkins#installing-using-s2i-build


### Creating a builder image using s2i.
1. Create a docker file with base image 
[Docker file](https://github.com/manoharendla/RoughBook/blob/master/Dockerfile)

2. Create assemble and run scripts in .s2i/bin folder 
     assemble script, usually its a build script and  is triggered when s2i build command is invoked with githuburl containing application code and build image. When the build command is invoked, the application source code is cloned by default to /tmp/src or can be cloned to a defined location using --destination=<PATH> , so the assemble script performs the step to copy the source code from /tmp/src to some other defined location.
   
    Run script is excuted when container startsup and is contains runnbale commands to start the servci
    
    
    


# 23/04/2018

Jobs need maven release plugin to work  but maven release plugin has more internal dependencies
I tried to create a jenkins job that can clone the source code of the plugin repo and run mvn clean install to create plugin binary in .hpi format .

using rysnc copied the binary to Jenkins plugin folder and upon restart of jenkins plugin reflected in jenkins plugin and worked well (tried for checkamrx plugin). 


Anlaysing on making use of s2i to create a customized build. Worked on a sample lighttpd example . Created a build and executed using oc new-app 

# 24/04/2018
Created a builder image  in my host machine using docker. How would I push that image to our internal docker registry with IP adress (172.**.**.** :5000) so that I can view that image in openshift dasboard and consume this image to create an application.

using oc-new image name deployment is failing with different errors each time. 


# 25/04/2018
oc export all --as-template=<template_name>
oc export -h

## Create a build config

buildConfig.yaml
```
{
  "kind": "BuildConfig",
  "apiVersion": "v1",
  "metadata": {
    "name": "ruby-20-centos7-build"
  },
  "spec": {
    "triggers": [
      {
        "type": "GitHub",
        "github": {
          "secret": "secret101"
        }
      }
    ],
    "source": {
      "type": "Git",
      "git": {
        "uri": "https://github.com/openshift/sti-ruby"
      }
    },
    "strategy": {
      "type": "Custom",
      "customStrategy": {
        "from": {
          "kind": "DockerImage",
          "name": "openshift/sti-image-builder"
        },
        "env": [
          {
            "name": "IMAGE_NAME",
            "value": "openshift/ruby-20-centos7"
          },
          {
            "name": "CONTEXT_DIR",
            "value": "/2.0/"
          }
        ],
        "exposeDockerSocket": true
      }
    },
    "output": {
      "to": {
        "kind": "ImageStreamTag",
        "name": "ruby-20-centos7:latest"
      }
    }
  }
}
```


`oc create -f buildConfig.yaml`
`oc start-build ruby-20-centos7-build`



