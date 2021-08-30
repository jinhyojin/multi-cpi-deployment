# 다른 IaaS PaaS-TA 배포
- 본 문서는 AWS, OpenStack, vSphere 에 대한 가이드만 제공
- 본 문서는 paasta-deployment v5.5.3을 기준으로 작성

## 01.Multi-cpi Network 설정
[PaaS-TA OpenVPN Release](https://github.com/jinhyojin/openvpn-deployment) 를 사용하여 각 인프라의 OpenVPN 서버와 클라이언트를 모두 연결 
![guide_image1](https://github.com/jinhyojin/multi-cpi-deployment/blob/main/guide/images/openvpn.png)

## (임시) Multi-cpi download 
> paasta-deployment/bosh/multi-cpi/ 에 추가예정
```
$ PaaS-TA_HOME = /home/ubuntu/workspace/paasta-5.5.3/deployment/paasta-deployment
$ cd ${PaaS-TA_HOME}/bosh
$ git clone https://github.com/jinhyojin/multi-cpi-deployment.git
$ mv multi-cpi-deployment/deployment multi-cpi
```

## 02.Multi-cpi director 설정 
> 새 인프라에 대한 cpi를 추가한다. 

<table>
<tr>
<td>파일명</td>
<td>설명</td>
</tr>
<tr>
<td>deploy-cpi-aws-secondary.yml</td>
<td>새 인프라가 AWS 일 경우 사용</td>
</tr>
<tr>
<td>deploy-cpi-openstack-secondary.yml</td>
<td>새 인프라가 OpenStack 일 경우 사용</td>
</tr>
<tr>
<td>deploy-cpi-vsphere-secondary.yml</td>
<td>새 인프라가 vSphere 일 경우 사용</td>
</tr>
<tr>
<td>deploy-cpi-registry-secondary.yml</td>
<td>기존 인프라가 vSphere 일 경우 사용</td>
</tr>
</table>
<br>

$ vim ${PaaS-TA_HOME}/bosh/deploy-aws.sh
```diff
 bosh create-env bosh.yml \
 	--state=aws/state.json \
 	--vars-store=aws/creds.yml \
 	-o aws/cpi.yml \
+ 	-o multi-cpi/deploy-cpi-openstack-secondary.yml \
 	-o uaa.yml \
 	-o credhub.yml \
 	-o jumpbox-user.yml \
 	-o cce.yml \
 	-l aws-vars.yml
```
<br>

$ vim ${PaaS-TA_HOME}/bosh/deploy-openstack.sh
```diff
 bosh create-env bosh.yml \
 	--state=openstack/state.json \
 	--vars-store=openstack/creds.yml \
 	-o openstack/cpi.yml \
+ 	-o multi-cpi/deploy-cpi-vsphere-secondary.yml \
 	-o uaa.yml \
 	-o credhub.yml \
 	-o jumpbox-user.yml \
 	-o cce.yml \
 	-o openstack/disable-readable-vm-names.yml \
 	-l openstack-vars.yml
```
<br>

$ vim ${PaaS-TA_HOME}/bosh/deploy-vsphere.sh
```diff
 bosh create-env bosh.yml \
 	--state=vsphere/state.json \
 	--vars-store=vsphere/creds.yml \
 	-o vsphere/cpi.yml \
 	-o vsphere/resource-pool.yml  \
+ 	-o multi-cpi/deploy-cpi-aws-secondary.yml \
+ 	-o multi-cpi/deploy-cpi-registry-secondary.yml \
 	-o uaa.yml  \
 	-o credhub.yml  \
 	-o jumpbox-user.yml  \
 	-l vsphere-vars.yml
```
<br>

> bosh director 배포
```
$ cd ${PaaS-TA_HOME}/bosh/
$ ./deploy-${IaaS}.sh
``` 

## 03.Multi-cpi cpi-config 설정 
<table>
<tr>
<td>파일명</td>
<td>설명</td>
</tr>
<tr>
<td>cpi-config.yml</td>
<td>multi-cpi 추가를 위한 template file</td>
</tr>
<tr>
<td>cpi-vars.yml</td>
<td>multi-cpi 설정 파일</td>
</tr>
</table>
<br>

> 새 인프라에 대한 정보를 입력한다. 
$ vim ${PaaS-TA_HOME}/bosh/multi-cpi/cpi-vars.yml
```
# MULTI-CPI VARIABLE :: aws
aws_access_key_id: "XXXXXXXXXXXXXXX"                    # AWS Access Key
aws_secret_access_key: "XXXXXXXXXXXXX"                  # AWS Secret Key
aws_default_key_name: "paasta-key"                      # AWS Key Name
aws_default_security_groups: ["paasta-security"]        # AWS Security-Group
aws_region: "ap-northeast-2"                            # AWS Region

# MULTI-CPI VARIABLE :: openstack
openstack_auth_url: "http://XX.XXX.XX.XX:XXXX/v3/"      # Openstack Keystone URL
openstack_username: "XXXXXX"                            # Openstack User Name
openstack_password: "XXXXXX"                            # Openstack User Password
openstack_domain: "XXXXXX"                              # Openstack Domain Name
openstack_project: "PaaS-TA"                            # Openstack Project
openstack_region: "RegionOne"                           # Openstack Region
openstack_default_key_name: "paasta-key"                # Openstack Key Name
openstack_default_security_groups: ["paasta-security"]  # Openstack Security Group

# MULTI-CPI VARIABLE :: vsphere
vcenter_ip: "XX.XX.XXX.XX"                              # vCenter Private IP
vcenter_user: "XXXXX"                                   # vCenter User Name
vcenter_password: "XXXXXX"                              # vCenter User Password
vcenter_cluster: "PaaS-TA"                              # vCenter Cluster Name
vcenter_ds: "PaaS-TA_Storage"                           # vCenter Data Storage Name
vcenter_disks: "PaaS-TA_Disks"                          # vCenter Disk Name
vcenter_dc: "PaaS-TA_DC"                                # vCenter Data Center Name
vcenter_templates: "PaaS-TA_Templates"                  # vCenter Templates Name
vcenter_vms: "PaaS-TA_VMs"                              # vCenter VMS Name
```
<br>

> 필요없는 cpi 정보는 주석처리 한다. 
$ vim ${PaaS-TA_HOME}/bosh/multi-cpi/cpi-config.yml
```
cpis:
- name: aws-cpi
  type: aws
  properties:
    access_key_id: ((aws_access_key_id))
    secret_access_key: ((aws_secret_access_key))
    default_key_name: ((aws_default_key_name))
    default_security_groups: ((aws_default_security_groups))
    region: ((aws_region))

- name: openstack-cpi
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

- name: vsphere-cpi
  type: vsphere
  properties:
    host: ((vcenter_ip))
    user: ((vcenter_user))
    password: ((vcenter_password))
    default_disk_type: thin
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

> cpi 설정 적용 
```
$ bosh update-cpi-config ${PaaS-TA_HOME}/bosh/multi-cpi/cpi-config.yml
``` 

## 04.Multi-cpi stemcell 설정
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

AWS-OpenStack example:
```
$ bosh upload-stemcell light-bosh-stemcell-1.25-aws-xen-hvm-ubuntu-bionic-go_agent.tgz --fix

$ bosh upload-stemcell bosh-stemcell-1.25-openstack-kvm-ubuntu-bionic-go_agent.tgz --fix
```


## 05.Multi-cpi cloud-config 설정
<table>
<tr>
<td>파일명</td>
<td>설명</td>
</tr>
<tr>
<td>cloud-config-aws-openstack.yml</td>
<td>aws, openstack template file </td>
</tr>
<tr>
<td>cloud-config-openstack-vsphere.yml</td>
<td>openstack, vsphere template file</td>
</tr>
<tr>
<td>cloud-config-vsphere-aws.yml</td>
<td>vsphere, aws template file</td>
</tr>
</table>
<br>

> aws-openstack 또는 openstack-aws 를 사용하는 경우 
<br> 필수사항 : openstack flavor 추가 필요 (t2.small, t2.medium, t2.large, t2.xlarge)
```
bosh update-cloud-config ${PaaS-TA_HOME}/bosh/multi-cpi/cloud-config-aws-openstack.yml
```
<br>

> openstack-vsphere 또는 vpshere-openstack 을 사용하는 경우 
```
bosh update-cloud-config ${PaaS-TA_HOME}/bosh/multi-cpi/cloud-config-openstack-vsphere.yml
```
<br>

> vsphere-aws 또는 aws-vsphere 를 사용하는 경우 
```
bosh update-cloud-config ${PaaS-TA_HOME}/bosh/multi-cpi/cloud-config-vsphere-aws.yml
```

## 06.Multi-cpi PaaS-TA 배포
```
$ bosh update-runtime-config ${PaaS-TA_HOME}/bosh/runtime-config/dns.yml
$ bosh update-runtime-config --name=os-conf ${PaaS-TA_HOME}/bosh/runtime-config/os-conf.yml
$ cd ${PaaS-TA_HOME}/paasta/
$ ./deploy-${IaaS}.sh
``` 