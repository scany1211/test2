

https://www.jfrog.com/confluence/display/JFROG/Scripted+Pipeline+Syntax

# Creating an Artifactory Server Instance

==To upload or download files to and from your Artifactory server, you need to create an Artifactory server instance in your Pipeline script.==

If your Artifactory server is already defined in Jenkins, you only need its server ID which can be obtained  under Manage | Configure System.

Then, to create your Artifactory server instance, add the following line to your script:
`def server = Artifactory.server 'my-server-id'`

If your Artifactory is not defined in Jenkins you can still create it as follows:
`def server = Artifactory.newServer url: 'artifactory-url', username: 'username', password: 'password'`

You can also user Jenkins Credential ID instead of username and password:
`def server = Artifactory.newServer url: 'artifactory-url', credentialsId: 'ccrreeddeennttiiaall'`

You can modify the server object using the following methods:
// If Jenkins is configured to use an http proxy, you can bypass the proxy when using this Artifactory server: 
server.bypassProxy = true
// If you're using username and password:
server.username = 'new-user-name'
server.password = 'new-password'
// If you're using Credentials ID:
server.credentialsId = 'ccrreeddeennttiiaall'
// Configure the connection timeout (in seconds).
// The default value (if not configured) is 300 seconds: 
server.connection.timeout = 300

Use variables

We recommend using variables rather than plain text to specify the Artifactory server details.

# Uploading and Downloading Files

To upload or download files you first need to create a spec which is a JSON file that specifies which files should be uploaded or downloaded and the target path.

For example:

``` groovy
def downloadSpec = """{
 "files": [
  {
      "pattern": "bazinga-repo/*.zip",
      "target": "bazinga/"
    }
 ]
}""" 
```

 

The above spec specifies that all ZIP files in the bazinga-repo Artifactory repository should be downloaded into the bazinga directory on your Jenkins agent file system.

"files" is an array

Since the "files" element is an array, you can specify several patterns and corresponding targets in a single download spec.

To download the files, add the following line to your script:

server.download spec: downloadSpec

Uploading files is very similar. The following example uploads all ZIP files that include froggy in their names into the froggy-files folder in the bazinga-repo Artifactory repository.

``` groovy
 def uploadSpec = """{
  "files": [
    {
      "pattern": "bazinga/*froggy*.zip",
      "target": "bazinga-repo/froggy-files/"
    }
 ]
}"""
server.upload spec: uploadSpec
```




You can read about using File Specs for downloading and uploading files here.

If you'd like the build to fail, in case no files are uploaded or downloaded, add the failNoOp argume to the upload or download methods as follows:

` server.download spec: downloadSpec, failNoOp: true
server.upload spec: uploadSpec, failNoOp: true `

# Setting and Deleting Properties on Files in Artifactory

When uploading files to Artifactory using the server.upload method, you have the option of setting properties on the files. The properties are defined as part of the File Spec sent to the method. These properties can be later used to filter and download those files.

In some cases, you may want want to set properties on files that are already in Artifactory. Here properties to be set are sent outside the File Spec. To define which file to set the properties, a File Spec is used. Here's an example:
def setPropsSpec = """{
 "files": [
  {
       "pattern": "my-froggy-local-repo/dir/*.zip",
       "props": "filter-by-this-prop=yes"
    }
 ]
}"""


server.setProps spec: setPropsSpec, props: “p1=v1;p2=v2”

In the above example, the p1 and p2 properties will be set with the v1 and v2 values respectively. The properties will be set on all the zip files inside the dir directory under the my-froggy-local-repo repository. Only files which already have the filter-by-this-prop property set to yes will be affected.

The failNoOp argument is optional. Setting it to true will cause the job to fail, if no properties have been set. Here's how you use it:
server.setProps spec: setPropsSpec, props: “p1=v1;p2=v2”, failNoOp: true

The server.deleteProps method can be used to delete properties from files in Artifactory, Like the server.setProps method, it also uses a File Spec. The only difference between the two methods, is that for deleteProps, we specify only the names of the properties to delete. The names are comma separated. The properties values should not be specified. The failNoOp argument is optional. Setting it to true will cause the job to fail, if no properties have been deleted.

Here's an example:
server.deleteProps spec: deletePropsSpec, props: “p1,p2,p3”, failNoOp: true

# Publishing Build-Info to Artifactory

If you're not yet familiar with the build-info entity, please read about it here.

Both the download and upload methods return a build-info object which can be published to Artifactory as shown in the following examples:
def buildInfo1 = server.download downloadSpec
def buildInfo2 = server.upload uploadSpec
buildInfo1.append buildInfo2
server.publishBuildInfo buildInfo1
def buildInfo = Artifactory.newBuildInfo()
server.download spec: downloadSpec, buildInfo: buildInfo
server.upload spec: uploadSpec, buildInfo: buildInfo
server.publishBuildInfo buildInfo

You also have the option of customising the build-info module names, used for the download and upload operations. Here's how you do it:
def buildInfo1 = server.download spec: downloadSpec, module: 'my-custom-build-info-module-name'
def buildInfo2 = server.upload spec: uploadSpec, module: 'my-custom-build-info-module-name'


Getting Dependencies and Artifacts from the Build-Info

The build-info instance stores the build-info locally. It can be later published to Artifactory. As shown above, the server.upload method adds artifacts to the build-info and the server.download method adds dependencies to the build-info.

You have the option of getting the list of dependencies and artifacts stored in the build-info instance. You can do this at any time, before or after the build-info is published to Artifactory. In the following example, ww first check if there are any dependencies stored in the build-info, and if there are, we access the properties of one of the dependencies. We then do the same for artifacts.
if (buildInfo.getDependencies().size() > 0) {
    def localPath = buildInfo.getDependencies()[0].getLocalPath()
    def remotePath = buildInfo.getDependencies()[0].getRemotePath()
    def md5 = buildInfo.getDependencies()[0].getMd5()
    def sha1 = buildInfo.getDependencies()[0].getSha1()
}

if (buildInfo.getArtifacts().size() > 0) {
    def localPath = buildInfo.getArtifacts()[0].getLocalPath()
    def remotePath = buildInfo.getArtifacts()[0].getRemotePath()
    def md5 = buildInfo.getArtifacts()[0].getMd5()
    def sha1 = buildInfo.getArtifacts()[0].getSha1()
}
Modifying the Default Build Name and Build Number

You can modify the default build name and build number set by Jenkins. Here's how you do it:
def buildInfo = Artifactory.newBuildInfo()
buildInfo.name = 'super-frog'
buildInfo.number = 'v1.2.3'
...
server.publishBuildInfo buildInfo

If you're setting the build name or number as shown above, it is important to do so before you're using this buildInfo instance for uploading files. 

Here's the reason for this: The server.upload method also tags the uploaded files with the build name and build number (using the build.name and build.number properties). Setting a new build name or number after the files are already uploaded to Artifactory, will not update the properties attached to the files.


# Setting the Build-Info Project

If the build-info should be published as part of a specific JFrog project, you should set the project key on the build-info instance before it is published to Artifactory. Here's how you do this:
def buildInfo = Artifactory.newBuildInfo()
buildInfo.project = 'my-jfrog-project-key'
...
server.publishBuildInfo buildInfo


Capturing Environment Variables

To set the Build-Info object to automatically capture environment variables while downloading and uploading files, add the following to your script:
def buildInfo = Artifactory.newBuildInfo()
buildInfo.env.capture = true

By default, environment variables names which include "password", "psw", "secret", "token", or "key" (case insensitive) are excluded and will not be published to Artifactory.

You can add more include/exclude patterns with wildcards as follows:
def buildInfo = Artifactory.newBuildInfo()
buildInfo.env.filter.addInclude("*a*")
buildInfo.env.filter.addExclude("DONT_COLLECT*")

Here's how you reset to the include/exclude patterns default values:
buildInfo.env.filter.reset()

You can also completely clear the include/exclude patterns:
buildInfo.env.filter.clear()

To collect environment variables at any point in the script, use:
buildInfo.env.collect()

You can get the value of an environment variable collected as follows:
value = buildInfo.env.vars['env-var-name']


Triggering Build Retention

To trigger build retention when publishing build-info to Artifactory, use the following method:
buildInfo.retention maxBuilds: 10
buildInfo.retention maxDays: 7

To have the build retention also delete the build artifacts, add the deleteBuildArtifacts with true value as shown below:
buildInfo.retention maxBuilds: 10, maxDays: 7, doNotDiscardBuilds: ["3", "4"], deleteBuildArtifacts: true

It is possible to trigger an asynchronous build retention. To do this, add the async argument with true as shown below:
buildInfo.retention maxBuilds: 10, deleteBuildArtifacts: true, async: true


Collecting Build Issues

The build-info can include the issues which were handled as part of the build. The list of issues is automatically collected by Jenkins from the git commit messages. This requires the project developers to use a consistent commit message format, which includes the issue ID and issue summary, for example:
HAP-1364 - Replace tabs with spaces
The list of issues can be then viewed in the Builds UI in Artifactory, along with a link to the issue in the issues tracking system.
The information required for collecting the issues is provided through a JSON configuration. This configuration can be provided as a file or as a JSON string.
Here's an example for issues collection configuration.
{
    "version": 1,
    "issues": {
        "trackerName": "JIRA",
        "regexp": "(.+-[0-9]+)\\s-\\s(.+)",
        "keyGroupIndex": 1,
        "summaryGroupIndex": 2,
        "trackerUrl": "http://my-jira.com/issues",
        "aggregate": "true",
        "aggregationStatus": "RELEASED"
    }
}

Configuration file properties:
Version	The schema version is intended for internal use. Do not change!
trackerName	The name (type) of the issue tracking system. For example, JIRA. This property can take any value.
trackerUrl	The issue tracking URL. This value is used for constructing a direct link to the issues in the Artifactory build UI.
keyGroupIndex	

The capturing group index in the regular expression used for retrieving the issue key. In the example above, setting the index to "1" retrieves HAP-1364 from this commit message:

HAP-1364 - Replace tabs with spaces
summaryGroupIndex	

The capturing group index in the regular expression for retrieving the issue summary. In the example above, setting the index to "2" retrieves the sample issue from this commit message:

HAP-1364 - Replace tabs with spaces

aggregate
	Set to true, if you wish all builds to include issues from previous builds.

aggregationStatus
	If aggregate is set to true, this property indicates how far in time should the issues be aggregated. In the above example, issues will be aggregated from previous builds, until a build with a RELEASE status is found. This status can be set when a build is promoted in Artifactory.
regexp	

A regular expression used for matching the git commit messages. The expression should include two capturing groups - for the issue key (ID) and the issue summary. In the example above, the regular expression matches the commit messages as displayed in the following example:

HAP-1364 - Replace tabs with spaces

Here's how you set issues collection in the pipeline script.
server = Artifactory.server 'my-server-id'

config = """{
    "version": 1,
    "issues": {
        "trackerName": "JIRA",
        "regexp": "(.+-[0-9]+)\\s-\\s(.+)",
        "keyGroupIndex": 1,
        "summaryGroupIndex": 2,
        "trackerUrl": "http://my-jira.com/issues",
        "aggregate": "true",
        "aggregationStatus": "RELEASED"
    }
}"""

buildInfo.issues.collect(server, config)
server.publishBuildInfo buildInfo

To help you get started, we recommend using the Github Examples.


Aggregating Builds

The build-info published to Artifactory can include multiple modules representing different build steps. As shown earlier in this section, you just need to pass the same buildInfo instance to all the methods that need it (server.upload for example).

What happens however if your build process runs on multiple machines or it is spread across different time periods? How do you aggregate all the build steps into one build-info?

You have the option of creating and publishing a separate build-info for each segment of the build process, and then aggregating all those published builds into one build-info. The end result is one build-info which references other, previously published build-infos.

In the following example, our pipeline script publishes two build-info instances to Artifactory:
def buildInfo1 = Artifactory.newBuildInfo()
buildInfo1.name = 'my-app-linux'
buildInfo1.number = '1'
server.publishBuildInfo buildInfo1

def buildInfo2 = Artifactory.newBuildInfo()
buildInfo2.name = 'my-app-windows'
buildInfo2.number = '1'
server.publishBuildInfo buildInfo2

At this point, we have two build-infos stored in Artifactory. Now let's create our final build-info, which references the previous two.
def finalBuildInfo = Artifactory.newBuildInfo()
server.buildAppend(finalBuildInfo, 'my-app-linux', '1')
server.buildAppend(finalBuildInfo, 'my-app-windows', "1")
server.publishBuildInfo finalBuildInfo

'finalBuildInfo' includes two modules, which reference 'my-app-linux' and 'my-app-windows'.

Build Promotion and Build scanning with Xray are currently not supporting aggregated builds.


Promoting Builds in Artifactory

To promote a build between repositories in Artifactory, define the promotion parameters in a promotionConfig object and promote that. For example:
def promotionConfig = [
    // Mandatory parameters
    'targetRepo'         : 'libs-prod-ready-local',

    // Optional parameters
     
    // The build name and build number to promote. If not specified, the Jenkins job's build name and build number are used
    'buildName'          : buildInfo.name,
    'buildNumber'        : buildInfo.number,
    // Only if this build is associated with a project in Artifactory, set the project key as follows.
    'project': 'my-project-key',
    // Comment and Status to be displayed in the Build History tab in Artifactory
    'comment'            : 'this is the promotion comment',
    'status'             : 'Released',
    // Specifies the source repository for build artifacts.
    'sourceRepo'         : 'libs-staging-local',
    // Indicates whether to promote the build dependencies, in addition to the artifacts. False by default
    'includeDependencies': true,
    // Indicates whether to copy the files. Move is the default
    'copy'               : true,
    // Indicates whether to fail the promotion process in case of failing to move or copy one of the files. False by default.
    'failFast'           : true
]

// Promote build
server.promote promotionConfig
Allowing Interactive Promotion for Published Builds

The 'Promoting Builds in Artifactory' section in this article describes how your Pipeline script can promote builds in Artifactory. In some cases however, you'd like the build promotion to be performed after the build finished. You can configure your Pipeline job to expose some or all the builds it publishes to Artifactory, so that they can be later promoted interactively using a GUI. Here's how the Interactive Promotions looks like:


When the build finishes, the promotion window will be accessible by clicking on the promotion icon, next to the build run.

Here's how you do this.

First you need to create a 'promotionConfig' instance, the same way it is shown in the 'Promoting Builds in Artifactory' section.

Next, you can use it, to expose a build for interactive promotion as follows:
Artifactory.addInteractivePromotion server: server, promotionConfig: promotionConfig, displayName: "Promote me please"

You can add as many builds as you like, by using the method multiple times. All the builds added will be displayed in the promotion window. 

The 'addInteractivePromotion' method expects the following arguments:

    "server" is the Artifactory on which the build promotions is done. You can create the server instance as described in the beginning of this article. 
    "promotionConfig" includes the promotion details. The "Promoting Builds in Artifactory" section describes how to create a promotionConfig instance.
     "displayName" is an optional argument. If you add it, the promotion window will display it instead of the build name and number.

Maven Builds with Artifactory

Maven builds can resolve dependencies, deploy artifacts and publish build-info to Artifactory.

The Jenkins Artifactory Plugin integrates with maven through the "install" goal. It is therefore mandatory to include "install" as one of the goals when using the Artifactory API for maven. There's no need to add the "deploy" goal, since the deployment to Artifactory does not use it.

To run Maven builds with Artifactory from your Pipeline script, you first need to create an Artifactory server instance, as described at the beginning of this article.

Here's an example:
def server = Artifactory.server 'my-server-id'

The next step is to create an Artifactory Maven Build instance:
def rtMaven = Artifactory.newMavenBuild()

Now let's define where the Maven build should download its dependencies from. Let's say you want the release dependencies to be resolved from the 'libs-release' repository and the snapshot dependencies from the 'libs-snapshot' repository. Both repositories are located on the Artifactory server instance you defined above. Here's how you define this, using the Artifactory Maven Build instance we created:
rtMaven.resolver server: server, releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot'

Now let's define where our build artifacts should be deployed to. Once again, we define the Artifactory server and repositories on the 'rtMaven' instance:
rtMaven.deployer server: server, releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local'

By default, all the build artifacts are deployed to Artifactory. In case you want to deploy only some artifacts, you can filter them based on their names, using the 'addInclude' method. In the following example, we are deploying only artifacts with names that start with 'frog'
rtMaven.deployer.artifactDeploymentPatterns.addInclude("frog*")

You can also exclude artifacts from being deployed. In the following example, we are deploying all artifacts, except for those that are zip files:
rtMaven.deployer.artifactDeploymentPatterns.addExclude("*.zip")

And to make things more interesting, you can combine both methods. For example, to deploy all artifacts with names that start with 'frog', but are not zip files, do the following:
rtMaven.deployer.artifactDeploymentPatterns.addInclude("frog*").addExclude("*.zip")

If you'd like to add custom properties to the deployed artifacts, you can do that as follows:
rtMaven.deployer.addProperty("status", "in-qa").addProperty("compatibility", "1", "2", "3")

By default, 3 threads will be used for uploading the maven artifacts. You can modify the number of threads used as follows:
rtMaven.deployer.threads = 6

In some cases, you want to disable artifacts deployment to Artifactory or make the deployment conditional. Here's how you do it:
rtMaven.deployer.deployArtifacts = false

In case you'd like to use the Maven Wrapper for this build, add this:
rtMaven.useWrapper = true

To select a Maven installation for our build, we should define a Maven Tool through Jenkins Manage, and then, set the tool name as follows:
rtMaven.tool = 'maven tool name'

Instead of using rtMaven.tool, you can set the path to the Maven installation directory using the MAVEN_HOME environment variable as follows:
env.MAVEN_HOME = '/tools/apache-maven-3.3.9'

Here's how you define Maven options for your build:
rtMaven.opts = '-Xms1024m -Xmx4096m'

In case you'd like Maven to use a different JDK than your build agent's default, no problem.
Simply set the JAVA_HOME environment variable to the desired JDK path (the path to the directory above the bin directory, which includes the java executable).
Here's you do it:
env.JAVA_HOME = 'full/path/to/JDK'

OK, we're ready to run our build. Here's how we define the pom file path (relative to the workspace) and the Maven goals. The deployment to Artifactory is performed during the 'install' phase:
def buildInfo = rtMaven.run pom: 'maven-example/pom.xml', goals: 'clean install'

The above method runs the Maven build. Notice that the method returns a buildInfo instance, which can be later published to Artifactory.

In some cases though, you'd like to pass an existing buildInfo instance to be used by the build. This can come in handy is when you want to set custom build name or build number on the build-info instance, or when you'd like to aggregate multiple builds into the same build-info instance. Here's how you pass an existing build-info instance to the rtMaven.run method:


rtMaven.run pom: 'maven-example/pom.xml', goals: 'clean install', buildInfo: existingBuildInfo

By default, the build artifacts will be deployed to Artifactory, unless rtMaven.deployer.deployArtifacts property was set to false. This can come in handy in two cases:

    You do not wish to publish the artifacts.
    You'd like to publish the artifacts later down the road. Here's how you can publish the artifacts at a later stage:

rtMaven.deployer.deployArtifacts buildInfo

Make sure to use the same buildInfo instance you received from the rtMaven.run method. Also make sure to run the above method on the same agent that ran the rtMaven.run method, because the artifacts were built and stored on the file-system of this agent.

By default, Maven uses the local Maven repository inside the .m2 directory under the user home. In case you'd like Maven to create the local repository in your job's workspace, add the -Dmaven.repo.local=.m2 system property to the goals value as shown here:
def buildInfo = rtMaven.run pom: 'maven-example/pom.xml', goals: 'clean install -Dmaven.repo.local=.m2'

What about the build-info?

The build-info has not yet been published to Artifactory, but it is stored locally in the 'buildInfo' instance returned by the 'run' method. You can now publish it to Artifactory as follows:
server.publishBuildInfo buildInfo

You can also merge multiple buildInfo instances into one buildInfo instance and publish it to Artifactory as one build, as described in the Publishing Build-Info to Artifactory section in this article.
Gradle Builds with Artifactory

Gradle builds can resolve dependencies, deploy artifacts and publish build-info to Artifactory.

Gradle Compatibility

The minimum Gradle version supported is 4.10


To run Gradle builds with Artifactory from your Pipeline script, you first need to create an Artifactory server instance, as described at the beginning of this article.
Here's an example:
def server = Artifactory.server 'my-server-id'

The next step is to create an Artifactory Gradle Build instance:
def rtGradle = Artifactory.newGradleBuild()

Now let's define where the Gradle build should download its dependencies from. Let's say you want the dependencies to be resolved from the 'libs-release' repository, located on the Artifactory server instance you defined above. Here's how you define this, using the Artifactory Gradle Build instance we created:
rtGradle.resolver server: server, repo: 'libs-release'

Now let's define where our build artifacts should be deployed to. Once again, we define the Artifactory server and repositories on the 'rtGradle' instance:
rtGradle.deployer server: server, repo: 'libs-release-local'
If you're using gradle to build a project, which produces maven artifacts, you also have the option of defining two deployment repositories - one repository will be used for snapshot artifacts and one for release artifacts. Here's how you define it:
rtGradle.deployer server: server, releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local'

Gradle allows customizing the list of deployed artifacts by defining publications as part fo the Gradle build script. Gradle publications are used to group artifacts together. You have the option of defining which of the defined publications Jenkins should use. Only the artifacts grouped by these publications will be deployed to Artifactory. If you do not define the publications, a default publication, which includes the list of the produced artifacts by a java project will be used. Here's how you define the list of publications:
rtGradle.deployer.publications.add("mavenJava").add("ivyJava")

If you'd like to deploy the artifacts from all the publications defined in the gradle script, you can set the "ALL_PUBLICATIONS" string as follows.
rtGradle.deployer.publications.add("ALL_PUBLICATIONS")

By default, all the build artifacts are deployed to Artifactory. In case you want to deploy only some artifacts, you can filter them based on their names, using the 'addInclude' method. In the following example, we are deploying only artifacts with names that start with 'frog'
rtGradle.deployer.artifactDeploymentPatterns.addInclude("frog*")

You can also exclude artifacts from being deployed. In the following example, we are deploying all artifacts, except for those that are zip files:
rtGradle.deployer.artifactDeploymentPatterns.addExclude("*.zip")

And to make things more interesting, you can combine both methods. For example, to deploy all artifacts with names that start with 'frog', but are not zip files, do the following:
rtGradle.deployer.artifactDeploymentPatterns.addInclude("frog*").addExclude("*.zip")

If you'd like to add custom properties to the deployed artifacts, you can do that as follows:
rtGradle.deployer.addProperty("status", "in-qa").addProperty("compatibility", "1", "2", "3")
By default, 3 threads will be used for uploading the artifacts to Artifactory. You can modify the number of threads used as follows:


rtGradle.deployer.threads = 6

In some cases, you want to disable artifacts deployment to Artifactory or make the deployment conditional. Here's how you do it:
rtGradle.deployer.deployArtifacts = false

In case the "com.jfrog.artifactory" Gradle Plugin is already applied in your Gradle script, we need to let Jenkins know it shouldn't apply it. Here's how we do it:
rtGradle.usesPlugin = true

In case you'd like to use the Gradle Wrapper for this build, add this:
rtGradle.useWrapper = true

If you don't want to use the Gradle Wrapper, and set a Gradle installation instead, you should define a Gradle Tool through Jenkins Manage, and then, set the tool name as follows:
rtGradle.tool = 'gradle tool name'

In case you'd like Gradle to use a different JDK than your build agent's default, no problem.
Simply set the JAVA_HOME environment variable to the desired JDK path (the path to the directory above the bin directory, which includes the java executable).
Here's you do it:
env.JAVA_HOME = 'path to JDK'

OK, looks like we're ready to run our Gradle build. Here's how we define the build.gradle file path (relative to the workspace) and the Gradle tasks. The deployment to Artifactory is performed as part of the 'artifactoryPublish' task:
def buildInfo = rtGradle.run rootDir: "projectDir/", buildFile: 'build.gradle', tasks: 'clean artifactoryPublish'

The above method runs the Gradle build. Notice that the method returns a buildInfo instance, which can be later published to Artifactory.

In some cases though, you'd like to pass an existing buildInfo instance to be used by the build. This can come in handy is when you want to set custom build name or build number on the build-info instance, or when you'd like to aggregate multiple builds into the same build-info instance. Here's how you pass an existing build-info instance to the rtGradle.run method:
rtGradle.run rootDir: "projectDir/", buildFile: 'build.gradle', tasks: 'clean artifactoryPublish', buildInfo: existingBuildInfo

By default, the build artifacts will be deployed to Artifactory, unless the rtGradle.deployer.deployArtifacts property was set to false. This can come in handy in two cases:

    You do not wish to publish the artifacts.
    You'd like to publish the artifacts later down the road. Here's how you can publish the artifacts at a later stage:

rtGradle.deployer.deployArtifacts buildInfo

Make sure to use the same buildInfo instance you received from the rtGradle.run method. Also make sure to run the above method on the same agent that ran the rtGradle.run method, because the artifacts were built and stored on the file-system of this agent.

What about the build-info?
The build-info has not yet been published to Artifactory, but it is stored locally in the 'buildInfo' instance returned by the 'run' method. You can now publish it to Artifactory as follows:
server.publishBuildInfo buildInfo

You can also merge multiple buildInfo instances into one buildInfo instance and publish it to Artifactory as one build, as described in the Publishing Build-Info to Artifactory section in this article.

That's it! We're all set.

The rtGradle instance supports additional configuration APIs. You can use these APIs as follows:
def rtGradle = Artifactory.newGradleBuild()
// Deploy Maven descriptors to Artifactory:
rtGradle.deployer.deployMavenDescriptors = true
// Deploy Ivy descriptors (pom.xml files) to Artifactory:
rtGradle.deployer.deployIvyDescriptors = true

// The following properties are used for Ivy publication configuration.
// The values below are the defaults.

// Set the deployed Ivy descriptor pattern:
rtGradle.deployer.ivyPattern = '[organisation]/[module]/ivy-[revision].xml'
// Set the deployed Ivy artifacts pattern:
rtGradle.deployer.artifactPattern = '[organisation]/[module]/[revision]/[artifact]-[revision](-[classifier]).[ext]'
// Set mavenCompatible to true, if you wish to replace dots with slashes in the Ivy layout path, to match the Maven layout:
rtGradle.deployer.mavenCompatible = true

You also have the option of defining default values in the gradle build script. Read more about it here.
Maven Release Management with Artifactory

With the Artifactory Pipeline DSL you can easily manage and run a release build for your Maven project by following the instructions below:

First, clone the code from your source control:
git url: 'https://github.com/eyalbe4/project-examples.git'

If the pom file has a snapshot version, Maven will create snapshot artifacts, because the pom files include a snapshot version (for example, 1.0.0-SNAPSHOT).
Since you want your build to create release artifacts, you need to change the version in the pom file to 1.0.0.
To do that, create a mavenDescriptor instance, and set the version to 1.0.0:
def descriptor = Artifactory.mavenDescriptor()
descriptor.version = '1.0.0'

If the project's pom file is not located at the root of the cloned project, but inside a sub-directory, add it to the mavenDescriptor instance:
descriptor.pomFile = 'maven-example/pom.xml'

In most cases, you want to verify that your release build does not include snapshot dependencies. The are two ways to do that.

The first way, is to configure the descriptor to fail the build if snapshot dependencies are found in the pom files. In this case, the job will fail before the new version is set to the pom files.
Here's how you configure this:
descriptor.failOnSnapshot = true

The second way to verify this is by using the hasSnapshots method, which returns a boolean true value if snapshot dependencies are found:
def snapshots = descriptor.hasSnapshots()
if (snapshots) {
    ....
}

That's it. Using the mavenDescriptor as it is now will change the version inside the root pom file. In addition, if the project includes sub-modules with pom files, which include a version, it will change them as well.
Sometimes however, some sub-modules should use different release versions. For example, suppose there's one module whose version should change to 1.0.1, instead of 1.0.0. The other modules should still have their versions changed to 1.0.0. Here's how to do that:
descriptor.setVersion "the.group.id:the.artifact.id", "1.0.1"

The above setVersion method receives two arguments: the module name and its new release version. The module name is composed of the group ID and the artifact ID with a colon between them.
Now you can transform the pom files to include the new versions:
descriptor.transform()

The transform method changed the versions on the local pom files.
You can now build the code and deploy the release Maven artifacts to Artifactory as described in the "Maven Builds with Artifactory" section in this article. 

The next step is to commit the changes made to the pom files to the source control, and also tag the new release version in the source control repository. If you're using git, you can use the git client installed on your build agent and run a few shell commands from inside the Pipeline script to do that.
The last thing you'll probably want to do is to change the pom files version to the next development version and commit the changes. You can do that again by using a mavenDescriptor instance.
Python Builds with Artifactory

Python builds can resolve dependencies, deploy artifacts and publish build-info to Artifactory. To run Python builds with Artifactory start by following these steps, to make sure your Jenkins agent is ready:

    Make sure Python is installed on the build agent and that the python command is in the PATH.
    Install pip. You can use the Pip Documentation and also Installing packages using pip and virtual environments.
    Make sure wheel and setuptools are installed. You can use the Installing Packages Documentation.
    Validate that the build agent is ready by running the following commands from the terminal:

Output Python version:
> python --version

Output pip version:
> pip --version

Verify wheel is installed:
> wheel -h

Verify setuptools is installed:
> pip show setuptools

Verify that virtual-environment is activated:
> echo $VIRTUAL_ENV

In your Pipeline script, you first need to create an Artifactory server instance, as described in the Creating an Artifactory Server Instance section.

Here's an example:
def server = Artifactory.server 'my-server-id'

The next step is to create an Artifactory a Pip Build instance:
def rtPip = Artifactory.newPipBuild()

Now let's define where the build should download its dependencies from. We set the Artifactory server instance we created earlier and the repository name on the resolver:
rtPip.resolver repo: 'pypi-virtual', server: server

It is mostly recommended to run pip commands inside a virtual environment, to achieve isolation for the pip build. To follow this recommendation, create a shell command which sets up a virtual environment. Let's save this shell command in a variable, we'll soon use:
def virtual_env_activation = "source /Users/myUser/venv-example/bin/activate"

Now we can download our project's dependencies as follows:
def buildInfo = rtPip.install args: "-r python-example/requirements.txt", envActivation: virtual_env_activation

You'll need to adjust value of the args argument in the above command, to match your Python project. Notice that we sent the command for activating the virtual env as the value of the envActivation argument. This argument is optional.

The above method returns a buildInfo instance. If we already have a buildInfo instance we'd like to reuse, we can alternatively send the buildInfo as an argument as shown below. Read the Publishing Build-Info to Artifactory section for more details.
rtPip.install buildInfo: buildInfo, args: "-r python-example/requirements.txt", envActivation: virtual_env_activation

Jenkins spawns a new java process during this step's execution.
You have the option of passing any java args to this new process, by passing the javaArgs argument:


def buildInfo = rtPip.install args: "-r python-example/requirements.txt", javaArgs: '-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005'

In most cases, your build also produces artifacts. The artifacts produced can be deployed to Artifactory using the server.upload method, as described in the Uploading and Downloading Files section in this article.

You can now publish the build-info to Artifactory as described in the Publishing Build-Info to Artifactory  section

Examples

It is highly recommended to use these example projects as a reference, when setting up yout first pip build. 


NuGet and .NET Core Builds with Artifactory

The Artifactory Plugin's integration with the NuGet and .NET Core clients allow build resolve dependencies, deploy artifacts and publish build-info to Artifactory.

Depending on the client you'd like to use, please make sure either the nuget or dotnet clients are included in the build agent's PATH.

To run NuGet or .NET Core builds with Artifactory from your Pipeline script, you first need to create an Artifactory server instance, as described in the Creating an Artifactory Server Instance section.
Here's an example:
def server = Artifactory.server 'my-server-id'

The next step is to create a NuGet or .NET Core Build instance:
def rtBuild = Artifactory.newDotnetBuild()

// OR

def rtBuild = Artifactory.newNugetBuild()

By default, the build uses NuGet API protocol v2. If you'd like to use v3, set it on the build instance as follows.
rtBuild.setApiProtocol 'v3'

Now let's define where the build should download its dependencies from. We set the Artifactory server instance we created earlier and the repository name on the resolver:
rtBuild.resolver repo: 'nuget-remote', server: server

Now we can download our project's NuGet dependencies, using either the NuGet or .NET Core clients:
def buildInfo = rtBuild.run args: 'restore ./src/GraphQL.sln'

The above method returns a buildInfo instance. If we already have a buildInfo instance we'd like to reuse, we can alternatively send the buildInfo as an argument as shown below. Read the Publishing Build-Info to Artifactory section for more details.
rtBuild.run buildInfo: buildInfo, args: 'restore ./src/GraphQL.sln'

You also have the option of customising the build-info module name associated with this build. You do this as follows:
def buildInfo = rtBuild.run args: 'restore ./src/GraphQL.sln', module: 'my-build-info-module-name'

Jenkins spawns a new java process during this step's execution.
You have the option of passing any java args to this new process, by passing the javaArgs argument:


def buildInfo = rtBuild.run args: 'restore ./src/GraphQL.sln', javaArgs: '-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005'

In most cases, your build also produces artifacts. The artifacts can be NuGet packages, DLL files or any other type of artifact. The artifacts produced can be deployed to Artifactory using the server.upload method, as described in the Uploading and Downloading Files section in this article.

You can now publish the build-info to Artifactory as described in the Publishing Build-Info to Artifactory  section




NPM Builds with Artifactory

NPM builds can resolve dependencies, deploy artifacts and publish build-info to Artifactory. To run NPM builds with Artifactory from your Pipeline script, you first need to create an Artifactory server instance, as described in the Creating an Artifactory Server Instance section.
Here's an example:
def server = Artifactory.server 'my-server-id'

The next step is to create an Artifactory NPM Build instance:
def rtNpm = Artifactory.newNpmBuild()

Now let's define where the NPM build should download its dependencies from. We set the Artifactory server instance we created earlier and the repository name on the resolver:
rtNpm.resolver server: server, repo: 'npm-virtual'

The build uses the npm executable to install (download the dependencies) and publish. By default, Jenkins uses the npm executable present in the agent's PATH. You can also reference a tool defined in Jenkins, and set the script to use it as follows:
// Set the name of a tool defined in Jenkins configuration
rtNpm.tool = 'nodejs-tool-name'
// or set the tool as an environment variable
env.NODEJS_HOME = "${tool 'nodejs-tool-name'}"
// or set a path to the NodeJS home directory (not the npm executable)
env.NODEJS_HOME = 'full/path/to/the/nodeJS/home'
// or
nodejs(nodeJSInstallationName: 'nodejs-tool-name') {
    // Only in this code scope, the npm defined by 'nodejs-tool-name' is used.
}

Now we can download our project's npm dependencies. The following method runs npm install behind the scenes:
def buildInfo = rtNpm.install path: 'npm-example'

You can also add npm flags or arguments as follows:
def buildInfo = rtNpm.install path: 'npm-example', args: '--verbose'

The npm ci command is also supported the same way:
def buildInfo = rtNpm.ci path: 'npm-example'

The above methods return a buildInfo instance. If we already have a buildInfo instance we'd like to reuse, we can alternatively send the buildInfo as an argument as shown below. Read the Publishing Build-Info to Artifactory section for more details.
rtNpm.install path: 'npm-example', buildInfo: my-build-info

You also have the option of customising the build-info module name associated with this build. You do this as follows:
def buildInfo = rtNpm.install path: 'npm-example', module: 'my-build-info-module-name'

Jenkins spawns a new java process during this step's execution.
You have the option of passing any java args to this new process, by passing the javaArgs argument:
def buildInfo = rtNpm.install path: 'npm-example', javaArgs: '-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005'

The action of publishing the NPM package to Artifactory is very similar. We start by defining the deployer:
rtNpm.deployer server: server, repo: 'npm-local'

The following method will do two things: package the code (by running npm pack) and publish it to Artifactory:
def buildInfo = rtNpm.publish path: 'npm-example'

Similarly to the install method, the following is also supported:
rtNpm.publish path: 'npm-example', buildInfo: my-build-info

You also have the option of customising the build-info module name associated with this operation. You do this as follows:
def buildInfo = rtNpm.publish path: 'npm-example', module: 'my-build-info-module-name'

Jenkins spawns a new java process during this step's execution.
You have the option of passing any java args to this new process, by passing the javaArgs argument:
def buildInfo = rtNpm.publish path: 'npm-example', javaArgs: '-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005'

You can now publish the build-info to Artifactory as described in the Publishing Build-Info to Artifactory  section
Go Builds with Artifactory

While building your Go projects, Jenkins can resolve dependencies, deploy artifacts and publish build-info to Artifactory.


Please make sure that the go client is included in the build agent's PATH.


To run Go builds with Artifactory from your Pipeline script, you first need to create an Artifactory server instance, as described in the Creating an Artifactory Server Instance section.
Here's an example:
def server = Artifactory.server 'my-server-id'

The next step is to create an Artifactory Go Build instance:
def rtGo = Artifactory.newGoBuild()

Now let's define where the Go build should download its dependencies from. We set the Artifactory server instance we created earlier and the repository name on the resolver:
rtGo.resolver server: server, repo: 'go-virtual'

Now let's build the project. Here's how we do it:
def buildInfo = rtGo.run path: 'path/to/the/project/root', args: 'build'

Please make sure that the go client is included in the build agent's PATH.

The above method returns a buildInfo instance. If we already have a buildInfo instance we'd like to reuse, we can alternatively send the buildInfo as an argument as shown below. Read the Publishing Build-Info to Artifactory section for more details.
rtGo.run path: 'path/to/the/project/root', args: 'build', buildInfo: my-build-info

You also have the option of customising the build-info module name associated with this build. You do this as follows:
def buildinfo = rtGo.run path: 'path/to/the/project/root', args: 'build', module: 'my-build-info-module-name'

Now that the project is built, you can pack and publish it to Artifactory as a Go package. We start by defining the deployer:
rtGo.deployer server: server, repo: 'go-local'

The following method will do two things: package the code and publish it to Artifactory:
def buildInfo = rtGo.publish path: 'golang-example/hello', version: '1.0.0'

If you already have a buildInfo instance configured, you can pass it as an argument as follows:
rtGo.publish buildInfo: buildInfo, path: 'path/to/the/project/root', version: '1.0.0'

You also have the option of customising the build-info module name associated with this build. You do this as follows:
def buildinfo = rtGo.publish path: 'path/to/the/project/root', version: '1.0.0', module: 'my-build-info-module-name'

You can now publish the build-info to Artifactory as described in the Publishing Build-Info to Artifactory section
Conan Builds with Artifactory

Conan is a C/C++ Package Manager. The Artifactory Pipeline DSL includes APIs that make it easy for you to run Conan builds, using the Conan Client installed on your build agents. Here's what you need to do before you create your first Conan build job with Jenkins:

1, Install the latest Conan Client on your Jenkins build agent. Please refer to the Conan documentation for installation instructions.

2. Add the Conan Client executable to the PATH environment variable on your build agent, to make sure Jenkins is able to use the client.

3. Create a Conan repository in Artifactory as described in the Conan Repositories Artifactory documentation.

OK. Let's start coding your first Conan Pipeline script.

We'll start by creating an Artifactory server instance, as described at the beginning of this article.
Here's an example:
def server = Artifactory.server 'my-server-id'

Now let's create a Conan Client instance
def conanClient = Artifactory.newConanClient()

When creating the Conan client, you can also specify the Conan user home directory as shown below:
def conanClient = Artifactory.newConanClient userHome: "conan/my-conan-user-home"

We can now configure our new conanClient instance by adding an Artifactory repository to it. In our example, we're adding the 'conan-local' repository, located in the Artifactory server, referenced by the server instance we obtained:

String remoteName = conanClient.remote.add server: server, repo: "conan-local"

The above method also accepts the the following optional arguments:

force: true -  Adding this argument will make the conan client not to raise an error. If an existing remote exists with the provided name.

verifySSL: false - Adding this argument will make the conan client skip the validation of SSL certificates.

As you can see in the above example, the conanClient.remote.add method returns a string variable - remoteName. What is this 'remoteName' variable? What is it for?

Well, a 'Conan remote' is a repository, which can be used to download dependencies from and upload artifacts to. When we added the 'conan-local' Artifactory repository to our Conan Client, we actually added a Conan remote. The 'remoteName' variable contains the name of the new Conan remote we added.

OK. We're ready to start running Conan commands. You'll need to be familiar with the Conan commands syntax, exposed by the Conan Client to run the commands. You can read about the commands syntax in the Conan documentation.

Let's run the first command:

def buildInfo1 = conanClient.run command: "install --build missing"

The 'conanClient.run' method returns a buildInfo instance, that we can later publish to Artifactory. If you already have a buildInfo instance, and you'd like the 'conanClient.run' method to aggregate the build-info to it, you can also send the buildInfo instance to the run command as and an argument as shown below:


conanClient.run command: "install --build missing", buildInfo: buildInfo

The next thing we want to do is to use the Conan remote we created. For example, let's upload our artifacts to the Conan remote. Notice how we use the 'remoteName' variable we got earlier, when building the Conan command:

String command = "upload * --all -r ${remoteName} --confirm"
conanClient.run command: command, buildInfo: buildInfo

We can now publish the the buildInfo to Artifactory, as described in the Publishing Build-Info to Artifactory section. For example:

server.publishBuildInfo buildInfo

Docker Builds with Artifactory
General

The Jenkins Artifactory Plugin supports a Pipeline DSL that allows collecting and publishing build-info to Artifactory for your Docker builds. To setup your Jenkins build agents to collect build-info for your Docker builds, please refer to the setup instructions.
Working with Docker Daemon Directly

The Jenkins Artifactory Plugin supports working with the docker daemon directly through its REST API. Please make sure ti set up Jenkins to work with docker and Artifasctory as mentioned in the previous section.
// Create an Artifactory server instance, as described above in this article:
def server = Artifactory.server 'my-server-id'

// Create an Artifactory Docker instance. The instance stores the Artifactory credentials and the Docker daemon host address.
// If the docker daemon host is not specified, "/var/run/docker.sock" is used as a default value (the host argument should not be specified in this case).
def rtDocker = Artifactory.docker server: server, host: "tcp://<daemon IP>:<daemon port>"

// Jenkins spawns a new java process during this step's execution.
// You have the option of passing any java args to this new process when creating the rtDocker instance.
// Here's how you do this:
// def rtDocker = Artifactory.docker server: server, javaArgs: '-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005'

// Pull a docker image from Artifactory.
def buildInfo = rtDocker.pull '<artifactory-docker-registry-url>/hello-world:latest', '<source-artifactory-repository>'
// If you already have a buildInfo instance, you can pass it as an argument to the rtDocker.pull method as follows:
// rtDocker.pull '<artifactory-docker-registry-url>/hello-world:latest', '<source-artifactory-repository>', buildInfo

// Attach custom properties to the published artifacts:
rtDocker.addProperty("project-name", "docker1").addProperty("status", "stable")

// Push a docker image to Artifactory (here we're pushing hello-world:latest). The push method also expects
// Artifactory repository name (<target-artifactory-repository>).
// Please make sure that <artifactoryDockerRegistry> is configured to reference the <target-artifactory-repository> Artifactory repository. In case it references a different repository, your build will fail with "Could not find manifest.json in Artifactory..." following the push.
def buildInfo = rtDocker.push '<artifactory-docker-registry-url>/hello-world:latest', '<target-artifactory-repository>'
// If you already have a buildInfo instance, you can pass it as an argument to the rtDocker.push method as follows:
// rtDocker.push '<artifactory-docker-registry-url>/hello-world:latest', '<target-artifactory-repository>', buildInfo

// Publish the build-info to Artifactory:
server.publishBuildInfo buildInfo
Using Kaniko

The rtDocker.createDockerBuild method allows collecting build-info for docker images that were published to Artifactory using Kaniko. See our kaniko project example on GitHub to learn how to do this.
Using Jib

The rtDocker.createDockerBuild method allows collecting build-info for docker images that were published to Artifactory using the JIB Maven Plugin. See our maven-jib-example on GitHub to learn how to do this. Since this example also runs maven using the Artifactory pipeline APIs, we also recommend referring to the Maven Builds with Artifactory section included in this documentation page.
Scanning Builds with JFrog Xray

From version 2.9.0, Jenkins Artifactory Plugin is integrated with JFrog Xray through JFrog Artifactory allowing you to have build artifacts scanned for vulnerabilities and other issues. If issues or vulnerabilities are found, you may choose to fail a build job or perform other actions according to the Pipeline script you write. This integration requires JFrog Artifactory v4.16 and above and JFrog Xray v1.6 and above. 

You may scan any build that has been published to Artifactory. It does not matter when the build was published, as long as it was published before triggering the scan by JFrog Xray.

The following instructions show you how to configure your Pipeline script to have a build scanned.

First, for Xray to scan builds, you need to configure a Watch with the right filters that specify which artifacts and vulnerabilities should trigger an alert, and set a Fail Build Job Action for that Watch. You can read more about CI/CD integration with Xray here.

Now you can configure your Jenkins Pipeline job to scan the build.. Start by creating a scanConfig instance with the build name and build number you wish to scan:
def scanConfig = [
    'buildName'     : 'my-build-name',
    'buildNumber'   : '17',
    // Only if this build is associated with a project in Artifactory, set the project key as follows.
    'project'       : 'my-project-key'
  ]

If you're scanning a build which has already been published to Artifactory in the same job, you can use the build name and build number stored on the buildInfo instance you used to publish the build. For example:
server.publishBuildInfo buildInfo
def scanConfig = [
    'buildName'      : buildInfo.name,
    'buildNumber'    : buildInfo.number,
    // Only if this build is associated with a project in Artifactory, set the project key as follows.
    'project'.       : 'my-project-key'
  ]

Before you trigger the scan, there's one more thing you need to be aware of. By default, if the Xray scan finds vulnerabilities or issues in the build that trigger an alert, the build job will fail. If you don't want the build job to fail, you can add the 'failBuild' property to the scanConfig instance and set it to 'false' as shown here:
def scanConfig = [
    'buildName'      : buildInfo.name,
    'buildNumber'    : buildInfo.number,
     // Only if this build is associated with a project in Artifactory, set the project key as follows.
    'project'        : 'my-project-key',
    'failBuild'      : false
  ]

OK, we're ready to initiate the scan. The scan should be initiated on the same Artifactory server instance, to which the build was published:
def scanResult = server.xrayScan scanConfig

That's it. The build will now be scanned. If the scan is not configured to fail the build job, you can use the scanResult instance returned from the xrayScan method to see some details about the scan.
For example, to print the result to the log, you could use the following code snippet:
echo scanResult as String

For more details on the integration with JFrog Xray and JFrog Artifactory to scan builds for issues and vulnerabilities, please refer to CI/CD Integration in the JFrog Xray documentation.
Managing Release Bundles
General

The Jenkins Artifactory Plugin exposes a set of pipeline APIs for managing and distributing Release Bundles. These APIs require version 2.0 or higher of JFrog Distribution. These APIs work with JFrog Distribution's REST endpoint, and not with Artifactory REST endpoint. It is therefore recommended to verify that JFrog Distribution is accessible from Jenkins through Jenkins | Manage | Configure System. The serverId value in all examples in this section should be replaced with the JFrog Platform ID you configured.

To make it easier to get started using the JFrog Distribution pipeline APIs, you can use the jfrog-distribution-example available here.

To use the APIs, you first need obtain a distribution instance using the ID configured in Jenkins | Manage | Configure System. Here's how you do this.
def jfrogInstance = JFrog.instance 'jfrog-instance-1'
def dsServer = jfrogInstance.distribution
Creating and Updating Release Bundles

The createReleaseBundle and updateReleaseBundle methods create and update a release bundle on JFrog Distribution. The methods accept the release bundle name and release bundle version to be created. The methods also accept a File Spec, which defines the files in Artifactory to be bundled into the release bundle. Let's start by creating the File Spec as follows.
def fileSpec = """{
    "files": [{
        "pattern": "libs-repo/release/*.zip"
    }]
}"""

Now let's create the release bundle as follows.
dsServer.createReleaseBundle name: 'release-bundle-1', version: '1.0.0', spec: fileSoec

The above createReleaseBundle method supports additional optional arguments as shown below.
dsServer.createReleaseBundle
    version: '1.0.0',
    spec: fileSoec
    // The default is "plain_text". The syntax for the release notes. Can be one of 'markdown', 'asciidoc', or 'plain_text'.
    releaseNotesSyntax: "markdown",
    // Optional. If set to true, automatically signs the release bundle version.
    signImmediately: true,
    // Optional. Path to a file describing the release notes for the release bundle version.
    releaseNotesPath: "path/to/release-notes",
    // Optional. The passphrase for the signing key.
    gpgPassphrase: "abc",
    // Optional. A repository name at the source Artifactory instance, to store release bundle artifacts in. If not provided, Artifactory will use the default one.
    storingRepo: "release-bundles-1",
    // Optional.
    description: "Some desc",
    // Optional. Path to a file with the File Spec content.
    specPath: "path/to/filespec.json",
    // Optional. Set to true to disable communication with JFrog Distribution.
    dryRun: true

The updateReleaseBundle method accepts the exact same arguments as the createReleaseBundle method.
Signing Release Bundles

Release bundles must be signed before they can be distributed. Here's how you sign a release bundle.
dsServer.createReleaseBundle
    name: "example-release-bundle",
    version: "1",
    // Optional GPG passphrase
    gpgPassphrase: "abc",
    // Optional repository name at the source Artifactory instance, to store release bundle artifacts in. If not provided, Artifactory will use the default one.
    storingRepo: "release-bundles-1"  
Distributing Release Bundles

To better control where the release bundle will be distributed to, you have the option of defining the distribution rules as follows.
def rules = """{
    "distribution_rules": [
    {
        "site_name": "*",
        "city_name": "*",
        "country_codes": ["*"]
    }
    ]
}"""

After making sure the release bundle is signed, you can distribute it as follows, by optionally using distribution rules.
dsServer.createReleaseBundle
    name: "example-release-bundle",
    version: "1",
    // Optional distribution rules
    distRules: rules,
    // Optional country codes. Cannot be used together with 'distRules'  
    countryCodes: ["001", "002"]
    // Optional site name. Cannot be used together with 'distRules'
    siteName: "my-site",
    // Optional city name. Cannot be used together with 'distRules'
    cityName: "New York",
    // Optional. If set to true, the response will be returned only after the distribution is completed.
    sync: true,
    // Optional. Set to true to disable communication with JFrog Distribution.
    dryRun: true
Deleting Release Bundles

Here's how you delete a release bundle.
dsServer.deleteReleaseBundle
    name: "example-release-bundle",
    version: "1",
    // Optional distribution rules
    distRules: rules,
    // Optional country codes. Cannot be used together with 'distRules'  
    countryCodes: ["001", "002"]
    // Optional site name. Cannot be used together with 'distRules'
    siteName: "my-site",
    // Optional city name. Cannot be used together with 'distRules'
    cityName: "New York",
    // Optional. If set to true, the response will be returned only after the deletion is completed.
    sync: true,
    // Optional. Set to true to disable communication with JFrog Distribution.
    dryRun: true


Build Triggers

The Artifactory Trigger allows a Jenkins job to be automatically triggered when files are added or modified in a specific Artifactory path. The trigger periodically polls Artifactory to check if the job should be triggered. You can read more about it here.

You have the option of defining the Artifactory Trigger from within your pipeline. Here's how you do it:

Start by creating an Artifactory server instance, as described at the beginning of this article.
Here's an example:
def server = Artifactory.server 'my-server-id'

Next, set the trigger as follows:
server.setBuildTrigger spec: "*/10 * * * *", paths: "generic-libs-local/builds/starship"

When a job is triggered following deployments to Artifactory, you can get the URL of the file in Artifactory which triggered the job. Here's how you get it:
def url = currentBuild.getBuildCauses('org.jfrog.hudson.trigger.ArtifactoryCause')[0]?.url

If the cause of the job triggering is different, the url value will be empty.