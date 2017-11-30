## Managing Technical Debt using VSTS and SonarQube 

## Overview

In this lab, you will be introduced to technical debt management, configuring VSTS Team Build definition to use SonarQube and how to understand the analysis results.

Technical debt is the set of problems in a development effort that make forward progress on customer value inefficient. Technical debt saps productivity by making code hard to understand, fragile, time-consuming to change, difficult to validate, and creates unplanned work that blocks progress.

<a href="https://www.sonarqube.org/">SonarQube</a> is an open source platform for continuous inspection of code quality to perform automatic reviews with static analysis of code to

- Detect Bugs
- Code Smells
- Security Vulnerabilities
- Centralize Quality</a>

## Pre-requisites

1. **Microsoft Azure Account:** You need a valid and active azure account for the labs

2. You need a **Visual Studio Team Services Account** and <a href="https://docs.microsoft.com/en-us/vsts/accounts/use-personal-access-tokens-to-authenticate">Personal Access Token</a>

## Setting up the Environment

1. Click **Deploy To Azure** to provision SonarQube Server on Azure VM.

   <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fhsachinraj%2FAzurelabs%2Fmaster%2Fsonarqube%2Ftemplates%2Fazuredeploy.json"><img src="http://azuredeploy.net/deploybutton.png"></a>

    <img src="images/CustomDeployAzure1.png"></a>

    <img src="images/CustomDeployAzure2.png"></a>

  
   Provide the following parameters as shown.
   <table width="100%">
   <thead>
      <tr>
         <th width="50%"><b>Parameter Name</b></th>
         <th><b>Description</b></th>
         

   </thead>
   <tr>
      <td>Subscription Details</td>
      <td>Choose the active Azure subscription, create a new resource group along with the location of creation.</td>
      
   </tr>
 

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
   <tr>
      <td>_artifacts Location</td>
      <td>This value will be automatically generated. Leave the field as it is.</td>
      
   </tr>
   <tr>
      <td>_artifacts Location Sas Token</td>
      <td>The Sas Token will be automatically generated. Leave the field as it is.</td>
      
   </tr>
   </table>



2. After providing all of the required values in the above table, check the Terms & Conditions checkbox and click on the Purchase button.

   <img src="images/CustomDeployAzure3.png"></a>

   >It takes approximately 25-30 minutes to provision the environment

3. Once the deployment is successful, you will see the resources in Azure Portal. 

   <img src="images/azure_resources.png">

## Exercise 1: Create a SonarQube Project

1. Access the **SonarQube** portal providing the DNS name suffixed by the port number. 

   >The default port for SonarQube is 9000. Copy the DNS name from the created Virtual Machine in Azure Portal as shown and append :9000 at the end. The final URL will be **http://YOUR_DNS_NAME:9000**

   <img src="images/dns_name.png">

2. Login to the SonarQube Portal using the following credentials-    
   >**Username= admin, Password= admin**

   <img src="images/sonarqube_portal.png">

Now, let us create a SonarQube project.

1. Login to the SonarQube portal.

2. Click on **Administration** in the toolbar, go to **Projects** tab and click **Management**.

   <img src="images/sonar_admin.png">

3. Create a project with **Name** and **Key** as **MyShuttle**. 

   - **Name**: Name of the SonarQube project that will be displayed on the web interface

   - **Branch**[Optional]: Track the quality of short-lived and long-lived code branches

   - **Key**: The SonarQube project key that is unique for each project

   <img src="images/project_creation.png">

## Exercise 2: Setting up the VSTS project

1. Use <a href="http://bit.ly/2AWznna" target="_blank">VSTS Demo Data Generator</a> to provision a project on your VSTS account.

   ![](images/vstsdemogen.png)

2. Provide the **Project Name**, the **SonarQube URL** that was created previously and click on **Create Project**. Once the project is provisioned, click the URL to navigate.

   <img src="images/vsts_project_provisioning.png">
   


## Exercise 3: Modify the Build to Integrate with SonarQube

Now that the SonarQube server is running, we will modify VSTS build definition to integrate with SonarQube to analyse the java code provisioned by the VSTS Demo Generator system.

1. Go to **Builds** under **Build and Release** tab, edit the build definition **SonarQube**. The tasks used in the build definition are listed.

   <table width="100%">
   <thead>
      <tr>
         <th width="50%"><b>Tasks</b></th>
         <th><b>Usage</b></th>
      </tr>
   </thead>
   <tr>
      <td><img src="images/maven.png"> <b>Maven</b></td>
      <td>compiles and runs unit tests for the java code</td>
   </tr>
   <tr>
      <td><img src="images/copy-files.png"> <b>Copy Files</b></td>
      <td>copies the build artifacts to VSTS</td>
   </tr>
   <tr>
      <td><img src="images/publish-build-artifacts.png"> <b>Publish Build Artifacts</b></td>
      <td> publishes the build artifacts </td>
   </tr>
   </table>

3. Click on the **Maven** task and scroll down to the **Code Analysis** section. Configure the SonarQube settings as follows-

   <table width="100%">
   <thead>
      <tr>
         <th width="40%"><b>Parameter</b></th>
         <th><b>Value</b></th>
         <th><b>Notes</b></th>
      </tr>
   </thead>
   <tr>
      <td><b>SonarQube Project Name</b></td>
      <td>MyShuttle</td>
      <td>The name of the project in SonarQube</td>
   </tr>
   <tr>
      <td><b>SonarQube Project Key</b></td>
      <td>MyShuttle</td>
      <td>The unique key of the project in SonarQube</td>
   </tr>

   >Here, the SonarQube Project Name and SonarQube Project Key values are based on the values you provide in Exercise 1: Step 3. If **MyShuttle** is the value provided during the Project Key and Project Name creation, then you don't have to change it here as its already pre-configured.

   </table>
   
   <br/>

   <img src="images/build_configure.png">

3. Save and queue the build.

   <img src="images/build_in_progress.png">

4. You will see the build summary with **Test Results, Code Coverage** and link to **SonarQube Analysis Report** when completed.  The **Quality Gate** displayed in the report is the best way to enforce a code quality policy in your organization.

   <img src="images/build_summary.png">

5. Click on the **Detailed SonarQube Report** link in the build summary to open the project in SonarQube.

   <img src="images/analysis_report.png">

## Exercise 4: Analyse SonarQube Reports

We will analyse the report in SonarQube portal to see if there are critical bugs and fix them in our code. In the below dashboard, we have a critical bug. Let us fix this.

1. Go to SonarQube and click **Bugs**.

   <img src="images/sonar_portal.png">

2. Double click to see the details of the bug.

   <img src="images/bug_details.png">

3. You will see the error in line number 28 of **LoginServlet.java** file as **Make "List" serializable or don't store it in the session**.

   <img src="images/bug_details_2.png">

4. The error is due to explicitly casting the list object by making serializable. Lets fix the bug.

   Go to below path to edit the file in **VSTS** code tab:-
   
   >src/main/java/com/microsoft/example/servlet/LoginServlet.java

   Make the following changes in the code as shown:

   - Import the below package.

      >import java.io.Serializable;

      <img src="images/code_import.png">

   - Go to line number **28** and replace the existing code with below snippet.

      >session.setAttribute("employeeList", (Serializable)fareList);

      <img src="images/code_edit.png">

5. Commit the changes.

6. Once the CI build completes, you will see the Quality Gate **failing** in the build summary.

   <img src="images/build_summary_bug_fix.png">

7. Go to SonarQube portal. You will see the bug count is **0**. 

   <img src="images/bug_fix_sonar_portal.png">

We see the Quality Gate is failing after the code change as there is no unit tests created for the new code that we added to fix the bug.

## Summary

With **Visual Studio Team Services** and its integration with **SonarQube**, we can measure technical debt, analyze the results, manage and improve the code quality efficiently.

## Feedback



