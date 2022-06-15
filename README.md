## Quay All In One Install Guide

### 1. Pre-Requisites

- RHEL 8 with `podman v3.3` 설치
- Quay 서비스를 위한 정규화된 도메인 이름 (DNS를 통해 확인하거나 /etc/hosts 설정)



### 2. Installation

- 설치 패키지 사이트에서 파일 다운로드 

  (https://github.com/quay/mirror-registry/releases)

  ```bash
  wget -P ./ https://github.com/quay/mirror-registry/releases/download/v0.1.4/openshift-mirror-registry-online.tar.gz
  ```

- 설치 파일 압축 해제

  ```bash
  tar -xvf mirror-registry-online.tar.gz
  ```

- 파일 확인

  ```bash
  [root@bastion test]# ls -rlt
  total 519808
  -rw-r--r--. 1 1001  121      5580 Aug 26 14:19 README.md
  -rw-r--r--. 1 1001  121 389075456 Aug 26 14:20 execution-environment.tar
  -rwxr-xr-x. 1 root root   8015176 Aug 26 14:21 openshift-mirror-registry
  -rw-r--r--. 1 root root 135179084 Dec  8 09:02 openshift-mirror-registry-online.tar.gz
  ```

- `registry.redhat.io` 로그인

  ```bsh
  [root@bastion quay-test]# podman login registry.redhat.io
  Username: ${USERNAME}
  Password: ${PASSWORD}
  Login Succeeded!
  ```

- 설치

  ```bash
  [root@bastion test]# ./openshift-mirror-registry install
  ```

- 설치 완료 메시지

  ```bash
  --- 생략 ---
  TASK [mirror_appliance : Create init user] **********************************************************************************************
  included: /runner/project/roles/mirror_appliance/tasks/create-init-user.yaml for root@localhost
  
  TASK [mirror_appliance : Creating init user at endpoint https://localhost:8443/api/v1/user/initialize] **********************************
  ok: [root@localhost]
  
  PLAY RECAP ******************************************************************************************************************************
  root@localhost             : ok=36   changed=12   unreachable=0    failed=0    skipped=4    rescued=0    ignored=0
  
  INFO[2022-01-17 16:13:24] Quay installed successfully
  INFO[2022-01-17 16:13:24] Quay is available at https://localhost:8443 with credentials (init, 3lRTK9py7H80Z2QsSrnq4a5u1XIFNB6z)
  ```

- 로그인

  ```bash
  [root@bastion test]# podman login -u init -p 3lRTK9py7H80Z2QsSrnq4a5u1XIFNB6z --tls-verify=false ${QUAY_REGISTRY_IP}:{$PORT}
  Login Succeeded!
  ```

- 콘솔 접속

  크롬에서는 보안이 인증되지 않은 인증서이기 때문에 접속이 차단되므로, 파이어폭스를 이용하여 접속합니다.

  ![01_quay_console](https://github.com/justone0127/all-in-one-Quay-Instance/blob/main/images/01_quay_console.png)

- 계정은 위에 정보를 확인하여 로그인 (user: init / password: 3lRTK9py7H80Z2QsSrnq4a5u1XIFNB6z )

  ![02_console_login](https://github.com/justone0127/all-in-one-Quay-Instance/blob/main/images/02_console_login.png)

### 3. 계정 생성

- Create Account 메뉴를 통해 새로운 계정 생성

  ![03_new_account](https://github.com/justone0127/all-in-one-Quay-Instance/blob/main/images/03_new_account.png)

- 새로운 계정으로 로그인

  ![04_new_account_login](https://github.com/justone0127/all-in-one-Quay-Instance/blob/main/images/04_new_account_login.png)

### 4. Image Push

- 새로운 계정으로 로그인

  ```bash
  [root@bastion test]# podman login -u ${QUAY_USER} -p ${QUAY_PWD} --tls-verify=false ${QUAY_REGISTRY_IP}:{$PORT}
  Login Succeeded!
  ```

- Image Push

  외부 이미지 레지스트리 이미지를 해당 레지스트리로 푸시해보도록 하겠습니다.

  ```bash
  skopeo copy --src-creds ${USERNAME}:${PASSWORD} --src-tls-verify=false --dest-creds ${QUAY_USER}:${QUAY_PWD} --dest-tls-verify=false docker://registry.redhat.io/rhscl/httpd-24-rhel7:2.4-152 docker://${QUAY_REGISTRY_IP}:{$PORT}/repository/admin/test/httpd-24-rhel7:2.4-152
  ```
  
- Image Push 결과

  ![05_image_push_result](https://github.com/justone0127/all-in-one-Quay-Instance/blob/main/images/05_image_push_result.png)

  

  
