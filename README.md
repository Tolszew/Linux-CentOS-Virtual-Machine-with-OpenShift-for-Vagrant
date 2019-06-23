# How to Build and Run a Linux CentOS Virtual Machine with OpenShift for Vagrant

## Our Goal

We want to run the AWS DEV cluster, which Sabre uses for cloud applications, on our Windows 7 OS. This will give our developers more freedom to explore and manoeuvre.  

Because the AWS DEV cluster runs on RedHat Linux, we have to use a virtual machine to mirror it on Windows. To do this, we'll be using **Vagrant** to install and configure the guest OS, and **VirtualBox** to run it.

To get the guest OS up, we're going to use a free, RedHat-compatible **CentOS** image, which we'll build with **Packer**. Packer requires additional tools/plugins, including **Cygwin**, which is a collection of tools that will help integrate the Linux environment with Windows commands.

On top of the base **CentOS** image, our guest OS, we'll be placing the **OpenShift** platform, and several other tools for handling the Sabre corporate proxy, including **Maven**, **Git**, **Java**, and **CNTLM**. 

When we're finished building the AWS DEV cluster from the CentOS image, we'll be able to run the 'Amazon Web Services DEV' mirror locally using **Vagrant**. After starting the virtual machine, we will have the **OpenShift** platform ready to use together with developer environment tools and an Internet connection.

## Building the DEV Cluster

### Main Software Used - Quick Recap
**Windows 7 SP1:** Our host operating system     
**CentOS:** Our guest operating system     
**Packer:** An open source tool that will help us build the Vagrant box files so that VirtualBox can run     
**Cygwin:** You can think of it as an OS that adds a layer on top of Windows and allows us to run Linux applications     **PowerShell:** A more refined Command Prompt which Vagrant uses internally     
**Vagrant 2.0.2:** The virtual machine manager that will help us setup and configure our VM     
**VirtualBox 5.2.12:** The virtualization application that creates the VM for CentOS    

Before we begin with the setup, keep in mind that you have to perform all of the commands from the project's root directory unless otherwise indicated.

### Setting Up with Packer
 
- Install Packer from here: [Packer Installation](https://www.packer.io) 
- To access Packer from the command line, add it to the PATH system variable.
- Upgrade PowerShell to at least version 4.0. The package can be found here: [PowerShell](https://www.microsoft.com/en-us/download/details.aspx?id=40855)    
- Have a look at the box-build project sources on the Git Server: git.server.com/box-build.git
Next, we need to configure proxy in CentOS.
- Edit the [centos-base-variables.json](centos-base-variables.json) file to specify the credentials for the proxy server. Remember to HTML-encode non-alphanumeric characters.

### Creating Vagrant Box Files with Cygwin

Now, we have to use Cygwin to run several commands and create the Vagrant box files to run VirtualBox.
Before running a command, make sure that all of the files in the project contain Unix style line endings.

- Open Cygwin and change the directory to the project folder.
- Execute the following: `export ISO_URL= #`
       
The value for `#` needs to point either to the URL with the CentOS ISO image, or to the location on the disk (escape backslashes with a backslash). For example: `C:\\centos-image.iso` 

Make sure to use the **DVD** version of CentOS ISO. You can find the ISO through one of the mirrors listed here: [ISO Image](http://isoredirect.centos.org/centos/7/isos/x86_64)

To generate the Vagrant box file, run the following command in Cygwin: 
`packer build -var-file=centos-base-variables.json -force centos-base.json`

When Packer finishes, there should be a `centos-base.box` file in the current directory.

### Configuring the Proxy Environment Variables for OpenShift

Next, we need to configure the proxy environment variables for our OpenShift platform. 
- Edit the [centos-openshift-variables.json](centos-openshift-variables.json) file by specifying the `proxy user` and `proxy password`

Because Guest Additions are propagated to the image from your local VirtualBox, make sure you have version 5.2.12 of VirtualBox installed. 

To create the OpenShift box file, do the following:
- Open the terminal and change the directory to the folder with the project.
- Execute the following command:    
`packer build -var-file=centos-openshift-variables.json -force centos-openshift.json`

When Packer finishes, there should be a `centos-7-openshift.box` file available.

Next, upload the `centos-7-openshift.box` file to the Maven repository: maven.repository.com    
You want the credentials to be configured for the server, with the id user-upload in your `~/.m2/settings.xml`
- To upload the file, run the following command:

    ```
    mvn deploy:deploy-file -DgroupId=com.example \      
    -DartifactId=vagrant-centos-open-shift \    
    -Dversion=1.0.7 \      
    -Dpackaging=box \        
    -Dfile=centos-7-openshift.box \      
    -DrepositoryId=user-upload \      
    -Durl=http://maven.repository.com/content/repositories/releases
    ```     
    
When the upload is successful, do the following:

- Create a Git Tag for the deployed version: 1.0.X
- Update `-Dversion` (from the above command) with the 1.0.X+1 pattern, and push the file.

### Next Steps
Now you're ready to install Vagrant, which will help you setup your virtual environment.


## Installing and Running the CentOS Virtual Machine on Windows

The following software was used install and run the VM:

- Vagrant 2.1.1
- VirtualBox 5.2.12
- Windows 7 SP1
- Cygwin and PowerShell     

Keep in mind that you will have to execute all Vagrant commands from the project's root directory.     
The Vagrant image should be version 1.0.6. See the details below about how to download the image.


### Installing Vagrant and Nugrant Behind the Proxy Server

Before we begin with the installation, make sure that you've upgraded PowerShell to at least version 4.0. Vagrant uses PowerShell internally and the old version of PowerShell made Vagrant freeze up.   
The package can be found here: [PowerShell Package](https://www.microsoft.com/en-us/download/details.aspx?id=40855)

- Install Vagrant from here: [Vagrant Installation](https://www.vagrantup.com/downloads.html) 
- To access Vagrant from the command line, add it to the PATH system variable.
- To be able to use `.vagrantuser` files, you have to also install the Nugrant plugin. 
- Next, set the `http_proxy` and `https_proxy` environment variables, and then execute the following command:

```
    export http_proxy=http://PROXY_USER:PROXY_PASSWORD_HERE@proxy.server.com:80/
    export https_proxy=http://PROXY_USER:PROXY_PASSWORD_HERE@proxy.server.com:80/
    vagrant plugin install nugrant
```
If your password contains non-alphanumerical characters, you need to HTML-encode it before exporting variables. 
To encode your password and value, enter a string in place of `PROXY_PASSWORD_HERE`     
See the box-run project sources here: git.server.com/box-run.git

After you have installed Vagrant and Nugrant, feel free to use PowerShell or CMD, or continue to use Cygwin when working with Vagrant.

### Configuring Vagrant

To configure Vagrant, you need to edit the `.vagrantuser` file, and specify the following:
- Whether or not to use CNTLM for guests (details below) 
- The memory limit
- The CPUs assigned
- Whether or not additional software needs to be installed
- Whether to start the GUI

**Important:** If you're running the image for the first time, or the global password has changed, export the following environment variables: `PROXY_USER` and `PROXY_PASS` 
This can be done only once, because the proper configuration file will be kept in the VM. Do not HTML-encode the password if it contains non-alphanumeric characters. Instead, wrap it in double quotes.

Before running a command, make sure that all of the files in the project contain Unix style line endings.

- Unset the ``http_proxy`` and ``https_proxy`` environment variables.
- Add a box (image) to the Vagrant registry. Open the terminal and change the directory to the project folder by executing the following:     
`vagrant box add metadata.json`
The `vagrant box add metadata.json` should be executed only if there is no image in the local repository, or if there is a new version of the image and a fresh one is required. If you already have the image in the repository and want to replace it, remove it first, and then run the command that adds the image again. 
To remove the existing image, run the following command:`vagrant box remove centos-7-openshift.box`

- After the `vagrant box add metadata.json` command finishes, you can check if the image was added to Vagrant's registry by executing:   
`vagrant box list`

### Generating an SHH Key
Secure Shell is used to connect Vagrant with your Windows environment.
To establish the connection, you need to have a private SHH key locally stored in the correct location. 
  * If you are using Cygwin, your private key has to be in `~/.ssh/id_rsa`. 
  * If you are using PowerShell or CMD, your private key has to be in `%USERPROFILE%\.ssh`
  * If you don't have a private key, you can use `PuTTYgen.exe` to generate one.     
  Make sure that the private key is not be password protected.     
  To generate a key, follow the instructions here: [Puttygen Instructions](https://www.ssh.com/ssh/putty/windows/puttygen) 

Once the key is generated, you need to save it. 
- If you are using Cygwin, copy the file to `~/.ssh`
- If you are using PowerShell or CMD, save it to `%USERPROFILE%\.ssh as id_rsa`

### Running the VM

When the box is successfully deployed in the Vagrant registry, you can boot the virtual machine. 
- Execute the following command:    
`vagrant up`
The VM should be up and running in a short time, although you won't see it when it starts. Vagrant doesn't use a UI to run the VM, so you have to SSH to the box in order to interact with the machine. 
- To do so, execute the following command:    
`vagrant ssh`     
If at any time you want to stop working with the instance and halt the VM, execute the following command:    
`vagrant halt`

There it is! Now you have a virtual machine which mimics the AWS DEV on your Windows 7 OS.



