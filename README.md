# How to Build and Run a Linux CentOS Virtual Machine with OpenShift for Vagrant

## Vagrant, Your Knight in Shining Armour

Of the many uses you can get out of a virtual machine (we could list them, but you probably already know them all), perhaps nothing is as crucial as being able to leap over that perennial gap between Windows-based software and Linux. you have to tap into an AWS Dev OpenShift cluster, which runs on RedHat Linux, while stuck in Windows 7. 

The base Linux image is free RedHat compatible CentOS. Added to the base CentOS image is the OpenShift platform, and other required tools, such as Maven, Git, Java and CNTLM to handle Internet access via Sabre corporate proxy. The 'local DEV cluster' image is built using Packer and requires some additional tools/plugins (e.g.: Cygwin).

With the built 'local DEV cluster' image available, you can run the 'Amazon Web Services DEV' mirror locally using Vagrant. After starting the virtual machine, you get OpenShift platform ready to use together with developer environment tools and an Internet connection.

## Building the Local DEV Cluster

This procedure was tested using the following software:

- Vagrant 2.0.2
- VirtualBox 5.2.12
- Windows 7 SP1
- Cygwin

All commands must be performed from the project's root directory if not stated otherwise.

### Setup Procedure

- Install Packer from here: [Packer Installation](https://www.packer.io) 
- To be able to access Packer from the command line, add it to the PATH system variable.
- Upgrade PowerShell to at least version 4.0.    
The package can be found here: [Packer Package](https://www.microsoft.com/en-us/download/details.aspx?id=40855)    
See the box-build project sources that are on the Git Server: git.server.com/box-build.git
- Edit the [centos-base-variables.json](centos-base-variables.json) file to specify the credentials for the proxy server. Remember to HTML-encode non-alphanumeric characters.

Before running a command, make sure that all of the files in the project contain Unix style line endings.

- Open Cygwin and change the directory to the project folder.
- Execute the following: `export ISO_URL= #`
       
 The value for `#` must point either to the URL with the CentOS ISO image, or to the location on the disk (escape backslashes with a backslash). For example: `C:\\centos-image.iso` 

The ISO image must be the **DVD** version. You can find it here: [ISO Image](http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-1804.iso)

- Run the following: 
`packer build -var-file=centos-base-variables.json -force centos-base.json`

When the Packer process finishes, there should be a `centos-base.box` file in the current directory.

 - Next, edit the [centos-openshift-variables.json](centos-openshift-variables.json) file by specifying the `proxy user` and `proxy password`

Because Guest Additions are propagated to the image from your local VirtualBox, make sure you have version 5.2.12 of VirtualBox installed. 

- Open the terminal and change the directory to the folder with the project.
- Next, execute the following:    
`packer build -var-file=centos-openshift-variables.json -force centos-openshift.json`

When the Packer process finishes, there should be a `centos-7-openshift.box` file available.

- Upload the file to the Maven repository: maven.repository.com    
Make sure that the credentials are configured for the server with the id user-upload in your `~/.m2/settings.xml`
- Next, run the following command:

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
    
When the upload is successful, do the following:

- Create a Git Tag for the deployed version: 1.0.X
- Next, update `-Dversion` from the above command with the 1.0.X+1 pattern, and push the file.

### Next Steps
You're now ready to install Vagrant, which will help run the 'Amazon Web Services DEV' mirror locally.


## Running the CentOS Virtual Machine on Windows

This procedure was tested using:

- Vagrant 2.1.1
- VirtualBox 5.2.12
- Windows 7 SP1
- Cygwin and PowerShell     

 All Vagrant commands must be executed from the project's root directory. The Vagrant image should be version 1.0.6. Please see the details below about how to download the image.


### Installing Vagrant Behind the Proxy Server

- Upgrade PowerShell to at least version 4.0. PowerShell is used internally by Vagrant and the old version of PowerShell made Vagrant freeze. The package can be found here: [PowerShell Package](https://www.microsoft.com/en-us/download/details.aspx?id=40855)   
- Install Vagrant from here: [Vagrant Installation](https://www.vagrantup.com/downloads.html) 
- Add it to the PATH system variable.
- To allow the use of `.vagrantuser` files, install the Nugrant plugin. 
- Set the `http_proxy` and `https_proxy` environment variables, and then execute the following command:

```
    export http_proxy=http://PROXY_USER:PROXY_PASSWORD_HERE@proxy.server.com:80/
    export https_proxy=http://PROXY_USER:PROXY_PASSWORD_HERE@proxy.server.com:80/
    vagrant plugin install nugrant
```
If your password contains non-alphanumerical characters, you need to HTML-encode it before exporting variables. 
To encode your password and value, enter a string in place of `PROXY_PASSWORD_HERE`     
See the box-run project sources here: git.server.com/box-run.git

When both Vagrant and Nugrant are installed, you can choose to use PowerShell or CMD, or continue to use Cygwin when working with Vagrant.

### Configuring Vagrant

- Edit the `.vagrantuser` file, and specify the following:
  * Whether to use CNTLM for guests (details below) 
  * The memory limit
  * The CPUs assigned
  * Whether additional software needs to be installed
  * Whether to start the GUI

**Important:** If you are running the image for the first time, or the global password has changed, export the following environment variables:
`PROXY_USER` and `PROXY_PASS` 
This can be done only once, because the proper configuration file will be kept in VM. Do not HTML-encode the password if it contains non-alphanumeric characters. Instead, wrap it in double quotes.

Before running a command, make sure that all of the files in the project contain Unix style line endings.
- Unset the ``http_proxy`` and ``https_proxy`` environment variables.
- Add a box (image) to the Vagrant registry. Open the terminal and change the directory to the project folder by executing the following:     
`vagrant box add metadata.json`
- After the command finishes, you can check if the image was added to Vagrant's registry by executing:    
`vagrant box list`

The `vagrant box add metadata.json` should be executed only if there is no image in the local repository, or if there is a new version of the image and a fresh one is required. If you already have the image in the repository and want to replace it, remove it first, and then run the command that adds the image a second time. To remove the existing image, run the following command:`vagrant box remove centos-7-openshift.box`

The `vagrant box add metadata.json` should be executed only if there is no image in the local repository, or if there is a new version of the image and a fresh one is required.

### Generating an SHH Key

Make sure that you have a private SHH key locally stored in the correct location. 
  * If you are using Cygwin, your private key has to be in `~/.ssh/id_rsa`. 
  * If you are using PowerShell or CMD, your private key has to be in `%USERPROFILE%\.ssh`
If you don't have a private key, you can use `PuTTYgen.exe` to generate one. Notice that the private key should not be password protected.
To generate a key, please follow the instructions here: [Puttygen Instructions](https://www.ssh.com/ssh/putty/windows/puttygen) 
- Once the key is generated, you need to save it. 
  * If you are using Cygwin, copy the file to `~/.ssh`
  * If you are using Powershell or CMD, save it to `%USERPROFILE%\.ssh as id_rsa`

### Running the Virtual Machine

- When the box is successfully deployed in the Vagrant registry, you can run VM by executing the following command:    
`vagrant up` 
- When the VM is up and running, you can SSH to the box by executing the following:    
`vagrant ssh`
- If you want to stop working with the instance and halt the VM, execute the following:    
`vagrant halt`



