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
RELEASE_IMAGE=\$(openshift-install version | awk '/release image/ {print $3}')
```

# Step 4
```
oc adm release extract --from=$RELEASE_IMAGE --credentials-requests --included --install-config=install-config.yaml --to=credrequests
```

# Step 5
```
cd /home/ec2-user/sts && \
ccoctl aws create-all \
  --credentials-requests-dir /home/ec2-user/sts/credrequests/ \
  --name "<sts_suffix>" \
  --region "<aws_region>" \
  --create-private-s3-bucket
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
      aws_cred_mode: Manual # Uncomment for STS
    ocp_cluster:
      name: <cluster-name>
      base_domain: <base-domain>
      ocp_version: 4.19
    install_config:
      cluster_name: "{{ ocp_cluster.name }}"
      base_domain: "{{ ocp_cluster.base_domain }}"
      aws_region: "{{ aws.aws_region }}"
      ssh_key: <ssh-key>
      pull_secret: '<pull-secret>'
```