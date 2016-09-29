Simple Versioning strategy for Continuous Delivery
=============

This document introduces a versioning strategy which integrates with a continuous delivery approach.

## Version number

Releases will be versioned with a unique identifier: `RELEASE.MAJOR.BUILDNR`. This is a mix of a static versioned defined by developers and a unique version defined by Jenkins, which let us define a completely automated flow which does not require any manual intervention.
- `RELEASE` version corresponds to the Release 
- `MAJOR` version corresponds to a new major update
- `BUILDNR` is provided by Jenkins
The `pom.xml` will only contain `RELEASE` and `MAJOR` versions and will be manually maintained by developers.
The concept of Maven snapshots will not be used - only releases with unique version numbers are deployed.
During implementation phase, `RELEASE` can be set to `0`


## Release process
Each commit is a potential release: If all tests are passed (Unit tests, Integration tests, Code quality gate) a release tag is pushed to git and the package is archived in Nexus.

This is an example of a basic deployment pipeline:
1. Developer defines `RELEASE`/`MAJOR` versions in `pom.xml` (optional) and pushes changes to git
2. Jenkins fetch the code from git and includes its builder number in `BUILDNR`
3. Jenkins executes unit & integration tests, perform static code analysis and builds the corresponding archive
4. If step 3 is successful, Jenkins creates a tag and archive the new version in Nexus
5. Jenkins deploys on DEV environment and executes sanity checks
6. If step 5 is successful, Jenkins deploys on SIT environment and executes sanity checks

## Branching strategy
All builds are done from the `master` branch.  
The use of feature branches are not recommended as they won't be built before being merged into master.
The concept of toggles is recommended to enable/disable features ongoing development - but those toggles should be deleted as soon as the feature is ready.

## Setup
Please find below an example of a simple set up using Maven and Jenkins.  

### Maven pom.xml
```
<version>0.1.${revision}</version>
<properties>
	<!-- If no revision property is passed in from the command line -->
       	<revision>0-SNAPSHOT</revision>
</properties>
```
And for tagging
```
<plugin>
	<artifactId>maven-scm-plugin</artifactId>
	<version>1.9.4</version>
	<configuration>
		<tag>${project.artifactId}-${project.version}</tag>
	</configuration>
</plugin>
```

### Jenkins build
Using Maven Release plugin to perform releases in Jenkins after each code push is quite impractical:
- Hard to recover from failures
- Many commits to git
- Several Maven cycles run

I suggest a much simpler way using a single Maven command:
```
mvn deploy scm:tag -Drevision=$BUILD_NUMBER
```
This will:
- Compile the code
- Perform tests defined in pom
- Generate an archive with the version `RELEASE.MAJOR.BUILDNR`
- Upload it on Nexus
If one of those steps fails, the process stop so no unstable version get archived on Nexus.

# References
https://axelfontaine.com/blog/dead-burried.html  
https://www.cloudbees.com/blog/new-way-do-continuous-delivery-maven-and-jenkins-pipeline  
http://martinfowler.com/articles/feature-toggles.html
