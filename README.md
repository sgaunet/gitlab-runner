This project contains manifests to deploy your own gitlab-runner in kubernetes. You will also find options to deploy it in a specific nodegroup (on AWS EKS with eksctl).

[A medium story give more details.](https://plyk.medium.com/gitlab-runner-in-eks-722250e006c0)

# Nodegroup to create with eksctl

```
  - name: ng-cicd
    amiFamily: AmazonLinux2
    privateNetworking: true
    instanceType: t3.medium
    desiredCapacity: 1
    minSize: 1
    maxSize: 1
    volumeSize: 60
    volumeType: gp2
    availabilityZones: ["eu-west-3a", "eu-west-3b","eu-west-3c"]
    securityGroups:
      attachIDs: ["sg-********************"]
    labels: { role: cicd }
    preBootstrapCommands:
      - sed -i '/^KUBELET_EXTRA_ARGS=/a KUBELET_EXTRA_ARGS+=" --register-with-taints=cicd=true:NoSchedule"' /etc/eks/bootstrap.sh
    tags:
      nodegroup-role: cicd
      k8s.io/cluster-autoscaler/node-template/taint/cicd: "true:NoSchedule"
    iam:
      attachPolicyARNs:
        - arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy
        - arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy
        - arn:aws:iam::aws:policy/ElasticLoadBalancingFullAccess
        - arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly
      withAddonPolicies:
        Autoscaler: false
        ebs: true
        efs: true
        cloudwatch: true
        albIngress: true
```

# Deploy

Apply the manifests in this order :

Set your context as admin.

* ns.yaml
* <env>/gitlab-runner-config.vault.yml
* gitlab-runner-service-account.yaml
* gitlab-runner-deployment.yaml

