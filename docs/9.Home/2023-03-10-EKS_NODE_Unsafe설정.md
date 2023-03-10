# EKS Pod의 Keep Alive 설정 

 - EKS pod의 keep alive 설정을 위해 unsafe sysctls enable하는 방법
 - unsafe sysctls enable은 kubelet extra arguments에 '--allowed-unsafe-sysctls' 파라미터를 통해 가능하십니다 [1]. 
 - EKS에서 kubelet arguments는 기본적으로 node가 생성될 때 bootstrap 스크립트[2]에 의해 설정됩니다. 
 - '--allowed-unsafe-sysctls' 파라미터가 설정된 kubelet을 기동하는 node group을 생성하기 위해, eksctl config file 설정 [3] 중 'overrideBootstrapCommand' 설정을 
    통해 bootstrap script를 지정하실 수 있으며 해당 설정을 사용하기위해서는 Custom AMI가 지정되어야 합니다. 
    
 - 참고로 한번 생성된 node group의 설정을 수정하는 것은 불가능하므로 [4] 새로운 설정을 위해서는 node group을 새로 생성해주셔야 합니다.

참고 코드 스니펫입니다.

pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep"]
    args: ["1d"]
  securityContext:
    sysctls:
    - name: net.ipv4.tcp_keepalive_intvl
      value: "60"
```

eksctl-config.yaml
```
apiVersion: eksctl.io/v1alpha5 
kind: ClusterConfig

metadata:
  name: <cluster name>
  region: <region>

managedNodeGroups:
- name: <nodegroup name>
  ami: <ami id>
  amiFamily: AmazonLinux2
  minSize: 0
  maxSize: 10
  desiredCapacity: 2
  ssh:
    allow: true
    publicKeyName: <public key>
  overrideBootstrapCommand: |
    #!/bin/bash
    set -ex
    /etc/eks/bootstrap.sh <cluster name> --kubelet-extra-args "--node-labels=alpha.eksctl.io/cluster-name=<cluster name>,alpha.eksctl.io/nodegroup-name=<nodegroup-name>,eks.amazonaws.com/nodegroup=<nodegroup-name>,eks.amazonaws.com/nodegroup-image=<ami  id> --allowed-unsafe-sysctls 'net.ipv4.tcp_keepalive_intvl'"
```

- 테스트 결과
$ kc exec -it ubuntu -- sysctl -a | grep -i keepalive
```
net.ipv4.tcp_keepalive_intvl = 60
net.ipv4.tcp_keepalive_probes = 9
net.ipv4.tcp_keepalive_time = 7200
```

- 위의 eksctl config file에서는 '--allowed-unsafe-sysctls' 파라미터 테스트를 위해 overrideBootstrapCommand의 최소한의 설정을 한 것
- 환경을 고려하여 충분한 테스트 후 적용하시길 추천함
- 관련 더 자세한 내용은 다음 문서[5, 6, 7]를 참조 가능

- 참고로 일시적인 파라미터 테스트를 위해 노드 접속 후 직접 kubelet config file을 수정하시어 kubelet restart 하셔서 확인해 볼수 있음


[1] https://kubernetes.io/docs/tasks/administer-cluster/sysctl-cluster/  
[2] https://github.com/awslabs/amazon-eks-ami/blob/master/files/bootstrap.sh  
[3] https://eksctl.io/usage/schema/#nodeGroups-overrideBootstrapCommand  
[4] https://eksctl.io/usage/managing-nodegroups/#nodegroup-immutability  
[5] https://aws.amazon.com/blogs/containers/introducing-launch-template-and-custom-ami-support-in-amazon-eks-managed-node-groups/  
[6] https://eksctl.io/usage/eks-managed-nodes/#creating-managed-nodegroups  
[7] https://eksctl.io/announcements/nodegroup-override-announcement/  
