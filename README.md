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
aws configure

ccoctl aws create-key-pair
```

# Step 3
```
ccoctl aws create-identity-provider --name=<unique-s3-resource-name> --region=<aws_region> --public-key-file=serviceaccount-signer.public
```

# Step 4
```
RELEASE_IMAGE=\$(openshift-install version | awk '/release image/ {print $3}')
```

# Step 5
```
oc adm release extract --from=$RELEASE_IMAGE --credentials-requests --included --install-config=install-config.yaml --to=credrequests
```

# Step 6
```
ccoctl aws create-iam-roles --name=<unique-s3-resource-name> --region=<aws_region> --credentials-requests-dir=credrequests --identity-provider-arn=<arn-from-previous-command>
```

# Step 7
```
openshift-install create manifests --dir ocp-install
```

# Step 8
```
cp manifests/* ./ocp-install/manifests/
```

# Step 9
```
cp -a tls ocp-install/.
```

# Step 10
```
openshift-install create cluster --dir ocp-install --log-level=info
```


# Example Inventory
```yaml
openshift:
  hosts:
    localhost:
      ansible_connection: local
  vars:
    tmp_dir: /home/<user>/tmp
    aws:
      aws_access_key_id: <aws-access-key-id>
      aws_secret_access_key: <aws-secret-access-key>
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
      control_plane:
        instance_type: "m5.xlarge"
      workers:
        instance_type: "m5.2xlarge"
        replicas: "3"
      infras:
        instance_type: "m5.2xlarge"
      ssh_key: <ssh-key>
      pull_secret: '<pull-secret>'
```