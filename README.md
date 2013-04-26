# Kitchensink HTML5 Mobile Functional Test
This project contains the functional tests for the [Kitchensink HTML5 Mobile Demo](https://github.com/jboss-jdf/jboss-as-quickstart/tree/master/kitchensink-html5-mobile) project on Android OS. The purpose of the project is to depict how to automate the testing of web applications with mobile support on Android OS using [Arquillian](http://arquillian.org/) testing platform and [Jenkins CI](http://jenkins-ci.org/).

The [Arquillian](http://arquillian.org/) testing platform is used to enable the testing automation. Arquillian integrates transparently with the testing framework which is JUnit in this case.

## Functional Test Content
The Kitchensink HTML5 Mobile Functional Test defines the three core aspects needed for the execution of an [Arquillian](http://arquillian.org/) test case:

- container — the runtime environment
- deployment — the process of dispatching an artifact to a container
- archive — a packaged assembly of code, configuration and resources

The container's configuration resides in the [Arquillian XML](https://github.com/tolis-e/continuous-integration-of-mobile-web-applications-on-android/blob/master/src/test/resources/arquillian.xml) configuration file while the deployment and the archive are defined in the [Deployments](https://github.com/tolis-e/continuous-integration-of-mobile-web-applications-on-android/blob/master/src/test/java/org/jboss/as/quickstarts/test/kitchensink/html5/mobile/demo/Deployments.java) file.

[Arquillian XML](https://github.com/tolis-e/continuous-integration-of-mobile-web-applications-on-android/blob/master/src/test/resources/arquillian.xml) is configured to recognize the Android Emulator using some variables which are placed into the build environment from the Jenkins [Android Emulator Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Android+Emulator+Plugin).

The test case is dispatched to the container's environment through coordination with ShrinkWrap, which is used to declaratively define a custom Java EE archive that encapsulates the test class and its dependent resources. Arquillian packages the ShrinkWrap defined archive at runtime and deploys it to the target container. It then negotiates the execution of the test methods and captures the test results using remote communication with the server. Finally, Arquillian undeploys the test archive.

The [POM](https://github.com/tolis-e/continuous-integration-of-mobile-web-applications-on-android/blob/master/pom.xml) file contains two profiles:

* arq-jboss-managed — managed container 
* arq-jboss-remote — remote container

By default the arq-jboss-remote (remote container) profile is active. An Arquillian remote container is a container whose lifecycle is NOT managed by Arquillian. In order to use the managed container profile, activate the corresponding profile in the [POM](https://github.com/tolis-e/continuous-integration-of-mobile-web-applications-on-android/blob/master/pom.xml) file and set the default="true" flag on the jboss-managed configuration in the [Arquillian XML](https://github.com/tolis-e/continuous-integration-of-mobile-web-applications-on-android/blob/master/src/test/resources/arquillian.xml) file instead of the jboss-remote configuration.

## How it works - Important Notes
The functional test used to test our mobile web application is using the [Arquillian](http://arquillian.org/) testing platform. The whole concept is based on a remote Web Driver approach. Each WebDriver command makes a RESTful HTTP request to an Android HTTP server using JSON. The remote server delegates the request to Android WebDriver and returns a response. The Android Server APK path is configured inside the [Arquillian XML](https://github.com/tolis-e/continuous-integration-of-mobile-web-applications-on-android/blob/master/src/test/resources/arquillian.xml) configuration file. 

Note that the address 127.0.0.1 on your development machine corresponds to the emulator's own loopback interface. For this reason the WAR file produced from the Mobile Web Application build procedure, should be deployed by Arquillian on an application server whose services are using a host or IP binding address different than 127.0.0.1. During my tests I've setup this address to be 192.168.0.1. Check the [Arquillian XML](https://github.com/tolis-e/continuous-integration-of-mobile-web-applications-on-android/blob/master/src/test/resources/arquillian.xml) file where the corresponding configuration resides. An alternative solution is to use the special address 10.0.2.2. This address is a special alias to your host loopback interface (i.e., 127.0.0.1 on your development machine). However it is strongly recommended to avoid hardcoding the host address inside the source code.

The [Jenkins Android Emulator](https://wiki.jenkins-ci.org/display/JENKINS/Android+Emulator+Plugin) plugin manages the emulator's lifecycle. The plugin is using the [Port Allocator](https://wiki.jenkins-ci.org/display/JENKINS/Port+Allocator+Plugin) plugin in order to create random ports so there isn't any need to manage or set the ADB ports manually. In addition it exposes some environment variables during the build procedure. One of these variables is the ANDROID_SERIAL environment variable which represents the AVD Serial Id and uniquely identifies each emulator. The ANDROID_SERIAL environment variable value is propagated to Arquillian and the corresponding configuration resides inside the [Arquillian XML](https://github.com/tolis-e/continuous-integration-of-mobile-web-applications-on-android/blob/master/src/test/resources/arquillian.xml) configuration file.

In case you have build slaves which are headless (e.g. servers without a graphical user interface), you can still run an Android Emulator even although, by default, the emulator does require a graphical environment. You have to untick the "Show emulator window" configuration option in your job configuration.  This is the equivalent of using the emulator's "-no-window" command-line option.

This example has been tested using:

* JBoss EAP 6.0
* Git (used vesrion 1.8.1.4)
* Jenkins CI (used version 1.512)
* Android SDK
* Maven (used Maven 3)
* JDK (used Open JDK 1.7.0)

## Steps to automate the procedure on [Jenkins CI](http://jenkins-ci.org/)

Clone the current repository and rename it to kitchensink-html5-mobile-test (it is not necessary to rename it - just the below instructions are based on that naming). In addition get the [Kitchensink HTML5 Mobile Demo](https://github.com/jboss-jdf/jboss-as-quickstart/tree/master/kitchensink-html5-mobile) project by cloning the [JBoss AS Quickstart](https://github.com/jboss-jdf/jboss-as-quickstart.git) repository.

Open a Web Browser and navigate to the Jenkins initial page.

Now you should see the Jenkin's first page:

![Jenkins First Page](https://raw.github.com/tolis-e/jenkins-mobile-web-app-android-img/master/jenkins_1.png)

Navigate to Manage Jenkins → Configure System:

![Jenkins Configure System](https://raw.github.com/tolis-e/jenkins-mobile-web-app-android-img/master/jenkins_2.png)

Setup the JDK and Maven:

![Jenkins Setup JDK and Maven](https://raw.github.com/tolis-e/jenkins-mobile-web-app-android-img/master/jenkins_3.png)

Setup Git and Android SDK:

![Jenkins Setup Git and Android SDK](https://raw.github.com/tolis-e/jenkins-mobile-web-app-android-img/master/jenkins_4.png)

Download and install the Jenkins Android Emulator plugin and the Git Plugin:

![Jenkins install the Jenkins Android Emulator plugin and the Git Plugin](https://raw.github.com/tolis-e/jenkins-mobile-web-app-android-img/master/jenkins_5.png)

![Jenkins install the Jenkins Android Emulator plugin and the Git Plugin](https://raw.github.com/tolis-e/jenkins-mobile-web-app-android-img/master/jenkins_6.png)

Now you should see this page on the screen:

![Jenkins plugins downloaded](https://raw.github.com/tolis-e/jenkins-mobile-web-app-android-img/master/jenkins_7.png)

Restart Jenkins so that the changes are applied.

Create a new job on Jenkins. This job will build - produce the WAR file which corresponds to our [kitchensink-html5-mobile](https://github.com/jboss-jdf/jboss-as-quickstart/tree/master/kitchensink-html5-mobile) mobile web application:

![Jenkins setup job](https://raw.github.com/tolis-e/jenkins-mobile-web-app-android-img/master/jenkins_8.png)

Setup the repository URL (instruct Jenkins to find the project):

![Jenkins setup repository URL](https://raw.github.com/tolis-e/jenkins-mobile-web-app-android-img/master/jenkins_9.png)

Setup the Build Triggers, Build configuration. For convenience I have configured the SCM poll to be every minute but this is something that you can adjust according your desire:

![Jenkins setup build triggers and configuration](https://raw.github.com/tolis-e/jenkins-mobile-web-app-android-img/master/jenkins_10.png)

Save the job's configuration. Execute the job by clicking Build Now and verify that the WAR file was produced.

Now let's create the job which executes our functional test using a selected AVD. As mentioned above the AVD lifecycle is managed through the Jenkins Android Emulator plugin.

![Jenkins setup job](https://raw.github.com/tolis-e/jenkins-mobile-web-app-android-img/master/jenkins_15.png)

Setup the repository URL:

![Jenkins setup repository URL](https://raw.github.com/tolis-e/jenkins-mobile-web-app-android-img/master/jenkins_16.png)

Setup the Build Triggers, Build configuration. For convenience I have configured the SCM poll to be every minute but this is something that you can adjust according your desire. You can also tick the Build after other projects are built option and select the job which creates the WAR file.

![Jenkins setup build triggers and configuration](https://raw.github.com/tolis-e/jenkins-mobile-web-app-android-img/master/jenkins_17.png)

Setup the emulator options. You can either select an existing emulator or configure a new one. In case you have build slaves which are headless (e.g. servers without a graphical user interface), you can still run an Android Emulator even although, by default, the emulator does require a graphical environment. You have to untick the "Show emulator window" configuration option in your job configuration. Obviously when executing the test on a headless build machine, it is executed much faster.

![Jenkins emulator options](https://raw.github.com/tolis-e/jenkins-mobile-web-app-android-img/master/jenkins_18.png)

Save the job's configuration and execute by clicking Build Now.

## Development approach/methodologies
The development approach is driven from the desire to decouple the testing algorithmic steps / scenarios from the implementation which is tied to a specific DOM structure. For that reason the Page Objects and Page Fragments patterns are used. The Page Objects pattern is used to encapsulate the tested page's structure into one class which contains all the page's parts together with all methods which you will find useful while testing it. The Page Fragments pattern encapsulates parts of the tested page into reusable pieces across all your tests.

## Functional Test Execution
The purpose of the project is to depict how to automate the testing of mobile web applications on Android OS using [Arquillian](http://arquillian.org/) testing platform and [Jenkins CI](http://jenkins-ci.org/). However, it is feasible to execute the functional test as a standalone test (without using Jenkins and the Android Emulator Plugin) after performing some modifications. More specifically, in a such case you have to configure the [Arquillian XML](https://github.com/tolis-e/continuous-integration-of-mobile-web-applications-on-android/blob/master/src/test/resources/arquillian.xml) accordingly so that an Android Emulator starts automatically (avdName, apiLevel, abi properties).

The execution of the functional test as a standalone test is done through maven:

    mvn test    

## Documentation

* [Arquillian Guides](http://arquillian.org/guides/)
