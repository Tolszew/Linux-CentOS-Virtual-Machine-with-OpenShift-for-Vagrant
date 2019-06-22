# How to Build and Run a Linux CentOS Virtual Machine with OpenShift for Vagrant


## Building the Local DEV Cluster

This procedure was tested using the following software:

- Vagrant 2.0.2
- VirtualBox 5.2.12
- Windows 7 SP1
- CygWin

**Note:** All commands must be performed from the project's root directory if not stated otherwise.

### Setup Procedure

Steps:

- Install Packer from https://www.packer.io 
- To be able to access Packer from the command line, add it to the PATH system variable.
- Upgrade PowerShell to at least version 4.0.    
The package can be found here: https://www.microsoft.com/en-us/download/details.aspx?id=40855
- See the box-build project sources that are on the Git server (git.server.com/box-build.git).
- Edit the `centos-base-variables.json` file to specify the credentials for the proxy server. Remember to HTML-encode non-alphanumeric characters.

**Note:** Before running a command, make sure that all of the files in the project contain Unix style line endings.

- Open CygWin and change the directory to the project folder.
- Execute the following: `export ISO_URL= #`
       
**Note:** The value must point either to the URL with the CentOS ISO image, or to the location on the disk (escape backslashes with a backslash). 

For example: `C:\\centos-image.iso` 

The ISO image must be the **DVD** version. You can find it here:   
http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-1804.iso

- Run the following: 
`packer build -var-file=centos-base-variables.json -force centos-base.json`

When the Packer process finishes, there should be a `centos-base.box` file in the current directory.

 - Next, edit the `centos-openshift-variables.json` file by specifying the proxy user and proxy password.

**Note:** Because Guest Additions are propagated to the image from your local VirtualBox, make sure you have version 5.2.12 of VirtualBox installed. 

- Open the terminal and change the directory to the folder with the project.
- Next, execute the following:
`packer build -var-file=centos-openshift-variables.json -force centos-openshift.json`

When the Packer process finishes, there should be a centos-7-openshift.box file

- Upload the file to the Maven repository (maven.repository.com). Make sure that the credentials are configured for the server with the id user-upload in your ~/.m2/settings.xml
- Run the following command:

    ```
    mvn deploy:deploy-file 
    -DgroupId=com.example \      
    -DartifactId=vagrant-centos-open-shift \    
    -Dversion=1.0.7 \      
    -Dpackaging=box \        
    -Dfile=centos-7-openshift.box \      
    -DrepositoryId=user-upload \      
    -Durl=http://maven.repository.com/content/repositories/releases
    ```     
    
When the upload is succesful, do the following:

- Create a Git Tag for the deployed version: 1.0.X
- Update `-Dversion` from the above command with 1.0.X+1 pattern and push the file.

## Next Steps
You're now ready to move on to run the CentOS Virtual Machine on Windows.


## Running the CentOS Virtual Machine on Windows

This procedure was tested using:

- Vagrant 2.1.1
- VirtualBox 5.2.12
- Windows 7 SP1
- CygWin and PowerShell     

**Note:** All Vagrant commands must be executed from the project's root directory. The Vagrant image should be version 1.0.6. Please see the details about how to download an image located below.

**Prerequisites:** You have built the local DEV cluster.

Steps:

- Upgrade PowerShell to at least version 4.0. The package can be found here: https://www.microsoft.com/en-us/download/details.aspx?id=40855. You need to upgrade PowerShell, because it is used internally by Vagrant and the old version of PowerShell made Vagrant freeze.
- Install Vagrant from here: https://www.vagrantup.com/downloads.html and add it to the PATH system variable.
- To allow the use of .vagrantuser files, install the Nugrant plugin. 
- Set the http_proxy and https_proxy environment variables, and then execute the following command:

```
    export http_proxy=http://PROXY_USER:PROXY_PASSWORD_HERE@proxy.server.com:80/
    export https_proxy=http://PROXY_USER:PROXY_PASSWORD_HERE@proxy.server.com:80/
    vagrant plugin install nugrant
```
**Note:** If your password contains non-alphanumerical characters, you need to HTML-encode it before exporting variables. 
To encode your password and value, enter a string in place of `PROXY_PASSWORD_HERE`

- See the box-run project sources on the Git server (git.server.com/box-run.git).

When both Vagrant and Nugrant are installed, you can choose to use either PowerShell, CMD, or stick to CygWin when working with Vagrant.

- Next, edit the .vagrantuser file and specify the following:
  * Whether to use CNTLM on guest (details below) 
  * The memory limit
  * The CPUs assigned
  * Whether additional software needs to be installed
  * Whether to start the GUI

**Important:** If you are running the image for the first time, or the global password has changed, export the following environment variables:
`PROXY_USER and PROXY_PASS` 
This can be done only once, because the proper configuration file will be kept in VM. Do not HTML-encode the password if it contains non-alphanumeric characters. Instead, wrap it in double quotes.

**Note:** Before running any command make sure that all of the files in the project contain Unix style line endings.

- Unset http_proxy and https_proxy environment variables.
- Add a box (image) to the Vagrant registry. Open the terminal and change the directory to the project folder by executing the following:`vagrant box add metadata.json`
- After the command finishes, you can check if the image was added to Vagrant's registry by executing: `vagrant box list`

**Note:** If you already have the image in the repository and want to replace it, remove it first, and then run the command that adds the image again. 

- To remove the existing image, run the following:    
`vagrant box remove centos-7-openshift.box`

**Note:** The `vagrant box add` `metadata.json` should be executed only if there is no image in the local repository, or if there is a new version of the image and a fresh one is required.

- Make sure you have a private SHH key locally stored in the correct location. 
  * If you are using Cygwin, your private key has to be in `~/.ssh/id_rsa`. 
  * If you are using PowerShell or CMD, your private key has to be in `%USERPROFILE%\.ssh`

If you don't have a private key, you can use `PuTTYgen.exe` to generate one. Notice that the private key should not be password protected.
To generate a key, please follow the instructions here: https://www.ssh.com/ssh/putty/windows/puttygen 
 
- Once the key is generated, you need to save it. 
  * If you are using Cygwin, copy the file to `~/.ssh`
  * If you are using Powershell or CMD, save it to `%USERPROFILE%\.ssh as id_rsa`

When the box is successfully deployed in the Vagrant registry, you can run VM by executing the following command:    
`vagrant up` 

When the VM is up and running, you can SSH to the box by executing the following:    
`vagrant ssh`

When you stop working with the instance and want to halt the VM, execute the following:    
vagrant halt`



