# CI/CD on OpenShift with Jenkins Pipelines

## Setup

First, install the CI/CD project and tools.  This is all ready to go with a single Kustomize command:
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