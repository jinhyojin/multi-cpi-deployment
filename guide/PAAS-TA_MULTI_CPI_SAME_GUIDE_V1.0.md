# 동일 IaaS PaaS-TA 배포
- 본 문서는 AWS, OpenStack, vSphere 에 대한 가이드만 제공
- 본 문서는 paasta-deployment v5.5.3을 기준으로 작성

## 01.Multi-cpi Network 설정
[PaaS-TA OpenVPN Release](https://github.com/jinhyojin/openvpn-deployment) 를 사용하여 각 인프라의 OpenVPN 서버와 클라이언트를 모두 연결 
![guide_image1](https://github.com/jinhyojin/multi-cpi-deployment/blob/main/guide/images/openvpn.png)

## 02.Multi-cpi cpi-config 설정 
> bosh director 배포 (이미 배포되어 있을 경우 다음 단계로 이동)
```
$ PaaS-TA_HOME = /home/ubuntu/workspace/paasta-5.5.3/deployment/paasta-deployment
$ cd ${PaaS-TA_HOME}/bosh/
$ ./deploy-${IaaS}.sh
``` 
<br>

> 동일 IaaS간의 cpi-config 파일을 생성 
AWS example: vim cpis-aws.yml
```
cpis:
- name: aws-cpi-1
  type: aws
  properties:
    access_key_id: ((aws_access_key_id))
    secret_access_key: ((aws_secret_access_key))
    default_key_name: ((aws_default_key_name))
    default_security_groups: ((aws_default_security_groups))
    region: ((aws_region))
- name: aws-cpi-2
  type: aws
  properties:
    access_key_id: ((aws_access_key_id))
    secret_access_key: ((aws_secret_access_key))
    default_key_name: ((aws_default_key_name))
    default_security_groups: ((aws_default_security_groups))
    region: ((aws_region))
```
<br>

OpenStack example: vim cpis-openstack.yml
```
cpis:
- name: openstack-cpi-1
  type: openstack
  properties:
    auth_url: ((openstack_auth_url))
    username: ((openstack_username))
    api_key: ((openstack_password))
    domain: ((openstack_domain))
    project: ((openstack_project))
    region: ((openstack_region))
    default_key_name: ((openstack_default_key_name))
    default_security_groups: ((openstack_default_security_groups))
    human_readable_vm_names: true
- name: openstack-cpi-2
  type: openstack
  properties:
    auth_url: ((openstack_auth_url))
    username: ((openstack_username))
    api_key: ((openstack_password))
    domain: ((openstack_domain))
    project: ((openstack_project))
    region: ((openstack_region))
    default_key_name: ((openstack_default_key_name))
    default_security_groups: ((openstack_default_security_groups))
    human_readable_vm_names: true
```
<br>

vSphere example: vim cpis-vsphere.yml
```
cpis:
- name: vsphere-cpi-2
  type: vsphere
  properties:
    host: ((vcenter_ip))
    user: ((vcenter_user))
    password: ((vcenter_password))
    datacenters:
    - clusters:
      - { ((vcenter_cluster)): {}}
      datastore_pattern: ((vcenter_ds))
      disk_path: ((vcenter_disks))
      name: ((vcenter_dc))
      persistent_datastore_pattern: ((vcenter_ds))
      template_folder: ((vcenter_templates))
      vm_folder: ((vcenter_vms))
- name: vsphere-cpi-2
  type: vsphere
  properties:
    host: ((vcenter_ip))
    user: ((vcenter_user))
    password: ((vcenter_password))
    datacenters:
    - clusters:
      - { ((vcenter_cluster)): {}}
      datastore_pattern: ((vcenter_ds))
      disk_path: ((vcenter_disks))
      name: ((vcenter_dc))
      persistent_datastore_pattern: ((vcenter_ds))
      template_folder: ((vcenter_templates))
      vm_folder: ((vcenter_vms))
```
<br>

> cpi 설정 적용 ( ${IaaS} : aws, openstack, vsphere )
```
$ bosh update-cpi-config cpis-${IaaS}.yml
``` 
<br>

## 03.Multi-cpi stemcell 설정
> [stemcell download](https://bosh.cloudfoundry.org/stemcells/) 
<br>AWS의 경우 lite-stemcell 사용
```
$ wget https://storage.googleapis.com/bosh-aws-light-stemcells/1.25/light-bosh-stemcell-1.25-aws-xen-hvm-ubuntu-bionic-go_agent.tgz
$ wget https://storage.googleapis.com/bosh-core-stemcells/1.25/bosh-stemcell-1.25-openstack-kvm-ubuntu-bionic-go_agent.tgz
$ wget https://storage.googleapis.com/bosh-core-stemcells/1.25/bosh-stemcell-1.25-vsphere-esxi-ubuntu-bionic-go_agent.tgz
```
<br>

> stemcell upload (각 IaaS에 맞게 업로드, 이미 stemcell이 업로드 되어 있다면 재업로드 필요 )
<br>--fix 명령어 사용 

AWS example:
```
$ bosh upload-stemcell light-bosh-stemcell-1.25-aws-xen-hvm-ubuntu-bionic-go_agent.tgz --fix
```
<br>

## 04.Multi-cpi cloud-config 설정 
> cloud-config 의 azs 에서 각 인프라의 cpi-name 을 지정
```diff
 azs:
 - cloud_properties:
     availability_zone: ap-northeast-1a
   name: z1
+  cpi: aws-cpi-1
 - cloud_properties:
     availability_zone: ap-northeast-1a
   name: z2
+  cpi: aws-cpi-1
 - cloud_properties:
     availability_zone: ap-northeast-2a
   name: z3
+  cpi: aws-cpi-2
 - cloud_properties:
     availability_zone: ap-northeast-2a
   name: z4
+  cpi: aws-cpi-2
 ...
 ...
```
<br>

## 05.Multi-cpi PaaS-TA 배포
```
$ bosh update-runtime-config ${PaaS-TA_HOME}/bosh/runtime-config/dns.yml
$ bosh update-runtime-config --name=os-conf ${PaaS-TA_HOME}/bosh/runtime-config/os-conf.yml
$ cd ${PaaS-TA_HOME}/paasta/
$ ./deploy-${IaaS}.sh
``` 