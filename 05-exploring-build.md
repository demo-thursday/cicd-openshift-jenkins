# Exploring the Build

[Back: Settings.xml](04-settings-xml.md)

## Jenkins Agent Templates

It's important to be able to configure Jenkins to use a custom `settings.xml` file in order to configure Maven profiles, mirrors, properties, etc...  Luckily, this is straight forward to do with OpenShift.

Configuring different Jenkins agents is as simple as creating a `ConfigMap` with a specific annotation: `role=jenkins-slave`

You can see the `ConfigMap` used in this demo is one of the yaml files present in the `jenkins2` directory, specifically [jenkins-cm.yaml](https://github.com/demo-thursday/cicd-openshift-jenkins/blob/master/jenkins2/jenkins-cm.yaml).  Although the XML looks complicated, most of it is boiler plate and makes sense as you read through it. Highlights include:

```
      <volumes>
        <org.csanchez.jenkins.plugins.kubernetes.volumes.ConfigMapVolume>
          <mountPath>/home/jenkins/.m2</mountPath>
          <configMapName>maven-settings-cm</configMapName>
        </org.csanchez.jenkins.plugins.kubernetes.volumes.ConfigMapVolume>
        <org.csanchez.jenkins.plugins.kubernetes.volumes.PersistentVolumeClaim>
          <mountPath>/data/m2</mountPath>
          <claimName>maven-m2</claimName>
          <readOnly>false</readOnly>
        </org.csanchez.jenkins.plugins.kubernetes.volumes.PersistentVolumeClaim>
      </volumes>
```

This maps two volumes into the Jenkins Maven agent.  The first is another `ConfigMap`, this one contains the contents of our custom Maven `settings.xml` file into the agent's `.m2` directory.

The second volume is a *persistent volume claim* that will be used to store dependencies once they are pulled in from Nexus.  This will greatly speed up subsequent builds.  This persistent volume will be mapped to `/data/m2` in the Jenkins Maven agent.

The next snippet of interest sets the Nexus username and password as environment variables in the Jenkins Maven agent.  These values come from a *Secret*.  If you look at the [settings.xml](https://github.com/demo-thursday/cicd-openshift-jenkins/blob/master/jenkins2/settings.xml) file that Maven will use, you can see that it references these environment variables in order to upload artifacts to Nexus.

```
          <envVars>
            <org.csanchez.jenkins.plugins.kubernetes.model.SecretEnvVar>
              <key>MAVEN_SERVER_USERNAME</key>
              <secretName>nexus-secret</secretName>
              <secretKey>username</secretKey>
            </org.csanchez.jenkins.plugins.kubernetes.model.SecretEnvVar>
            <org.csanchez.jenkins.plugins.kubernetes.model.SecretEnvVar>
              <key>MAVEN_SERVER_PASSWORD</key>
              <secretName>nexus-secret</secretName>
              <secretKey>password</secretKey>
            </org.csanchez.jenkins.plugins.kubernetes.model.SecretEnvVar>
          </envVars>
```

Finally, you also need to specify the location of the Jenkins agent container image to use.  In this case, it is the stock Maven agent from the Red Hat image registry:

```
          <image>registry.redhat.io/openshift4/ose-jenkins-agent-maven:v4.3</image>
```

## Recap

So, what did we do here?
1) Created a `ConfigMap` with an annotation of `role=jenkins-slave` that contains an XML-based template.
2) Configure two volumes:
    * One to mount a custom `settings.xml` file contained in a `ConfigMap` into the Maven agent `.m2` directory.
    * One to mount a `PersistentVolumeClaim` to cache artifacts and CVE data between builds.
3) Injected Nexus username and password data from a `Secret` as environment variables.
4) Specified the container image to use for this Jenkins agent.





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
