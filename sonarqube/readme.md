## Managing Technical Debt using VSTS and SonarQube 

## Overview

In this lab, you will be introduced to Technical Debt by configuring team build definitions to use SonarQube and  analyze the results.

<a href="https://www.sonarqube.org/">SonarQube</a> is an open source platform for continuous inspection of code quality to perform automatic reviews with static analysis of code to

- <a href="http://bit.ly/2AN9LIE">Detect Bugs
- Code Smells
- Security Vulnerabilities
- Centralize Quality</a>

## Pre-requisites

1. **Microsoft Azure Account:** You need a valid and active azure account for the labs

2. You need a **Visual Studio Team Services Account** and <a href="http://bit.ly/2gBL4r4">Personal Access Token</a>

## Setting up the VSTS project

1. Use <a href="http://bit.ly/2AWznna" target="_blank">VSTS Demo Data Generator</a> to provision a project on your VSTS account.

   ![](images/vstsdemogen.png)

2. Once the project is provisioned, select the URL to navigate to the project that you provisioned.

   <img src="images/vsts_demo.png" height="450px">

## Setting up the Environment

1. Click **Deploy To Azure** to provision SonarQube Server on Azure VM.

   <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fhsachinraj%2FAzurelabs%2Fmaster%2Fsonarqube%2Ftemplates%2Fazuredeploy.json"><img src="http://azuredeploy.net/deploybutton.png"></a>

   Provide the following parameters as shown.

   <table width="100%">
   <thead>
      <tr>
         <th width="50%"><b>Parameter Name</b></th>
         <th><b>Description</b></th>
         
      </tr>
   </thead>
   <tr>
      <td>SQ_VM_Name</td>
      <td>name of the VM where SonarQube will be installed</td>
      
   </tr>
   <tr>
      <td>SQ_DNS_NAME</td>
      <td><b>unique</b> dns name to be provided with the following pattern:- <b>^[a-z][a-z0-9-]{1,61}[a-z0-9]$</b> or it will throw an error. For ex: sonarqubedns</td>
      
   </tr>
   <tr>
      <td>SQ_VM_UserName</td>
      <td>local admin account for the SonarQube VM</td>
      
   </tr>
   <tr>
      <td>SQ_VM_UserPassword</td>
      <td>password for the SonarQube VM</td>
      
   </tr>
   <tr>
      <td>SQ_DBAdmin_UserName</td>
      <td>admin account for Azure SQL Server</td>
      
   </tr>
   <tr>
      <td>SQ_DBAdmin_Password</td>
      <td>password for Azure SQL Server</td>
      
   </tr>
   
   </table>

2. Once the deployment is successful, you will see the resources in Azure Portal.

   <img src="images/azure_resources.png">

3. Access the **SonarQube** portal by browsing the public address. The default port for SonarQube is 9000. Copy the DNS name from the VM in Azure Portal as shown and append :9000 at the end. The final URL will be **http://YOUR_DNS_NAME:9000**

   <img src="images/dns_name.png">

4. Login to the SonarQube Portal using the following credentials-    
   >**Username= admin, Password= admin**

   <img src="images/sonarqube_portal.png">


## Exercise 1: Configure SonarQube Server

To start code analysis, we need a project to be created in the sonarqube. The Projects area allows you to explore projects by multiple metrics in both the overall and leak persepectives. Here, we focus on an individual project.

1. Login to the sonarqube portal.

2. Click **Administration**, go to **Projects-->Management**.

   <img src="images/sonar_admin.png">

3. Create a project with its **Name** and **Key** as **MyShuttle**.

   <img src="images/project_creation.png">

## Exercise 2: Update Endpoint

During the project provisioning, a dummy endpoint would be created. We will update the endpoint with sonarqube details.

1. In VSTS, go to **Services** by clicking the gear icon, and click **Update service configuration** link .

   <img src="images/update_endpoint.png">

2. Update endpoint with the below details and click **OK**-

   - **Server URL**: http://YOUR_DNS_NAME:9000
   - **User name**: admin
   - **Password**:  admin

   <img src="images/update_2.png">

## Exercise 3: Configure and Trigger build

Now that SonarQube server is running, we will trigger the build to analyse the java code provisioned by the demo generator system.

1. Go to **Builds** under **Build and Release** tab, edit the build definition **SonarQube**. The tasks used in the build definition are listed.

   <table width="100%">
   <thead>
      <tr>
         <th width="50%"><b>Tasks</b></th>
         <th><b>Usage</b></th>
      </tr>
   </thead>
   <tr>
      <td><a href="http://bit.ly/2lvftfo"><b>Maven</b></a> <img src="images/maven.png"></td>
      <td>builds and tests java code</td>
   </tr>
   <tr>
      <td><a href="http://bit.ly/2grMxTQ"><b>Copy Files</b></a> <img src="images/copy-files.png"> </td>
      <td>copy files from source to destination folder using match patterns</td>
   </tr>
   <tr>
      <td><a href="http://bit.ly/2yBgXde"><b>Publish Build Artifacts</b></a> <img src="images/publish-build-artifacts.png"> </td>
      <td> publishes the build artifacts </td>
   </tr>
   </table>

3. Configure the **Maven** task with **Project Name, Project Key** as defined in **Exercise 1-->Step 3**.
   
   <br/>

   <img src="images/build_configure.png">

3. Save and queue the build.

   <img src="images/build_in_progress.png">

4. You will see the build summary with **Test Results, Code Coverage** and link to **SonarQube Analysis Report** when completed.  The **Quality Gate** displayed in the report is the best way to enforce a quality policy in your organization. Quality Gates are defined and managed in the **Quality Gates** page found in the top menu of the SonarQube portal.

   <img src="images/build_summary.png">

5. Go to **SonarQube** portal by clicking on the link in the build summary to see the dashboard.

   <img src="images/analysis_report.png">

## Exercise 4: Analyse SonarQube Reports

We will analyse the report in sonarqube portal to see if there are critical bugs and fix them in our code. In the below dashboard we have a critical bug. Let us fix this.

1. Go to sonarqube and click **Bugs**.

   <img src="images/sonar_portal.png">

2. Double click to see the details of the bug.

   <img src="images/bug_details.png">

   <br/>

   <img src="images/bug_details_2.png">

3. The error is due to explicitly casting the list object by making serializable. Lets fix the bug.

   Go to below path to edit the file in **VSTS** code tab:-
   
   >src/main/java/com/microsoft/example/servlet/LoginServlet.java

   Make the following changes in the code as shown:

   - Go to line number **3** and add the below code.

      >import java.io.Serializable;

      <img src="images/code_import.png">

   - Go to line number **28** and replace the existing code with below snippet.

      >session.setAttribute("employeeList", (Serializable)fareList);

      <img src="images/code_edit.png">

4. Commit the changes and go to **Build** to see the CI build in-progress.

5. Once the build is completed, you will see the Quality Gate **failing** in the build summary.

   <img src="images/build_summary_bug_fix.png">

6. Go to sonarqube portal. You will see the bug count is **0** and fixed now. 

   <img src="images/bug_fix_sonar_portal.png">

We see the Quality Gate is failing after code change as there is no code coverage available for the new code that we added to fix the bug.

This is how we analyse the code for every build and continuously improve the code quality to ship quality products.

## Summary

With **Visual Studio Team Services** and **SonarQube** you can easily schedule the execution of analysis and provide a shared vision of code quality for developers, tech leads, managers and executives, also to act as a toll gate for application promotion or release.

## Feedback



