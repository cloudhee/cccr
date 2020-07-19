### SMB 스토리지

SMB: TCP/IP 위에서 동작, 로컬 네트워크위에서 동작하는 시스템끼리는 디렉토리를 공유할수 없다.

CIFS는 SMB의확장 프로토콜.  SMB와 다른점은 로컬 네트워크뿐아니라 다른시스템에서 공유하는 디렉토리도 접근할 수 있다는것.



### SAMBA

유닉스 시스템에 구현하는 것으로 시작되었으며, 현재는 윈도우,리눅스,유닉스,맥 간의 데이터 공유하는 프로젝트로 발전.

SAMBA를 이용하여 SBM 스토리지 공유하거나 SMB 공유 사용, 로컬 네트워크에 프린트공유, 공유중인 프린트에도 연결 가능



#### 패키지 설치

```
yum -y install samba-common	
```

#### `/etc/samba/smb.conf`

samba-common 패키지에 포함되어있으며 global 섹션, 사용자 정의섹션, home 섹션, printers 섹션 등으로 이루어져있다.



#### 서버 설정💻

-----

##### 패키지설치

```
yum -y install samba samba-client 
```

> smb 사용자를 관리 할 때 사용하는 smbpasswd 명령, 삼바 서버에서 제공하는 smb 공유를 확인할 때 사용하는 smbclient 도구 제공

##### 공유 디렉토리 생성

```
mkdir -p /share/samba
```

##### SELinux가 사용중일 때 컨텍스트 설정 방법

SELinux는 간단히 말해 리눅스 보안을 강화해주는 커널 모듈.

```
semanage fcontext -s -t samba_share_t '/share/samba(/.*)?'
restorecon -RFv /share/samba/
```

> semanage를 이용해서 보안 컨텍스트 추가할 수 있음 
>
> restorecon은 잘못된 보안 컨텍스트를 복구해주는 명령어 
>
> SELinux에서 samba를 사용하도록 설정해줘야 접근가능하고 그렇지않으면 오류가 난다고 함. [출처](https://suwoni-codelab.com/linux/2019/02/08/linux-samba-selinux/)
>
> 책에 관련 설명은 챕터 2 리눅스보안-SELinux에 나와있음

##### 설정 파일에 사용자 정의 섹션 생성

```
vi /etc/samba/smb.conf
```

`smb.conf`

``` 
[share]
	comment = Samba Test
	path = /share/samba
	writable = yes
	write list = smbuser, @smbgroup
	valid users = smbuser, @smbgroup, @wheel
	browseable = no
```

> 지정되지 않은 값은 global 섹션에 지정되어있는 값이나 디폴트 값이 자동으로 지정됨

설정 저장 후, `testparm` 명령을 사용하여 설정에 문제가 없는지 확인.

##### 서비스 시작 및 방화벽 설정

```
systemctl start smb nmb
systemctl enable smb nmb
```

> nmb: netbios message block

##### 방화벽 규칙에 samba 서비스 추가

`````
firewall-cmd --permanent --add-service=samba
firewall-cmd --reload
`````

##### 사용자 추가 및 등록

```
useradd -s /sbin/nologin smbuser //smb 공유에서만 사용할 사용자인 경우 
smbpasswd -a smbuser 
```

등록된 SMB사용자 조회

```
pdbedit --list
```





#### SMB 클라이언트 연결 🖥

-----

##### 수동마운트

지속적으로 사용할 경우 유용, 네트워크 자원은 낭비될 수 있음

##### 패키지 설치

```
yum -y install cifs-utils // cifs 파일 시스템 지원하는 패키지
yum -y install samba-client // 삼바 서버에서 제공하는 공유섹션을 탐색하기위한 smbclient 도구 제공
```

##### 공유 영역 탐색

```
smbclient -L [server ip] -U smbuser
```

> `/etc/samba/smb.conf` 에서 browseable이 yes로 지정되어있어야 탐색이 가능함 (244p)

##### 설정파일 수정 후 서비스 재시작(server)

```
systemctl restart smb nmb
```

##### 마운트 포인트 생성 및 마운트

```
mkdir -p /mnt/share
```

- `/root/smb-auth`

```
username=smbuser
password=dkagh1.
domain=SAMBA
```

```
chmod 400 /root/smb-auth
```
- 마운트 명령어 형식
 `mount -o auth-information //server-address/smb-share-name mount-point`
	```
mount -a credentials=/root/smb-auth //[server ip]/share /mnt/share/
	```

 - `/etc/fstab`에 등록 

   ```
   //[server address]/share	/mnt/share	clifs	credentials=/root/smb-auth	0	0
   ```

   > 시스템 재부팅 시 마운트설정이 연결되어있기위함(218p)

##### 다중 사용자 마운트

##### 자동마운트

> 정리할부분 