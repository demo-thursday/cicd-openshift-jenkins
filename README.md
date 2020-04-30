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

[Next: Setup the Environments](01-setup.md)