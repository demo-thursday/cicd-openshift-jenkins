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

> **_NOTE:_**  At this point, if you're wondering "Ok... what is Kustomize and how did I just create three OpenShift project, install and configure Jenkins, SonarQube, Nexus, two app enviornments, and builds with two `oc` commands?", then you might want to take a look at the [GitOps with Kustomize and Argo CD Demo]() from last week!

Finally, we need to grant the Jenkins service account access to our "DEV" and "UAT" environments so that it can roll out changes as part of the pipeline.  This could also have been done with plain yaml files and Kustomize, but it's important to call these commands out here to emphasize the Role Based Access Control that is ubiquitous across the platform.  If we want Jenkins in the *cicd* project to be able to deploy applications in other projects, then we need to grant the Jenkins service account access to those projects.

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

## Jenkinsfile and POM file

The `Jenkinsfile` that defines our pipeline is found in the root of the git repository.  So is the main application `pom.xml` file.

### Jenkinsfile

The `Jenkinsfile` for this application is found in the root of the `demo-thursday` branch: https://github.com/pittar/spring-petclinic/blob/demo-thursday/Jenkinsfile

The `Jenkinsfile` should look familiar to anyone who has used Jenkins in the past.  Since we are using the Jenkins master image that ships with OpenShift, it has a few things going for it:
1. It is fully integrated with OpenShift OAuth.
2. It has the [OpenShift Jenkins Pipeline plugin](https://github.com/openshift/jenkins-client-plugin) pre-installed, giving you access to a robust Domain Specific Language (DSL) to interact with your OpenShift cluster.  You will see this in action soon.

### POM file

The POM for this application can be found in the root of the `demo-thursday` branch: https://github.com/pittar/spring-petclinic/blob/demo-thursday/pom.xml

Maven is an extremely mature and robust Java build tool that is familiar to virtually every Java developer.  The majority of the `pom.xml` file in this application should look quite familiar to Java developers, but there are a few sections that are worth calling out:

#### Distribution Management

```
	<distributionManagement>
		<snapshotRepository>
			<id>nexus-snapshots</id>
			<url>${repository.snapshots}</url>
		</snapshotRepository>
		<repository>
			<id>nexus-releases</id>
			<url>${repository.releases}</url>
		</repository>
	</distributionManagement>
```

This block instructs Maven where the **snapshot** and **release** repositories are for your company/department.  This is often the same artifact repository server, but different paths within it.  For this demo, we will be using [Sonatype Nexus](https://www.sonatype.com/nexus-repository-oss) as our artifact repository.  Nexus offers separate **snapshot** and **release** repositories out of the box.

Also notice that the URLs are defined as variables.  This allows us to externalie that configuration in a generic Maven `settings.xml` file.  We will see this a bit later.

### Profiles

If you scroll way down in the POM file you will notice two [Maven build profiles](http://maven.apache.org/guides/introduction/introduction-to-profiles.html).  One for `quality` and one for `failsafe`.  We will only use the `quality` profile in this demo, but I left the `failsafe` one in place as an example of how you might run automated functional tests with a tool such as [Selenium Grid](https://www.selenium.dev/documentation/en/grid/).


... To be continued.


## Additional Resources

[Red Hat Community of Practice: Jenkins Slaves](https://github.com/redhat-cop/containers-quickstarts/tree/master/jenkins-slaves)

Jenkins resources - OpenShift 3.x
https://docs.openshift.com/container-platform/3.11/dev_guide/dev_tutorials/openshift_pipeline.html
https://docs.openshift.com/container-platform/3.11/using_images/other_images/jenkins.html
https://docs.openshift.com/container-platform/3.11/using_images/other_images/jenkins_slaves.html
https://docs.openshift.com/container-platform/3.11/dev_guide/migrating_applications/continuous_integration_and_deployment.html


Jenkins resources - OpenShift 4.x
https://docs.openshift.com/container-platform/4.3/openshift_images/using_images/images-other-jenkins.html
https://docs.openshift.com/container-platform/4.3/openshift_images/using_images/images-other-jenkins-agent.html
https://docs.openshift.com/container-platform/4.3/builds/understanding-image-builds.html
