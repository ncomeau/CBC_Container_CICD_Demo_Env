# CBC_Container_CICD_Demo

This repository is intended to provide the building blocks for creating a demo example of CBC Container Security, specifically cbctl in the build phase, in an effort to showcase the unequivocal value that can be garnered by integrating CBC Container Security into your CI/CD pipeline.

This section does not deal with the pipeline generation which will be utilized as examples of a build pipeline itself. Rather, this section is for the automated setup, and simple configuration of a demo environment - via such as means as an OVA file which provides; a mostly pre-configured Jenkins Server, a number of helper scripts used for outputing results to Slack via a pre-configured bot, and installation of necessary environment pre-requesites (microk8s, docker, etc.). 

The benefit to this approach is a simple setup of your lab environment, while allowing for the ability to import multiple different pipeline Jenkins files - to highlight different capabilities of the tool, and perpetually be iterated on.

Jake Barosin and myself will provide example pipeline scripts that can be utilized for such demos after the necessary setup instructions below.



## Environment Setup

1. Download the pre-configured Ubuntu OVA zip from this [repo.](https://onevmw-my.sharepoint.com/:f:/g/personal/ncomeau_vmware_com/Eni52TfGmutDvEkY7Z7Zk6MBo0GTAqxYmSQAEfbRX8L01A?e=A7t15f)
    * The download of this OVA is restircted to VMware personnel, as you are required to access it with your VMware email creds
    * This file is quite large, so please ensure you have a stable network connection, and allot the necessary time for download
    
2. Join Slack Org used for Demo
    * You can find the join link within the above-linked repo ☝️
      * This link does expire 14 days after initial creation, so if the link is expired, please contact me via VMware Slack DM (@ncomeau) for acccess
    * Join the ```#cbctl_jenkins_pipeline``` channel, as the helper scripts are hard coded to interact with the bots within this channel for notifications
    
3. Once you have downloaded the OVA zip, extract it, and import into your VMware Fusion Library 
    * From the VMware Fusion Management page, hit the "+" icon in the upper left
    * Select ```Import```
    * Select ```Choose File...```
    * Select the extracted OVA from the location you have just downloaded it
    * Select the imported file & click ```Continue```
    * Pick a name for your new vm, hit save, and allow it to import
      * _**NOTE:** You likely will want to provision the vm with more than the default allocated resources, if you intend to use this for multiple demo scenarios moving forward_
      ![image](https://user-images.githubusercontent.com/18126247/125986432-9c2ae2fa-389d-4dca-b23d-d080d96ce5be.png)
4. Once you have successfully imported your new vm, log into that vm with the username and password credentials provided via the ```login.txt``` file in the above reference repo ☝️☝️☝️
    * _**NOTE:** These creds are used for both the Ubuntu VM login & the Jenkins Server Login in a later step_
    
5. Open terminal window on your Ubuntu Machine
    * Run ```sudo -i``` and input in your user password to achieve root access
      * _**This is critical, as all the modules are installed under root context**_ 
    
6. Run the script in the root dir with the following command: ```sh setup.sh```
    * This script will start your Jenkins Server instance, as well as generate a config file for microk8s (used in upcoming step)
      * This config file will be generated at path _/home/demo/Desktop/config_
      * You might have also noticed a cleanup.sh script in that same dir - this is purely for optional use to easily prune any containers within the feline namespace, should you opt to leverage that namespace in future testing.


7. Navigate to your Jenkins Server
    * Open a browser on your Ubuntu VM
    * Navigate to ```0.0.0.0:8080```
    * Login with the same user/pass for the Ubuntu VM
      * _**NOTE:** After the initial setup, you can access the Jenkins Server from your Mac's browser, by navigating to:  ```http://Ubuntu_IP_Addr:8080```. However, for the initial config, since you need to import the file created in step 6, you have to be on you Ubuntu VM._

8. Configure Jenkins Server with Microk8s Config
    * Navigate to ```Manage Jenkins``` on the left-hand navigation
    * Under the ```System Configuration``` section, select ```Manage Nodes and Clouds```
    * In ```Manage Nodes and Clouds```, select ```Configure Clouds``` on the left-hand navigation
    * You will see a partially configured ```Kubernetes``` cloud - select ```Kubernetes Cloud Details``` on the right-hand side
    * Input in your Ubunutu VMs IP Address in the 3 designated fields: 
      *  _Kubernetes URL_
      *  _Jenkins URL_
      *  _Jenkins Tunnel_
    *  Under the ```Credentials``` section select _\[Add > Jenkins\]_
        * _**Kind** = 'Secret File'_
        * _**File** = Import the config file created on step 6 (Located at /home/demo/Desktop/config)_
        * _**ID** = Kubeconfig_
        * _**Description** = Anything you want (I usually just say 'kubeconfig')_
        * **Save Config**
    * Now, under the ```Credentials``` section dropdown menu your new creds should appear - select them
    * Try ```Test Connection``` button on the right-hand side
      * ![image](https://user-images.githubusercontent.com/18126247/125995463-d4abb12a-36b0-471c-983b-f3830a72fea6.png)
    * Once successful, hit ```Apply``` and then ``Save`` on the bottom of the screen!


9. Everything should be all set for your demo env! All thats left to do is cofigure cbctl, and then import in some pipelines! 🎉 🎉 🎉




## CB Container Security Setup

#### On Ubuntu Demo VM

1. As root (sudo -i) exec into your newly created Jenkins Server Container via the following command:

    ```docker exec -it jenkins-server /bin/bash```
    
2. Move on to the below setup within the CBC Console 

3. Once you have completed the setup for cbctl, copy and paste the command it specifies into your Jenkins Container shell

#### In CBC Console

1. Log into your CBC instance

2. Navigate to _\[Inventory > Kubernetes > K8s Clusters > CLI Config Tab > Add CLI\]_ 
    * Click through the install
      * I like to name my build step _'dev'_  personally - but it can be whatever you want
    * Once you get to step 3 ```Configure/Download CLI``` **STOP** 

3. Move back to Step 3 in prior section - which effectively states;
      * On you Ubuntu Demo VM, in the Jenkins Server Container shell you have running in terminal - 
**Copy and paste the configuration snippet presented to you in the CBC console, into your Jenkins Server Container shell you should have running in terminal**
        * _You do not need to worry about the binary download at this point, as that is addressed in the automated setup process_

4. _(Optional)_ You could/should make a scope with the build step that you defined in your CBC CLI setup, and then create a policy around that for the ```validate``` commands. 
    * However, any specific policy configurations required for pipelines will be specified in the associated README.md file for those builds.


## Basic Demo

As mentioned prior, Jake Barosin and I will provide links here to seperate repositories for different build pipelines, with the applicable instructions. However, if you are eager to test if your configuration works, below is a basic example of a cbctl image scanning pipeline, and slack output.

1. Navigate to your Jenkins Dashboard

2. Select ```New Item``` on the upper left-hand side

3. Enter a name for your project --> select ```Pipeline``` --> click 'ok' in the bottom left

4. Scroll down to the ```Pipeline``` section --> copy & paste in the following --> click 'apply' & 'save'
```
node {
     stage('Cbctl Image Scan') {
         
     sh '/var/jenkins_home/app/cbctl image scan ncomeau/demo -o json > cbctl_image_scan.json'
     
    }
    stage('Send Results to Slack'){
        try{
            sh 'git clone https://github.com/slackapi/python-slack-sdk.git'
         }
         catch(exists){
             echo 'already exists'
         }
        try{
            sh 'python3 /var/jenkins_home/app/image_scan_slack.py cbctl_image_scan.json'
            sh "python3 /var/jenkins_home/app/success.py '${env.JOB_NAME}' '${env.BUILD_NUMBER}'"
        }
        catch(fail){
            sh "python3 /var/jenkins_home/app/failure.py '${env.JOB_NAME}' '${env.BUILD_NUMBER}' '${env.STAGE_NAME}'"
        }
    }
}
```
5. Then simply click ```Build Now``` and let the magic happen! 🧙‍♂️ 🧙‍♀️

6. Upon successful completeion, your pipeline should look like the below & you should see the following in the slack instance configured in the setup steps!

### _Pipeline_
![image](https://user-images.githubusercontent.com/18126247/126007812-5caab659-0e53-41d2-9894-ca6289f1b292.png)

### _Slack_
![image](https://user-images.githubusercontent.com/18126247/126007932-0cf4cc0c-234b-478a-bc9d-39a80e2c2362.png)
