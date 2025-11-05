# 71301925-aws-sts
# Manual STS Configuration

# Step 1
## ocp-install/install-config.yaml
```yaml
...
apiVersion: v1
baseDomain: example.com
credentialsMode: Manual  # <-- Add This
# ...
...
```

# Step 2
```
mkdir -p /home/ec2-user/install-dir /home/ec2-user/sts
```

# Step 3
```
RELEASE_IMAGE=$(openshift-install version | awk '/release image/ {print $3}')
```

# Step 4
```
oc adm release extract --from=$RELEASE_IMAGE --credentials-requests --included --install-config=install-config.yaml --to=credrequests
```

# Step 5
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

# Step 6
```
openshift-install create manifests --dir /home/ec2-user/install-dir
```

# Step 7
```
cp -r /home/ec2-user/sts/manifests/. /home/ec2-user/install-dir/manifests/
```

# Step 8
```
cp -r /home/ec2-user/sts/tls /home/ec2-user/install-dir/
```

# Step 9
```
<Cluster Install>
```


# Example Inventory
```yaml
all:
  hosts:
    localhost:
      ansible_connection: local
  vars:
    tmp_dir: /home/<user>/tmp
    aws:
      aws_region: <aws-region>
      sts_resource_name: <unique-resource-name>
    ocp_cluster:
      name: <cluster-name>
      base_domain: <base-domain>
      ocp_version: 4.18
    install_config:
      cluster_name: "{{ ocp_cluster.name }}"
      base_domain: "{{ ocp_cluster.base_domain }}"
      cred_mode: Manual
      aws_region: "{{ aws.aws_region }}"
      ssh_key: <ssh-key>
      pull_secret: '<pull-secret>'
```