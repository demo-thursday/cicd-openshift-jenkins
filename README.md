# CI/CD with Jenkins, SonarQube and Sonatype Nexus

**Original Demo Date: 2020-04-23**

## Intro

Continuous Integration and Continuous Delivery (CI/CD) is the engine that runs a development shop.  A mature CI/CD pipeline greatly improves the productivity of a development team and quality of the code they produce.  A few core responsibilities of a modern CI/CD pipline are:
* Compile code
* Run unit tests
* Report on quality and security issues in the code base
* Archive compiled artifacts
* Deploy an application through a chain of environments.

This demo will use Maven, Jenkins, Sonatype Nexus, and SonarQube as part of a CI/CD pipeline that will:
* Compile application code
* Run unit tests
* Run quality reports to surface any code quality, security, or vulnerability issues
* Archive the Java artifact
* Build a new container image for the application
* Deploy the application to the DEV, then UAT environments

## Setup

First we will us the `oc` command line tool to setup the CI/CD environment. This is all ready to go with a single Kustomize command:
```
$ oc apply -k overlays/cicd
```

Next, install the application environments and builds.  Again, this is done with Kustomize:
```
$ oc apply -k overlays/apps
```

Finally, we need to grant the Jenkins service account access to our "DEV" and "UAT" environments so that it can roll out changes as part of the pipeline.

```
$ oc policy add-role-to-user admin system:serviceaccount:cicd:jenkins -n petclinic-dev
$ oc policy add-role-to-user admin system:serviceaccount:cicd:jenkins -n petclinic-uat
```

We also need to allow the `petclinic` service accounts from DEV and UAT to be able to pull images that reside in the **CICD** project:
```
$ oc policy add-role-to-user system:image-puller system:serviceaccount:petclinic-dev:petclinic -n cicd
$ oc policy add-role-to-user system:image-puller system:serviceaccount:petclinic-uat:petclinic -n cicd
```

Finally... Start your Jenkins pipeline!  You can find this in the `cicd` project in OpenShift.  Click on **Builds** then, **Start build** from the kabab menu at the end of the `petclinic-jenkins-pipeline` line.

## Application Code

The Spring Boot app we will be building and deploying can be found here on the **demo-thursday** branch:
https://github.com/pittar/spring-petclinic/tree/demo-thursday

## Additional Resources

[Red Hat Community of Practice: Jenkins Slaves](https://github.com/redhat-cop/containers-quickstarts/tree/master/jenkins-slaves)

Jenkins resources - OpenShift 3.x
https://docs.openshift.com/container-platform/3.11/using_images/other_images/jenkins.html
https://docs.openshift.com/container-platform/3.11/dev_guide/dev_tutorials/openshift_pipeline.html
https://docs.openshift.com/container-platform/3.11/using_images/other_images/jenkins.html
https://docs.openshift.com/container-platform/3.11/using_images/other_images/jenkins_slaves.html
https://docs.openshift.com/container-platform/3.11/dev_guide/migrating_applications/continuous_integration_and_deployment.html


Jenkins resources - OpenShift 4.x