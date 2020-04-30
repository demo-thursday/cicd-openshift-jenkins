# Application Code

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

[Next: Customizing Jenkins](03-customizing-jenkins.md)