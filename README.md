**Description**

Use Jenkins to set up a CI/CD pipeline that will compile and test a Maven project and deploy it to a Tomcat Server.

**Background of the problem statement:**

You’re a DevOps engineer at Pied Piper, a software company that provides web and mobile app development services. Your team is tasked with building a web app for an online book shop named Bookzy. The developers have decided to use Springboot to generate the project, Maven for compilation, and Git as the Source Code Management system for the project. You’re required to set up a Jenkins pipeline that involves three different freestyle jobs for code compilation, testing, and deployment respectively. Your Pipeline has to poll the SCM nightly for commits, build the project, and trigger the test job if the build is stable. The test job has to run unit tests, publish the JUnit test report, and trigger the freestyle job for deployment. The deployment job is required to package the Maven project, deploy the war file to a Tomcat server, and notify you via email if deployment failed.

**You must use the following:**

- Java: To create the website
- Git: As a version control system for the program
- Jenkins: To create the build pipeline
- Spring boot: To create the Maven app
- Maven: To compile the program
- Tomcat: To host the website
- AWS EC2: To run Tomcat
- JUnit: To run tests and publish results
 

**The following requirements should be met:**

- The app should be built with Maven.
- The Tomcat server should allow remote deployment.
- The pipeline should consist of three freestyle jobs for compilation, test, and deployment respectively.



To set up a CI/CD pipeline using Jenkins for a Maven project that will be compiled, tested, and deployed to a Tomcat server, you need to follow these steps:

**Tomcat Server Configuration** : ```https://github.com/Pavithra-42/jenkins_tomcat_demo.git```

**Step 1: Create a Maven Project**
1. Go to Spring Initializr.
2. Select Maven Project, Java, and Spring Boot version.
3. Fill in Group and Artifact details.
4. Add dependencies as needed (e.g., Spring Web, Spring Data JPA).
5. Click on Generate to download the project zip file.
6. Extract the project and initialize a Git repository in it.
7. Push the project to your Git repository.
**NOTE**: Use this https://github.com/Pavithra-42/simple-java-maven-app.git as target/sure-fire in test job is expecting file structure.

**Step 2: Setup Jenkins**

**Job 1: Code Compilation**

1. Create a new freestyle project in Jenkins named compile.
2. Configure Source Code Management:
- Choose Git.
- Provide the Repository URL and credentials if required.
3. Build Triggers:
- Choose Poll SCM and set the schedule to ```H 1 * * *``` (nightly build at 1AM everyday).
4. Build Environment:
- Ensure Delete workspace before build starts is checked to avoid stale files.
5. Build:
- Add a build step: Invoke top-level Maven targets.
- Set the Goals to ```clean compile```.
6. Post-build Actions:
- Add Build other projects.
- Specify the next job (test-job).

**Job 2: Testing**

1. Create a new freestyle project in Jenkins named test.
2. Configure Source Code Management:
- Choose Git.
- Provide the Repository URL and credentials if required.
3. Build Triggers:
- Choose Build after other projects are built.
- Specify compile-job and set trigger to Stable.
4. Build:
- Add a build step: Invoke top-level Maven targets.
- Set the Goals to ```test```.
5. Post-build Actions:
- Add Publish JUnit test result report.
- Specify the Test Report XMLs path, usually **/target/surefire-reports/*.xml.
- Add Build other projects.
- Specify the next job (deploy-job).

**Job 3: Deployment**

1. Create a new freestyle project in Jenkins named deploy-job.
2. Configure Source Code Management:
- Choose Git.
- Provide the Repository URL and credentials if required.
3. Build Triggers:
- Choose Build after other projects are built.
- Specify ``test``` and set trigger to Stable.
4. Build:
- Add a build step: Invoke top-level Maven targets.
- Set the Goals to ```package```.
- Install ```Publish Over SSH``` plugin and configure this plugin in jenkins system configuration.
- Copy the tomcat server public key to authorized_keys and copy the tomcat server private_key to jenkins system configuration.
- Add another build step: Send files or execute commands over SSH.
- Configure the SSH server (AWS EC2 instance) and set the commands to deploy the WAR file to Tomcat. Example commands:
```
scp target/*.war user@ec2-instance:/path/to/tomcat/webapps/
ssh user@ec2-instance 'bash -s' <<'ENDSSH'
cd /path/to/tomcat/bin
./shutdown.sh
./startup.sh
ENDSSH
```
5. Post-build Actions:

Add Editable Email Notification:
- Set recipient email.
- Configure triggers for build success and failure.

