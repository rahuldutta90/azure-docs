---
title: Local Git Deployment to Azure App Service
description: Learn how to enable local Git deployment to Azure App Service.
services: app-service
documentationcenter: ''
author: dariagrigoriu
manager: cfowler
editor: mollybos

ms.assetid: ac50a623-c4b8-4dfd-96b2-a09420770063
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/14/2016
ms.author: dariagrigoriu

---
# Local Git Deployment to Azure App Service

This tutorial shows you how to deploy your app to [Azure Web Apps](app-service-web-overview.md) from a Git repository on your local computer. App Service supports this approach with the **Local Git** deployment option in the [Azure portal].

## Prerequisites

To complete this tutorial, you need:

* Git. You can download the installation binary [here](http://www.git-scm.com/downloads).
* Basic knowledge of Git.
* A Microsoft Azure account. If you don't have an account, you can 
  [sign up for a free trial](https://azure.microsoft.com/pricing/free-trial) or 
  [activate your Visual Studio subscriber benefits](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details).

> [!NOTE]
> If you want to get started with Azure App Service before signing up for an Azure account, go to [Try App Service](https://azure.microsoft.com/try/app-service/), where you can immediately create a short-lived starter app in App Service. No credit cards required; no commitments.
>

## <a name="Step1"></a>Step 1: Create a local repository

Perform the following tasks to create a new Git repository.

1. Start a command-line tool, such as **Git Bash** (Windows) or **Bash** (Unix Shell). On OS X systems, you can access the commandline through the **Terminal** application.
1. Navigate to the directory where the content to deploy would be located.
1. Use the following command to initialize a new Git repository:

  ```bash
  git init
  ```

## <a name="Step2"></a>Step 2: Commit your content

App Service supports applications created in a variety of programming languages.

1. If your repository does not already include content, add a static .html file as follows; or skip this step:
   * Using a text editor, create a new file named **index.html** at the root of the Git repository
   * Add the following text as the contents of the index.html file and save it: *Hello Git!*
1. From the command-line, verify that you are under the root of your Git repository. Then use the following command to add files to your repository:

        git add -A 
1. Next, commit the changes to the repository by using the following command:

        git commit -m "Hello Azure App Service"

## <a name="Step3"></a>Step 3: Enable the App Service app repository

Perform the following steps to enable a Git repository for your App Service app.

1. Log in to the [Azure portal].
1. In your App Service app's view, click **Settings > Deployment source**. Click **Choose source**, then click **Local Git Repository**, and then click **OK**.

    ![Local Git Repository](./media/app-service-deploy-local-git/local_git_selection.png)

1. If this is your first time setting up a repository in Azure, you need to create login credentials for it. You use them to log into the Azure repository and push changes from your local Git repository. From your Web App's view, click **Settings > Deployment credentials**, then configure your deployment username and password. When you're done, click **Save**.

    ![](./media/app-service-deploy-local-git/deployment_credentials.png)

## <a name="Step4"></a>Step 4: Deploy your project

Use the following steps to publish your app to App Service using Local Git.

1. In your Web App's view in the Azure portal, click **Settings > Properties** for the **Git URL**.

    ![](./media/app-service-deploy-local-git/git_url.png)

    **Git URL** is the remote reference to deploy to from your local repository. You use this URL in the next steps.
1. Using the command line, verify that you are in the root of your local Git repository.
1. Use `git remote` to add the remote reference listed in **Git URL** from step 1. Your command looks similar to the following:

    ```bash
    git remote add azure https://<username>@localgitdeployment.scm.azurewebsites.net:443/localgitdeployment.git
    ```

   > [!NOTE]
   > The **remote** command adds a named reference to a remote repository. In this example, it creates a reference named 'azure' for your web app's repository.
   >

1. Push your content to App Service using the new **azure** remote you created.

    ```bash
    git push azure master
    ```

    You are prompted for the password you created earlier when you reset your deployment credentials in the Azure portal. Enter the password (note that Gitbash does not echo asterisks to the console as you type your password). 
1. Go back to your app in the Azure portal. A log entry of your most recent push should be displayed in the **Deployments** view.

    ![](./media/app-service-deploy-local-git/deployment_history.png)

1. Click the **Browse** button at the top of the web app page to verify the content has been deployed.

## <a name="Step5"></a>Troubleshooting

The following are errors or problems commonly encountered when using Git to publish to an App Service app in Azure:

---
**Symptom**: `Unable to access '[siteURL]': Failed to connect to [scmAddress]`

**Cause**: This error can occur if the app is not up and running.

**Resolution**: Start the app in the Azure portal. Git deployment is unavailable when the Web App is stopped.

---
**Symptom**: `Couldn't resolve host 'hostname'`

**Cause**: This error can occur if the address information entered when creating the 'azure' remote was incorrect.

**Resolution**: Use the `git remote -v` command to list all remotes, along with the associated URL. Verify that the URL for the 'azure' remote is correct. If needed, remove and recreate this remote using the correct URL.

---
**Symptom**: `No refs in common and none specified; doing nothing. Perhaps you should specify a branch such as 'master'.`

**Cause**: This error can occur if you do not specify a branch during `git push`, or if you have not set the `push.default` value in `.gitconfig`.

**Resolution**: Perform the push operation again, specifying the master branch. For example:

```bash
git push azure master
```

---
**Symptom**: `src refspec [branchname] does not match any.`

**Cause**: This error can occur if you attempt to push to a branch other than master on the 'azure' remote.

**Resolution**: Perform the push operation again, specifying the master branch. For example:

```bash
git push azure master
```

---
**Symptom**: `RPC failed; result=22, HTTP code = 5xx.`

**Cause**: This error can occur if you attempt to push a large git repository over HTTPS.

**Resolution**: Change the git configuration on the local machine to make the postBuffer bigger

```bash
git config --global http.postBuffer 524288000
```

---
**Symptom**: `Error - Changes committed to remote repository but your web app not updated.`

**Cause**: This error can occur if you are deploying a Node.js app containing a package.json file that specifies additional required modules.

**Resolution**: Additional messages containing 'npm ERR!' should be logged prior to this error, and can provide additional context on the failure. The following are known causes of this error and the corresponding 'npm ERR!' message:

* **Malformed package.json file**: npm ERR! Couldn't read dependencies.
* **Native module that does not have a binary distribution for Windows**:

  * `npm ERR! \cmd "/c" "node-gyp rebuild"\ failed with 1`

      OR
  * `npm ERR! [modulename@version] preinstall: \make || gmake\`

## Additional Resources

* [Git documentation](http://git-scm.com/documentation)
* [Project Kudu documentation](https://github.com/projectkudu/kudu/wiki)
* [Continuous Deployment to Azure App Service](app-service-continuous-deployment.md)
* [How to use PowerShell for Azure](/powershell/azure/overview)
* [How to use the Azure Command-Line Interface](../cli-install-nodejs.md)

[Azure Developer Center]: http://www.windowsazure.com/en-us/develop/overview/
[Azure portal]: https://portal.azure.com
[Git website]: http://git-scm.com
[Installing Git]: http://git-scm.com/book/en/Getting-Started-Installing-Git
[Azure Command-Line Interface]: https://azure.microsoft.com/en-us/documentation/articles/xplat-cli-azure-resource-manager/

[Using Git with CodePlex]: http://codeplex.codeplex.com/wikipage?title=Using%20Git%20with%20CodePlex&referringTitle=Source%20control%20clients&ProjectName=codeplex
[Quick Start - Mercurial]: http://mercurial.selenic.com/wiki/QuickStart
