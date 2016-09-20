# Resource optimization using Mistral YAML


These example demonstrate how to use OpenStack Mistral actions in Mistral workflow to delete the VMs which is in SHUTOFF state for more than 24Hrs.

## Prerequisites:
      * Openstack with mistral setup(Ubuntu 14.04).
      * PyV8 Engine.

In order to use Java script inside the YAML need to install PyV8 engine, steps are as follow.
Before jumping into the installation part disable ssl verfication of git by execute the following command.
 
      $ git config --global http.sslverify false

## Installation: 
* **Install required libraries:**
``` 
 $ sudo apt-get install libboost-all-dev g++ libtool autoconf libv8-legacy-dev subversion make
 $ sudo apt-get install build-essential autoconf libtool pkg-config python-opengl python-imaging python-pyrex python-pyside.qtopengl idle-python2.7 qt4-dev-tools qt4-designe libqtgui4 libqtcore4 libqt4-xml libqt4-test libqt4-script libqt4-network libqt4-dbus python-qt4 python-qt4-gl libgle3 python-dev
```
* **Clone the PyV8 directory from the github and install it using the following commands:**
```
  $ sudo git clone https://github.com/buffer/pyv8.git
  $ cd pyv8
  $ sudo easy_install greenlet
  $ sudo easy_install gevent
  $ python setup.py build
  $ sudo python setup.py install
```
 **Note:** If you had any issue in Java PyV8 engine while executing the yaml do this additionally. **Restart your machine** 

**Yaml in a nutshell**: It contains five tasks namely shutoff_vm_detail, calculate_hours, create_snapshot send_email and delete_vm. Each and every tasks having mistral actions. 

## Following are the brief of every actions:

* **shutoff_vm_detail:**

  This task calls Nova action "**nova.servers_list**" that returns information about all servers in a tenant as JSON list. What we really need here is to extract instance shuttoff date, number of machine in shutoff state, VM IDs, VM names and IP addresses. In order to do that we need to declare clause that holds data in respective variables.
  
  In this task we are using special task property assigned with so that the result doesnre not interested in all information that Nova returns, only shutoff date, number of machine in shutoff state, VM IDs, VM names, and IPs,  are relevant. This makes sense to do because even if we have a tenant with 30-40 virtual servers all information about them returned by Nova will take ~100 KB of disk space. 

* **calculate_hours:**

  This task uses Java action "**std.javascript**" by using java script will get the current date of the machine then for loop used for calculating the VMs shoutoff timings in hours one by one. After that if any of the machine is in shutfoff state for more than 24 hours then statements inside the if loop will be executed eventually it will return VM IDs, VM names, and VM Ips.

  In order to use the variables(vm_ids,vm_names,vm_ips) outside the java script, we need to pulish it into variables(ids,names,ips). After completing this task yaml goes into next task.

* **create_snapshot:** 

 In this yaml part contains the action called "**nova.servers_backup**". The role of this is action is taking the backup when it becomes more safer side before deleting it. once this job is done calls the next action.

* **send_email:** 
  
  Our YAML is more worthy when the ability to notify a cloud administrator. In order to do that we need action called "**std.email**". After words delete part being executed.

* **delete_vm:**
  
  In this part we calls Nova action "**nova.servers_delete**". Here we are using special task it is functionality to iterate over a list of server id to delete the VMs.

Last but not the least, this script only works if we passed data that is needed to send an email and taking backup, Json file as an input parameter.

The *_delete_input.json_* content are as followed:

      {
       "from_email": "form_email@example.com",
       "to_email": "to_email@example.com",
       "smtp_server": "smtp_server",
       "smtp_password": "password",
       "backup_type": "once",
       "rotation":"1"
       }         
// Replace the input with valid parameters. In our case we used gmail smtp server  ( smtp.gmail.com:587 ) for sending mail from our gmail account. In order to use gmail go to this link https://www.google.com/settings/security/lesssecureapps and make sure that Access for less secure apps should be “**Trun on**” and use your gmail ID and password in the input parameter.


## Taking into Practical part: 

Uploading workflow to Mistral
```
    $ mistral workflow-create delete_vm.yaml
```
Note: In order to print all available workflows run: 
```
    $ mistral workflow-list
```
Executing the Workflow
```
    $ mistral execution-create delete_vm delete_input.json
```
