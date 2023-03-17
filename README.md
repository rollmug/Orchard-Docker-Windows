# Orchard + Docker + Windows Server
### Running Orchard Core CMS with Docker on Windows Server 2022

This is a walkthrough on getting up and running with Orchard Core CMS on Windows Server 2022, using Docker with Windows containers, along with a docker-compose.yml file.

**Overview:**

- [enable the Containers](#enable-containers) feature in Windows Server Manager
- [install Docker](#install-docker)
- [install Docker Compose](#install-docker-compose)
- [create files and set permissions](#orchard-core-cms)
- [run docker-compose command](#running-docker-compose-and-orchard)
- live long and prosper

## Enable Containers

Log into your Windows Server machine as Administrator, and open Windows Server Manager. Then:

1. select "Manage" from the top menu bar (or under the Quick Start menu on Dashboard), and select "Add Roles and Features"
2. click Next on the "Before You Begin" page
3. for Installation type, select "Role-based", and then Next
4. select the name of the server where the feature will be installed and click "Next"
5. no changes are to be made on the Server Roles page, so just click "Next"
6. on the Features page, check the box next to "Containers" and click "Next"
7. click "Install"
8. when it's done, click close
9. restart the machine

## Install Docker 

After the system has rebooted, run PowerShell as Administrator. From there, run the following:

`Install-Module -Name DockerMsftProvider -Repository PSGallery –Force`

This command will prompt you to install the Nuget provider in order to install the module. Go for it. Then, run the below command to install the latest Docker version:

`Install-Package -Name docker -ProviderName DockerMsftProvider`

Select "Yes to all", then: 

`Restart-Computer`

Once it's rebooted, re-open PowerShell as Administrator, and confirms that it's working by running: 

`Get-WindowsFeature -Name containers`

Now check the docker version, and verify it's running with by using these two commands:

`docker --version`

and also: 

`Get-Service docker`

## Install Docker Compose

Run PowerShell as Administrator. Enable TLS 1.2 by running:

`[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12`

Run the following command to download the latest release of Compose (as of this writing, v2.16.0, but change the version as needed):

`Start-BitsTransfer -Source "https://github.com/docker/compose/releases/download/v2.16.0/docker-compose-Windows-x86_64.exe" -Destination $Env:ProgramFiles\Docker\docker-compose.exe`

Test the installation:

`docker-compose version`

## Orchard Core CMS

In either File Exporer or from the command line, create the parent directory for your Orchard Core CMS install, for example, `Orchard-Core-CMS`, and cd into it. Into that directory, either clone the files from this repo, or create them manually (there's only 2: the `App_Data` dir and the `docker-compose.yml` file).

Since we want to persist Orchard data even if the Docker container is stopped or removed, we need to first set some permissions on our parent folder before we do anything else.

Follow these steps to add the "Users" group to our app:

1. in File Explorer, right-click the parent dir (in our case, `Orchard-Core-CMS`, or whatever you named it)
2. click Properties
3. click the Security tab
4. next to where it says, "To change permissions", click the "Edit" button.
5. click the Add button
6. where it says "Enter the object names", type "users" in the text box
7. click "Check Names" (should auto fill the value for you)
8. click OK
9. Highlight your newly added group
10. Under "Permissions for [User/Groupname]" select Full control
11. click apply
12. click "OK"
13. click "OK"

## Running Docker Compose and Orchard

From the command line, cd into the parent directory (`Orchard-Core-CMS`, or whatever you named it).

Run:

`docker-compose up`

This will take a while the first time you run it, but subsequent builds will be very fast. You'll see lots of logs. If all goes well, at the end of this you should be able to navigate to `http://localhost:8080` and continue your Orchard Core setup from within the browser.

Check to make sure files are persisted by navigating to the `App_Data` directory. You should now see some files that Orchard has created (logs, Sites, tenants.json).

If all is going well, now you can exit the session by typing Ctrl+C. We'll now want to run it again, but this time in the background so our terminal window doesn't need to stay open.

Now, run:

`docker-compose up -d`

The `-d` flag will run it in detached mode. Feel free to close the terminal session. You should again be able to navigate to `http://localhost:8080`, and the data from your initial setup should still be there.