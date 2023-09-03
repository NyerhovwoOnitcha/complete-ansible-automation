# Step 1: Create an ubuntu instance, install Jenkins and Ensure Jenkins is running

`sudo apt install default-jdk-headless`

```
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

![Install jenkins](./images/install%20jenkins.png)

`sudo systemctl status jenkins`

![Jenikins status](./images/jenkins%20status.png)

### By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group
 
### Perform initial Jenkins setup. From your browser access ` http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080`

### Retrieve the default admin password from your server using:

`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

### Install the suggested plugins and create an admin user


### Create a new Freestyle project ansible in Jenkins and point it to your ‘complete-ansible-automation’ repository.


### Configure Webhook in GitHub and set webhook to trigger ansible build.

![](./images/webhook%20images.png)

### Configure a Post-build job to save all (**) files, like you did it in Project 9.

https://github.com/NyerhovwoOnitcha/complete-ansible-automation/assets/101157174/831fe2d9-1e03-4b54-b7a6-5d4b82dea372

![](./images/webhook%20activated.png)

### Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder

![](./images/webhook%20activated2.png)

# STEP 2: Install ansible and begin ansible development

`sudo apt update`

`sudo apt install ansible`

![](./images/ansible%20version.png)

### Clone down your ansible-config-mgt repo to your Jenkins-Ansible instance

`git clone <complete ansible automation repo link>`

### In your ansible complete automation gitHub repository, create a new branch that will be used for development of a new feature.

### Checkout the newly created feature branch to your local machine and start building your code and directory structure

`git checkout -b ansible-feature`

![](./images/create%20ansible%20feature%20repo.png)

### Create a directory and name it playbooks – it will be used to store all your playbook files.

### Create a directory and name it inventory – it will be used to keep your hosts organised.
### Within the playbooks folder, create your first playbook, and name it common.yml
### Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively

![](./images/create%20structure.png)


### Update your dev inventory file with the following code (create the instances)

```
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ec2-user' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'

```

![](./images/dev%20inventory%20file.png)

# STEP 3: Create a common playbook

### Update your playbooks/common.yml file with following code:

```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest
```

###  Update GIT with the latest code, create a pull request and merge to main branch. Your webhook should be triggered and your ansible job should be built

### Once your code changes appear in master branch – Jenkins will do its job and save all the files (build artifacts) to /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/ directory on Jenkins-Ansible server.

![](./images/successful%20build.png)

### Head back on your terminal, checkout from the feature branch into the master, and pull down the latest changes.

`git checkout main`

`git fetch`

`git pull`

### Execute your playbook and verify it runs and wireshark is installed

`ansible-playbook -i inventory/dev.yml playbooks/common.yml`

![](./images/playbook%20runs.png)

![](./images/wireshark%20installed.png)

