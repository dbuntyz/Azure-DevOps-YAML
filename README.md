
#  Azure DevOps Pipeline for Laravel 11

  

Follow these steps to set up an Azure App Service for deploying a Laravel 11 application.

  

##  Step 1: Create an App Service

  

1. Go to **Azure Portal** and create a new **App Service**.

2.  **Choose Subscription**: Select your Azure subscription.

3.  **Choose Resource Group**: Select an existing resource group or create a new one.

4.  **Provide a Name**: Enter a globally unique name for your App Service.

5.  **Choose Code**: Select "Code" as the publishing option.

6.  **Runtime Stack**: Choose `PHP 8.2` as the runtime stack.

7.  **Region**: Select the region that best fits your needs.

8.  **App Service Plan**: Choose an existing App Service Plan or create a new one that fits your requirements.

9.  **Zone Redundancy**: Keep enabled if you need high availability; otherwise, disable it.

10. Click **Next (Database)**.

- Create a Database: Check this if you want to use Azure MySQL Database; otherwise, uncheck it if you have another server or plan to use SQLite.

- If checked:

- Choose Engine: Select the database engine that fits your needs.

- Provide Server Name: Enter a unique server name.

- Database Name: Provide the name of your database (this should be referenced in the `.env` file of your Laravel codebase).

11. Click **Next (Deployment)**.

- Deployment Method: Choose "Disable" since deployment will be handled via Azure DevOps.

- Basic Authentication: Keep this disabled unless required.

12. Click **Next (Networking)**.

- Keep the default settings unless you need specific network configurations.

13. Click **Next (Monitor + Secure)**.

- Enable Application Insights: Keep disabled unless monitoring is required.

- Enable Defender: Enable if required (note that additional costs are involved).

14. Click **Next (Tags)**.

- Provide tags in key-value pairs (recommended for better resource management).

15. Click **Review + Create** and wait for the App Service to be created.

  
  

##  Step 2: Configure Environment Variables

  

1. Go to the **App Service** in the Azure Portal.

2. Navigate to **Configuration** > **Application Settings** > **Environment Variables**.

3. Click **Add** and add the following environment variable:

-  **Name**: `PORT`

-  **Value**: `8000`

4. In the **App Service** > **Configuration** > **General Settings**, add the following startup command:

-  `php artisan serve --host 0.0.0.0`

## Step 3: Azure DevOps Setup
### Prerequisites

1.  An active Azure DevOps account with access to your repository.
2.  A Laravel 11 application stored Azure Repos.
3.  An Azure Web App created and configured to run PHP applications.
4.  Azure DevOps Service Connection with appropriate permissions to deploy to your Azure Web App.

## Step-by-Step Pipeline Configuration

### Step 1: Create a New Pipeline in Azure DevOps

1.  **Navigate to Azure DevOps**:
    
    - Go to your Azure DevOps organization and select your project.
    - Click on **Repos** in the sidebar.
	- Click **+ New repository** and give it a name.
	- Clone the newly created repository to your local machine using the provided Azure Repo URL.
	- Add your Laravel application files to the cloned folder.
	- Use the following commands to push your code:
	`git add .`
    `git commit -m "Initial commit of Laravel application"`
    `git push origin main`

2.  **Create a New Pipeline**:
    
    -   Click on **Pipelines** in the left sidebar.
    -   Click on **New Pipeline**.
    -   Select the repository where your Laravel application is stored.
3. **Upload your .env file in Secure Files**
	- Click on **Pipelines** from the left sidebar.
	- Select **Library** under the **Pipelines** section.
	- Click on **Secure Files** in the Library section.
	- Click the **+ Secure file** button.
	- Select the `.env` file from your local machine and upload it.
5.  **Choose YAML Configuration**:
    
    -   Select **YAML** and point it to your existing YAML file or add the below YAML content directly.

### Step 2: Add the YAML Script

Copy and paste the following YAML configuration into your pipeline file (`azure-pipelines.yml`):

    trigger:
      - main
    
    pool:
      vmImage: 'ubuntu-latest'
    
    variables:
      phpVersion: 8.2
    
    steps:
      # Install PHP 8.2 from PPA
      - script: |
          sudo add-apt-repository ppa:ondrej/php -y
          sudo apt-get update
          sudo apt-get install php8.2 -y
          php -v
        displayName: 'Install PHP 8.2 from PPA'
    
      # Install necessary PHP extensions
      - script: |
          sudo apt-get update
          sudo apt-get install -y \
            php8.2-xml \
            php8.2-mbstring \
            php8.2-dom \
            php8.2-zip \
            php8.2-xmlwriter \
            php8.2-curl \
            php8.2-gd
          php -m  # List enabled extensions
        displayName: 'Install PHP Extensions'
    
      # Download secure .env file
      - task: DownloadSecureFile@1
        inputs:
          secureFile: '.env'
        displayName: 'Download .env Secure File'
    
      # Move .env file to the working directory
      - script: |
          mv $(Agent.TempDirectory)/.env $(System.DefaultWorkingDirectory)
        displayName: 'Move .env to working directory'
    
      # Set the PHP version for use
      - script: |
          sudo update-alternatives --set php /usr/bin/php$(phpVersion)
          sudo update-alternatives --set phar /usr/bin/phar$(phpVersion)
          sudo update-alternatives --set phpdbg /usr/bin/phpdbg$(phpVersion)
          sudo update-alternatives --set php-cgi /usr/bin/php-cgi$(phpVersion)
          sudo update-alternatives --set phar.phar /usr/bin/phar.phar$(phpVersion)
          php -version
        displayName: 'Use PHP version $(phpVersion)'
    
      # Install Composer dependencies
      - script: |
          composer install --no-interaction --prefer-dist
        displayName: 'Install Composer Dependencies'
    
      # Package application files, including the .env file
      - script: |
          mkdir -p $(Build.ArtifactStagingDirectory)
          cp .env $(Build.ArtifactStagingDirectory) # Include .env file
          zip -r $(Build.ArtifactStagingDirectory)/app.zip .
        displayName: 'Package Application'
    
      # Publish the build artifacts
      - task: PublishBuildArtifacts@1
        inputs:
          PathtoPublish: '$(Build.ArtifactStagingDirectory)'
          ArtifactName: 'drop'
          publishLocation: 'Container'
        displayName: 'Publish Build Artifacts'
    
      # Deploy to Azure Web App
      - task: AzureRmWebAppDeployment@4
        inputs:
          ConnectionType: 'AzureRM'
          azureSubscription: 'TechOkay (ab1b1ec7-da7b-480d-97e9-5d6daf8271d5)' # Update with your Service Connection Name
          appType: 'webAppLinux'
          WebAppName: 'techokaylaravel' # Update with your Azure Web App Name
          packageForLinux: '$(Build.ArtifactStagingDirectory)/**/*.zip'
        displayName: 'Deploy to Azure Web App'


## Post-Deployment Access and Troubleshooting

After the Azure Pipeline executes successfully, your project will be deployed to the Azure App Service, and you should be able to access your Web App URL to view the project. However, if you encounter issues accessing the updated code, follow these steps:

### Steps to Ensure Web App Access:

1.  **Check Web App URL**:
    
    -   After the deployment, navigate to your Web App URL to check if the application is accessible.
2.  **If the Web App Does Not Reflect Changes**:
    
    -   **Stop the App Service**:
        -   Go to your App Service in the Azure Portal.
        -   Click on **Stop** to halt the service.
        -   Wait until the status changes and the page shows that the App Service is stopped or unavailable.
    -   **Refresh the Web App URL**:
        -   Keep refreshing the URL page to ensure it shows that the service is unavailable or stopped.
    -   **Start the App Service**:
        -   Once confirmed that the service is stopped, click on **Start** to turn it on again.
        -   This action will reflect the new code and restart the service with the latest changes.
3.  **Monitor Logs with Log Stream**:
    
    -   It is recommended to keep the **Log Stream** open in a different tab during this process to monitor current logs and see real-time output.
    -   The Log Stream will help identify any potential issues during the start-up phase and provide insights into the status of your application.

### Additional Tips

-   **Log Stream**: Watching logs as the App Service restarts can help quickly spot errors related to startup commands, environment configurations, or application code.
-   **Restart vs. Redeploy**: If changes are not reflected after restarting, consider redeploying the application via Azure DevOps to ensure the latest code is published correctly.