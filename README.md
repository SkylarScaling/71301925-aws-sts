# 71301925-aws-sts

# Cluster Provisioning Automation
## OpenShift on AWS with STS Cluster Provisioning Automation
For cluster provisioning on AWS with STS, with an existing VPC, use the following playbook:
```shell
ansible-playbook provision-cluster.yaml -i <inventory_file>
```

## Example Inventory
```yaml
all:
  hosts:
    localhost:
      ansible_connection: local
  vars:
    home_dir: /home/user
    tmp_dir: /home/user/tmp
    prefix_for_name: openshift-sts
    openshift_version: "4.20"
    ocp_patch_version: "13"
    ocp_cluster:
      name: <CLUSTER_NAME>
      base_domain: <BASE_DOMAIN>
    aws:
      account_id: <AWS_ACCOUNT_ID>
      aws_access_key_id: <AWS_ACCESS_KEY_ID>
      aws_secret_access_key: <AWS_SECRET_ACCESS_KEY>
      aws_region: <AWS_REGION>
      auth_s3_bucket: <S3_BUCKET_FOR_AUTH>
      sts_resource_name: "{{ ocp_cluster.name }}"
      # --- Existing Infrastructure, if it Already Exists ---
      vpc_id: vpc-xxxxx
      hosted_zone_id: ZXXXXXXXX
      public_subnets:
        - subnet-xxxxxx
        - subnet-xxxxxx
        - subnet-xxxxxx
      private_subnets:
        - subnet-xxxxxx
        - subnet-xxxxxx
        - subnet-xxxxxx
    cft:
      # --- Instance Sizes ---
      bootstrap_type: "m5.large"
      master_type: "m5.xlarge"
      worker_type: "m5.xlarge"
    install_config:
      cluster_name: "{{ ocp_cluster.name }}"
      base_domain: "{{ ocp_cluster.base_domain }}"
      cred_mode: Manual # Uncomment for STS
      manual_subnets: true # Uncomment for manual subnets
      machine_network:
        cidr: "172.16.0.0/16"
      workers:
        replicas: "3"
      ssh_key: "{{ lookup('file', '~/.ssh/sts_ed25519.pub') }}"
      ssh_private_key: "~/.ssh/sts_ed25519"
      pull_secret: '<PULL_SECRET_CONTENTS>'
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
    home_dir: /home/user
    tmp_dir: /home/user/tmp
    prefix_for_name: openshift-sts
    openshift_version: "4.20"
    ocp_patch_version: "13"
    ocp_cluster:
      name: <CLUSTER_NAME>
      base_domain: <BASE_DOMAIN>
    aws:
      account_id: <AWS_ACCOUNT_ID>
      aws_access_key_id: <AWS_ACCESS_KEY_ID>
      aws_secret_access_key: <AWS_SECRET_ACCESS_KEY>
      aws_region: <AWS_REGION>
      auth_s3_bucket: <S3_BUCKET_FOR_AUTH>
      sts_resource_name: "{{ ocp_cluster.name }}"
    cft:
      # --- Instance Sizes ---
      bootstrap_type: "m5.large"
      master_type: "m5.xlarge"
      worker_type: "m5.xlarge"
    install_config:
      cluster_name: "{{ ocp_cluster.name }}"
      base_domain: "{{ ocp_cluster.base_domain }}"
      cred_mode: Manual # Uncomment for STS
      manual_subnets: true # Uncomment for manual subnets
      machine_network:
        cidr: "172.16.0.0/16"
      workers:
        replicas: "3"
      ssh_key: "{{ lookup('file', '~/.ssh/sts_ed25519.pub') }}"
      ssh_private_key: "~/.ssh/sts_ed25519"
      pull_secret: '<PULL_SECRET_CONTENTS>'
```