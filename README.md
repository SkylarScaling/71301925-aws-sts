# 71301925-aws-sts

# Cluster Provisioning Automation
## OpenShift on AWS with STS Cluster Provisioning Automation
For cluster provisioning on AWS with STS, with an existing VPC, use the following playbook:
```shell
ansible-playbook provision-cluster.yaml -i <inventory_file>
```
# TODO UPDATE
## Example Inventory
```yaml
all:
  hosts:
    localhost:
      ansible_connection: local
  vars:
    tmp_dir: /home/<user>/tmp
    aws:
      account_id: <aws_account_id>
      aws_access_key_id: <aws_access_key_id>
      aws_secret_access_key: <aws_secret_access_key>
      aws_region: us-east-2
      sts_resource_name: "{{ ocp_cluster.name }}"
    ocp_cluster:
      name: <cluster_name>
      base_domain: <base_domain>
    install_config:    
      cluster_name: "{{ ocp_cluster.name }}"
      base_domain: "{{ ocp_cluster.base_domain }}"
      cred_mode: Manual
      control_plane:
        instance_type: "m5.xlarge"
      workers:
        instance_type: "m5.2xlarge"
        replicas: "3"
      infras:
        instance_type: "m5.2xlarge"
      ssh_key: ssh-ed25519 <ssh_key>
      pull_secret: '<pull_secret>'
```

## Full Cluster Provisioning Automation
For full cluster automation, including automated creation of the AWS VPC, use the following playbook:

```shell
ansible-playbook provision-cluster-full.yaml -i <inventory_file>
```

## Example Inventory
```yaml
all:
  hosts:
    localhost:
      ansible_connection: local
  vars:
    admin_user: <admin_user>
    admin_password: <admin_pw>
    tmp_dir: /home/<user>/tmp
    aws:
      account_id: <aws_account_id>
      aws_access_key_id: <aws_access_key_id>
      aws_secret_access_key: <aws_secret_access_key>
      aws_region: us-east-2
      sts_resource_name: "{{ ocp_cluster.name }}"
    ocp_cluster:
      name: <cluster_name>
      base_domain: <base_domain>
    install_config:    
      cluster_name: "{{ ocp_cluster.name }}"
      base_domain: "{{ ocp_cluster.base_domain }}"
      cred_mode: Manual
      control_plane:
        instance_type: "m5.xlarge"
      workers:
        instance_type: "m5.2xlarge"
        replicas: "3"
      infras:
        instance_type: "m5.2xlarge"
      ssh_key: ssh-ed25519 <ssh_key>
      pull_secret: '<pull_secret>'
```

# Manual STS Configuration

## Step 1
### ocp-install/install-config.yaml
```yaml
...
apiVersion: v1
baseDomain: example.com
credentialsMode: Manual  # <-- Add This
# ...
...
```

## Step 2
```
mkdir -p /home/ec2-user/install-dir /home/ec2-user/sts
```

## Step 3
```
RELEASE_IMAGE=$(openshift-install version | awk '/release image/ {print $3}')
```

## Step 4
```
oc adm release extract --from=$RELEASE_IMAGE --credentials-requests --included --install-config=install-config.yaml --to=credrequests
```

## Step 5
> \<TODO> This should be automated and owned by the AWS team as a prerequisite

```shell
$ cd /home/ec2-user/sts

# Creates serviceaccount-signer and tls directory with key
$ ccoctl aws create-key-pair 

# Creates Identity Provider json files (01-04) and manifests directory with cluster auth config
$ ccoctl aws create-identity-provider \
  --name={{ aws.sts_resource_name }} \
  --region={{ aws_region }} \
  --public-key-file=serviceaccount-signer.public \
  --dry-run 

# <TODO> Output from dry-run should be created in AWS

# Creates IAM Role json files (05-06) and additional credentials yaml files in manifests directory
$ ccoctl aws create-iam-roles \
  --name={{ aws.sts_resource_name }} \
  --region={{ aws_region }} \
  --credentials-requests-dir=/home/ec2-user/sts/credrequests \
  --identity-provider-arn=arn:aws:iam::{{ aws.account_id }}:oidc-provider/{{ aws.sts_resource_name }}-oidc.s3.{{ aws_region }}.amazonaws.com \
  --dry-run 
  
# <TODO> Output from dry-run should be created in AWS
# <TODO> Output from manifests and tls directory should be sent to OCP team for use in the following steps
```

## Step 6
```
openshift-install create manifests --dir /home/ec2-user/install-dir
```

## Step 7
```
cp -r /home/ec2-user/sts/manifests/. /home/ec2-user/install-dir/manifests/
```

## Step 8
```
cp -r /home/ec2-user/sts/tls /home/ec2-user/install-dir/
```

## Step 9
```
<Cluster Install>
```