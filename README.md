# Azure DevOps Pipeline for Laravel 11


# Documentation: Setting Up an Azure App Service for Laravel 11

Follow these steps to set up an Azure App Service for deploying a Laravel 11 application.

## Step 1: Create an App Service

1.  Go to **Azure Portal** and create a new **App Service**.
2.  **Choose Subscription**: Select your Azure subscription.
3.  **Choose Resource Group**: Select an existing resource group or create a new one.
4.  **Provide a Name**: Enter a globally unique name for your App Service.
5.  **Choose Code**: Select "Code" as the publishing option.
6.  **Runtime Stack**: Choose `PHP 8.2` as the runtime stack.
7.  **Region**: Select the region that best fits your needs.
8.  **App Service Plan**: Choose an existing App Service Plan or create a new one that fits your requirements.
9.  **Zone Redundancy**: Keep enabled if you need high availability; otherwise, disable it.
10.  Click **Next (Database)**.
    -   Create a Database: Check this if you want to use Azure MySQL Database; otherwise, uncheck it if you have another server or plan to use SQLite.
    -   If checked:
        -   Choose Engine: Select the database engine that fits your needs.
        -   Provide Server Name: Enter a unique server name.
        -   Database Name: Provide the name of your database (this should be referenced in the `.env` file of your Laravel codebase).
11.  Click **Next (Deployment)**.
    -   Deployment Method: Choose "Disable" since deployment will be handled via Azure DevOps.
    -   Basic Authentication: Keep this disabled unless required.
12.  Click **Next (Networking)**.
    -   Keep the default settings unless you need specific network configurations.
13.  Click **Next (Monitor + Secure)**.
    -   Enable Application Insights: Keep disabled unless monitoring is required.
    -   Enable Defender: Enable if required (note that additional costs are involved).
14.  Click **Next (Tags)**.
    -   Provide tags in key-value pairs (recommended for better resource management).
15.  Click **Review + Create** and wait for the App Service to be created.


## Step 2: Configure Environment Variables

1.  Go to the **App Service** in the Azure Portal.
2.  Navigate to **Configuration** > **Application Settings** > **Environment Variables**.
3.  Click **Add** and add the following environment variable:
    -   **Name**: `PORT`
    -   **Value**: `8000`
4.  In the **App Service** > **Configuration** > **General Settings**, add the following startup command: 
	- `php artisan serve --host 0.0.0.0`

