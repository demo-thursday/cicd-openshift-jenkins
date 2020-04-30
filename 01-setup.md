# Setup

## The Power of Kustomize

We will setup the CI/CD environment, as well as the application environments and builds with a couple of `oc` commands.

First we will us the `oc` command line tool to setup the CI/CD environment. This is all ready to go with a single Kustomize command:
```
$ oc apply -k overlays/cicd
```

Next, install the application environments and builds.  Again, this is done with Kustomize:
```
$ oc apply -k overlays/apps
```

> **_NOTE:_**  At this point, if you're wondering "Ok... what is Kustomize and how did I just create three OpenShift project, install and configure Jenkins, SonarQube, Nexus, two app enviornments, and builds with two `oc` commands?", then you might want to take a look at the [GitOps with Kustomize and Argo CD Demo](https://github.com/demo-thursday/gitops-kustomize-argocd) from last week!

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

## Recap

The first build will take some time as it has a lot of dependencies to download, as well as a vulnerability database to build.  Each build after this one will be much faster, since libraries will be cached and the vulnerability database will already be built.

What we have done here is:
1) Create a CI/CD environment that includes Jenkins, Nexus, and SonarQube.
2) Create DEV and UAT app environments.
3) Create Jenkins Pipeline build.
4) Configure role based access control to give Jenkins access to the new environments.

[Next: Application Code](02-application-code.md)