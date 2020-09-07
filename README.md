1) Setting Up Terraform
2) Setting Up AWS CLI and Ansible
3) AWS IAM Permissions For Terraform
	a) Create an EC2 role and attach an instance profile to an EC2 instance to work within AWS.
	b) Attach the IAM policy directly to an IAM user created for this deployment and configure AWS CLI with it's credentials to allow Terraform permissions to deploy in AWS.
4) Terraform fmt, validate, plan, and apply
............................

Deploying Jenkins Master and Worker Nodes in AWS Behind

1) Log into the Terraform Controller node EC2 instance
2) https://github.com/rajpolakala/SampleTest.git
3) cd SampleTest/jenkins_master_worker 
4) Run the gen_ssh_key.yaml Ansible Playbook to generate SSH keypair
5) Deploy the Terraform Code
a) terraform init, terraform fmt, terraform validate, terraform plan, terraform apply
.............................

Monitoring

Please find it in monitor folder