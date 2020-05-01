# Maven settings.xml

[Back: Customizing Jenkins](03-customizing-jenkins.md)

An important aspect of standardizing your Maven builds is to have common settings, mirrors, and profilies in a `settings.xml` file.  This also helps to keep the POM files in your applications more generic and allows you to offload sensitive information (such as your Nexus password) to a file that isn't kept in sourcde control.

In the [previous step](03-customizing-jenkins.md), we setup the Maven agent template in a `ConfigMap`.  Now we will create another `ConfigMap` to hold our `settings.xml` file.  This time, however, we will also use *Kustomize* to create the `ConfigMap` from a standard XML file.

## Generating a ConfigMap with Kustomize

First, let's take a look at the `kustomization.yaml` file that is in the **Jenkins2** base directory.  It can be found at [/base/Jenkins2/kustomization.yaml](https://github.com/demo-thursday/cicd-openshift-jenkins/blob/master/base/jenkins2/kustomization.yaml).

At the bottom of the file, you will find the instructions to generate a `ConfigMap` based on a list of files (in this case, only one):

```
generatorOptions:
  labels:
    app: jenkins
    app.kubernetes.io/component: maven-settings-cm
    app.kubernetes.io/instance: jenkins
    app.kubernetes.io/part-of: jenkins
  disableNameSuffixHash: true
configMapGenerator:
- name: maven-settings-cm
  files:
  - settings.xml
```

This block of yaml is pretty straight forward in what it does, but it's worth some explanation anyhow.
1) labels: A list of labels that will get added to the generated `ConfigMap`
2) disableSuffixHash: By default, Kustomize will add a suffix has to the name of the generated `ConfigMap` in order to ensure there are no duplicates.  We do **not** want a suffix appended to our file.
3) name: The name of the generated `ConfigMap`.  In this case, the name will be *maven-settings-cm*.
4) files: The list of files to be included in the `ConfigMap`.  Each will be it's own entry in the `data` portion of the resulting `ConfigMap`.

As you can tell by the last line, the generator will be looking for a file named `settings.xml` in the same directory as the `kustomization.yaml` file.  Let's take a look at it next.

## Settings.xml

In the same directory, you will find the [settings.xml file](https://github.com/demo-thursday/cicd-openshift-jenkins/blob/master/base/jenkins2/settings.xml) that the Jenkins Maven agent will use.  There isn't much magic here, it's really just a standard Maven settings file.  You can see how it ties all of our tools and configuration together.

For example, remember how we created a `PersistentVolumeClaim` to cache Maven resources and the OWASP CVE database?  That volume was mapped to the directory `/data/m2`.  In the settings file, you set this directory as the *local repository*.

```
<localRepository>/data/m2</localRepository>
```

We also need to give Maven the Nexus useername and password so that it can upload the artifacts we build to Nexus.  We already injected these values as environment variables in the previous step, so now we set them like this:

```
    <servers>
      <server>
        <id>nexus-releases</id>
        <username>${env.MAVEN_SERVER_USERNAME}</username>
        <password>${env.MAVEN_SERVER_PASSWORD}</password>
      </server>
      <server>
        <id>nexus-snapshots</id>
        <username>${env.MAVEN_SERVER_USERNAME}</username>
        <password>${env.MAVEN_SERVER_PASSWORD}</password>
      </server>
    </servers>
```

We also want Maven to pull dependencies from our Nexus server and not directly from the interent (e.g. Maven Central).  We do this by setting a *mirror* in this file as well.

```
    <mirrors>
      <mirror>
        <id>internal-repository</id>
        <name>Internal Nexus Maven Proxy</name>
        <url>http://nexus:8081/content/groups/public</url>
        <mirrorOf>external:*</mirrorOf>
      </mirror>
    </mirrors>
```

Finally, if you look at the bottom of the file you will see a few *profiles* that set properties.  Some of these are to setup SonarQube, some are for Maven, and they are all activated!

## Recap

In this `settings.xml` file we have:
* Specified the directory where artifacts are cached between builds.
* Configured Nexus to be our maven mirror for all artifacts.
* Configured Nexus as the *snapshot* and *release* repository.
* Configured and activated a number of *profiles*.

[Next: Exploring the Build](05-exploring-build.md)
