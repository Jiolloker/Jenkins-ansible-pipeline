# Jenkins-ansible-pipeline
```
docker-compose up -d
docker exec -it <container id> cat /var/jenkins_home/secrets/initialAdminPassword
Configure pipeline normally
Manage credentials of a github and ssh key credentials
Install ansible pluggin and configure ansible on global tool config.
```

# Ansible dynamic inventory test on AWS
Enable plugin in the ansible.cfg.
```
[inventory]
enable_plugins = aws_ec2
```
Insert paths to credentials and key. Also configure the user to use for ssh
```
[defaults]
#Path to the yaml file with aws credentials and plugin
inventory      = /home/ubuntu/aws_ec2.yaml
#Especify remote_user name and path to key pair for ssh connection
remote_user = ec2-user
private_key_file = /home/ubuntu/my_key_pair.pem
```
Lastly, for testing purposes.
```
[ssh_connection]
#Disable ssh connection confirmation, not secure but it was disabled for testing.
ssh_args = -o StrictHostKeyChecking=no
```

Also, the structure of you aws credentials yaml should look like this. The aws_ec2.yaml file storage location is left to your own discretion.
```
---
plugin: aws_ec2
aws_access_key: <YOUR-AWS-ACCESS-KEY-HERE>
aws_secret_key: <YOUR-AWS-SECRET-KEY-HERE>
keyed_groups:
  - key: tags
    prefix: tag
  - prefix: instance_type
    key: instance_type
  - key: placement.region
    prefix: aws_region
```
After everything is done, you can test out with a few commands to see if it works.
```
ansible-inventory -i /opt/ansible/inventory/aws_ec2.yaml --list
```
```
{
    "_meta": {
        "hostvars": {
            "ec2-54-84-19-78.compute-1.amazonaws.com": {
                "ami_launch_index": 0,
                "architecture": "x86_64",
                "block_device_mappings": [
                    {
                        "device_name": "/dev/xvda",
                        "ebs": {
                            "attach_time": "2023-04-27T19:25:53+00:00",
                            "delete_on_termination": true,
                            "status": "attached",
                            "volume_id": "vol-0debce2fa376ead79"
                        }
                    }
                ],
                ....omitted 
```

```
ansible all -m ping
```
![Alt text](https://github.com/Jiolloker/Jenkins-ansible-pipeline/blob/ansible-dynamic-inventory/img/ping.png)
```
ansible-inventory --graph
```
![Alt text](https://github.com/Jiolloker/Jenkins-ansible-pipeline/blob/ansible-dynamic-inventory/img/graph.png)

You can also point to a especific machine to run ansible commands
```
ansible aws_region_us_east_1 -m ping
```
```
ec2-54-84-19-78.compute-1.amazonaws.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.9"
    },
    "changed": false,
    "ping": "pong"
}
```

Configuring playbooks to run on certain machines goes like this
```
---
- hosts: aws_region_us_east_1   # You change the hosts by tags returned with the command ansible-inventory --graph
  become: True
  tasks:
    - name: Update all packages
      dnf:
        name: "*"
        state: "latest"
```
Running a playbook
```
ansible-playbook playbook.yml
```
![Alt text](https://github.com/Jiolloker/Jenkins-ansible-pipeline/blob/ansible-dynamic-inventory/img/playbook.png)




Resource used
[link](https://devopscube.com/setup-ansible-aws-dynamic-inventory/)
