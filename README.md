## Three-tier infrastructure design

![Alt text](images/diagram.png?raw=true "Title")

### Bastion Host EIP allocation

To have always the same EIP associated to the bastion host I wrote a userdata script to attach the EIPBastionHost to the instance during the instance boot up, if you terminate the instance or if a fault appen to an AZ, the istance can be started in the second AZ.

To be sure to select a free EIP this command is used at the moment:

```
aws ec2 describe-addresses --output text | awk '{ if ( $3 !~ /eipassoc-*/ ) print $4 } | head -1'
```

Pay attention to not forget other EIP not associated.

To perform the EIP association an IAM user with the following policy is required:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "ec2:ReleaseAddress",
                "ec2:DisassociateAddress",
                "ec2:DescribeMovingAddresses",
                "ec2:DescribeAddresses",
                "ec2:AssignIpv6Addresses",
                "ec2:MoveAddressToVpc",
                "ec2:AssignPrivateIpAddresses",
                "ec2:AllocateAddress",
                "ec2:RestoreAddressToClassic",
                "ec2:DescribePublicIpv4Pools",
                "ec2:UnassignIpv6Addresses",
                "ec2:UnassignPrivateIpAddresses",
                "ec2:AssociateAddress"
            ],
            "Resource": "*"
        }
    ]
}
```

### EC2 Instances initialization

pull code from s3 or github or artifactory

### CI/CD and Configuration Management

To have CI/CD and CM use an on-premise AWS installation which use the bastion as a ssh proxy host and the AWS dynamic inventory to connect to the instances, to install AWS on Centos with docker, you can use this script:

```
#!/bin/bash
yum install epel-release -y
yum install -y yum-utils device-mapper-persistent-data lvm2 ansible git python-devel python-pip vim-enhanced docker
pip install docker docker-compose
sleep 5
systemctl start docker
systemctl enable docker
cd /tmp
git clone https://github.com/ansible/awx.git
sleep 5
cd /tmp/awx
git clone https://github.com/ansible/awx-logos.git
sleep 5
ansible-playbook -i /tmp/awx/installer/inventory /tmp/awx/installer/install.yml
useradd -d /home/automation -s /bin/bash automation
```
