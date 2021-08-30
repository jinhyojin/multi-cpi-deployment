# PaaS-TA Multi-cpi 기본 구성
- 본 문서는 AWS, OpenStack, vSphere 에 대한 가이드만 제공


## 01.Multi-cpi Network 설정
[PaaS-TA OpenVPN Release](https://github.com/jinhyojin/openvpn-deployment) 를 사용하여 각 인프라의 OpenVPN 서버와 클라이언트를 모두 연결 
![guide_image1](https://github.com/jinhyojin/multi-cpi-deployment/blob/main/guide/images/openvpn.png)

## 02.Multi-cpi cpi-config 설정 
<table>
<tr>
<td>속성명</td>
<td>필수여부</td>
<td>설명</td>
</tr>
<tr>
<td>name</td>
<td>문자열,필수</td>
<td>CPI의 고유 이름</td>
</tr>
<tr>
<td>type</td>
<td>문자열,필수</td>
<td>CPI 유형</td>
</tr>
<tr>
<td>properties</td>
<td>해시,필수</td>
<td>CPI가 IaaS에서 리소스를 인증하고 프로비저닝할 수 있도록 각 호출에 대해 CPI에 제공할 속성 집합</td>
</tr>
</table>
<br>

AWS example:
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
```
<br>

OpenStack example:
```
cpis:
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
```
<br>

vSphere example:
```
cpis:
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

Multi IaaS example:
```
cpis:
- name: openstack-cpi-1
  type: openstack
  properties:
    ...
- name: openstack-cpi-2
  type: openstack
  properties:
    ...
```
<br>

## 03.Multi-cpi stemcell upload
> Stemcell은 특정 cpi에 할당되도록 업로드, 이미 stemcell이 업로드 되어 있다면 재업로드 필요
<br>AWS의 경우 lite-stemcell 사용
```
bosh upload-stemcell ####.tgz --fix
```
<br>

## 04.Multi-cpi cloud-config 설정 
> cloud-config 의 azs 에서 각 인프라의 cpi-name 을 지정
```
azs:
- name: z1
  cpi: openstack-cpi-1
- name: z2
  cpi: openstack-cpi-2
...
```