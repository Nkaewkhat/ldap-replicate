# OpenLDAP (Replication) Installation

## _การติดตั้ง OpenLDAP แบบมี Replication (provider - consumer)_

&nbsp;

นี่เป็นคู่มือประกอบการติดตั้ง OpenLDAP server แบบ Standard Replica (provider - consumer) หากต้องการวิธีการ setup แบบ single สามารถข้ามขั้นตอนการติดตั้ง consumer ไปได้เลย

แต่หากต้องการติดตั้งแบบ delta หรือต้องการข้อมูลอื่นๆ ให้เข้าไปดูรายละเอียดที่

- [OpenLDAP Replication](https://ubuntu.com/server/docs/service-ldap-replication)
- [OpenLDAP Administrator’s Guide](https://openldap.org/doc/admin24/guide.html#LDAP%20Sync%20Replication)
- [RFC 4533](http://www.rfc-editor.org/rfc/rfc4533.txt)

&nbsp;

## การเตรียมเครื่องที่จะใช้งาน

### Spec ของเครื่อง

เราสามารถเลือกเครื่องที่มีขนาดต่ำสุดเพื่อที่จะติดตั้ง Ubuntu Server 20.04 LTS (หรือ Version อื่นๆ ได้) โดยในคู่มือนี้เราจะเลือกเครื่องที่มี

- 1 Core vCPU
- 1 GB Memory
- Boot storage ขนาด 50GB
- สามารถใช้งาน Internet ได้ (ไม่ต้องการ Public IP หากใช้งานใน Private Network)

&nbsp;

## การติดตั้ง

### Update Package และ Kernel ต่างๆ ของทุก Server ให้ทันสมัย

> กระบวนการทั้งหมดนี้ทำใน root

```sh
apt update
apt upgrade -y
```

และเมื่อ Update แล้วให้ reboot และ remove package ที่ไม่ใช้งาน

```sh
reboot
```

```sh
apt autoremove
```

### ตั้งค่า hostname ของทั้ง 2 Server

ในคู่มือนี้เรากำหนดให้ Server ทั้ง 2 เครื่องอยู่คนละ subnet มีรายละเอียดดังนี้

| Seq. | IP Address | Server Name |
| ---- | ---------- | ----------- |
|    1 | 172.26.0.131 | ldap-provider.cryptonomics.co.th |
|    2 | 172.26.1.132 | ldap-consumer.cryptonomics.co.th |

### ในเครื่อง 172.26.0.131

```sh
#ตั้งค่า hostname ใน /etc/hosts , * สามารถใช้งาน Text Editor * ตัวไหนก็ได้ 
nano /etc/hosts

#ใส่ "#" (โดยไม่มี "")หน้า record ที่ขึ้นต้นด้วย 127.0.0.1
#แล้วเพิ่มข้อมูลที่ record 2 แถวบนสุดดังนี้
127.0.0.1 localhost ldap-provider ldap-provider.cryptonomics.co.th
172.26.1.132 ldap-consumer ldap-consumer.cryptonomics.co.th

#save แล้วออกจาก nano
```

เมื่อเสร็จ File `/etc/hosts` ในเครื่อง `172.26.0.131` จะดูคล้ายแบบนี้

```sh
#127.0.0.1 localhost
127.0.0.1 localhost ldap-provider ldap-provider.cryptonomics.co.th
172.26.1.132 ldap-consumer ldap-consumer.cryptonomics.co.th
# The following lines are desirable for IPv6 capable hosts
::1 ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts
```

run คำสั่ง hostnamectl เพื่อ update hostname และตรวจสอบ static-hostname

```sh
#update hostname
hostnamectl set-hostname ldap-provider.cryptonomics.co.th

#ตรวจสอบ hostname
hostnamectl
#output จะคล้ายแบบนี้
   Static hostname: ldap-provider.cryptonomics.co.th
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 987f8bc2cf2c1f46a16b6470618a23f8
           Boot ID: 89a81afdce464f4bb7daf4e341dffbcf
    Virtualization: kvm
  Operating System: Ubuntu 20.04.3 LTS
            Kernel: Linux 5.4.0-90-generic
      Architecture: x86-64
```

### ในเครื่อง 172.26.1.132

```sh
#ตั้งค่า hostname ใน /etc/hosts , * สามารถใช้งาน Text Editor * ตัวไหนก็ได้ 
nano /etc/hosts

#ใส่ "#" (โดยไม่มี "")หน้า record ที่ขึ้นต้นด้วย 127.0.0.1
#แล้วเพิ่มข้อมูลที่ record 2 แถวบนสุดดังนี้
127.0.0.1 localhost ldap-consumer ldap-consumer.cryptonomics.co.th
172.26.0.131 ldap-provider ldap-provider.cryptonomics.co.th

#save แล้วออกจาก nano
```

เมื่อเสร็จ File `/etc/hosts` ในเครื่อง `172.26.1.132` จะดูคล้ายแบบนี้

```sh
#127.0.0.1 localhost
127.0.0.1 localhost ldap-consumer ldap-consumer.cryptonomics.co.th
172.26.0.131 ldap-provider ldap-provider.cryptonomics.co.th
# The following lines are desirable for IPv6 capable hosts
::1 ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts
```

run คำสั่ง hostnamectl เพื่อ update hostname และตรวจสอบ static-hostname

```sh
#update hostname
hostnamectl set-hostname ldap-consumer.cryptonomics.co.th

#ตรวจสอบ hostname
hostnamectl
#output จะคล้ายแบบนี้
   Static hostname: ldap-consumer.cryptonomics.co.th
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 987f8bc2cf2c1f46a16b6470618a23f8
           Boot ID: 89a81afdce464f4bb7daf4e341dffbcf
    Virtualization: kvm
  Operating System: Ubuntu 20.04.3 LTS
            Kernel: Linux 5.4.0-90-generic
      Architecture: x86-64
```

&nbsp;

## ออก Certificate

### สร้าง Private Key, CSR แล้วนำมาขอออก Certificate เพื่อนำมาใช้ กับ STARTTLS

> OpenLDAP replicate บังคับใช้งาน TLS ในการเชื่อมต่อระหว่าง Provider และ Consumer

### ในเครื่อง ldap-provider

```sh
#generate new private key
openssl ecparam -genkey -name secp384r1 > resources/certificate/ldap-provider.cryptonomics.co.th.key

#generate new csr, change `emailAddress=email@cryptonomics.co.th` to your legit email Address
openssl req -new -subj "/C=TH/ST=./L=Bangkok/O=GMO-Z.com Cryptonomics (Thailand) Co., Ltd./OU=IT/CN=ldap-provider.cryptonomics.co.th/emailAddress=email@cryptonomics.co.th" -addext "subjectAltName = DNS:ldap-provider.cryptonomics.co.th" -key resources/certificate/ldap-provider.cryptonomics.co.th.key -out resources/certificate/ldap-provider.cryptonomics.co.th.csr

#show csr
cat resources/certificate/ldap-provider.cryptonomics.co.th.csr

-----BEGIN CERTIFICATE REQUEST-----
#
#    REQUEST CONTENT in Base64
#
-----END CERTIFICATE REQUEST-----
```

Copy ข้อมูลของไฟล์ CSR ตั้งแต่ `-----BEGIN CERTIFICATE REQUEST-----` ถึง `-----END CERTIFICATE REQUEST-----` แล้วส่งไปเปิด Ticket เพื่อออก Certificate

Save certificate ที่ถูก Sign ไว้ใน `resources/certificate/ldap-provider.cryptonomics.co.th.crt`

### ในเครื่อง ldap-consumer

```sh
#generate new private key
openssl ecparam -genkey -name secp384r1 > resources/certificate/ldap-consumer.cryptonomics.co.th.key

#generate new csr, change `emailAddress=email@cryptonomics.co.th` to your legit email Address
openssl req -new -subj "/C=TH/ST=./L=Bangkok/O=GMO-Z.com Cryptonomics (Thailand) Co., Ltd./OU=IT/CN=ldap-consumer.cryptonomics.co.th/emailAddress=email@cryptonomics.co.th" -addext "subjectAltName = DNS:ldap-consumer.cryptonomics.co.th" -key resources/certificate/ldap-consumer.cryptonomics.co.th.key -out resources/certificate/ldap-consumer.cryptonomics.co.th.csr

#show csr
cat resources/certificate/ldap-consumer.cryptonomics.co.th.csr

-----BEGIN CERTIFICATE REQUEST-----
#
#    REQUEST CONTENT in Base64
#
-----END CERTIFICATE REQUEST-----
```

Copy ข้อมูลของไฟล์ CSR ตั้งแต่ `-----BEGIN CERTIFICATE REQUEST-----` ถึง `-----END CERTIFICATE REQUEST-----` แล้วส่งไปเปิด Ticket เพื่อออก Certificate

Save certificate ที่ถูก Sign ไว้ใน `resources/certificate/ldap-consumer.cryptonomics.co.th.crt`

&nbsp;

## ติดตั้ง OpenLDAP (slapd) ทั้ง 2 Server

### ติดตั้ง OpenLDAP (slapd) และ Library ที่จำเป็น

```sh
#echo command ใส่ debconf-set-selections เพื่อทำ silent install
echo 'slapd slapd/root_password password 123456' | debconf-set-selections && \
echo 'slapd slapd/root_password_again password 123456' | debconf-set-selections && \
echo "slapd slapd/internal/adminpw password 123456" |debconf-set-selections && \
echo "slapd slapd/internal/generated_adminpw password 123456" |debconf-set-selections && \
echo "slapd slapd/password2 password 123456" |debconf-set-selections && \
echo "slapd slapd/password1 password 123456" |debconf-set-selections && \
echo "slapd slapd/domain string example.com" |debconf-set-selections && \
echo "slapd shared/organization string example" |debconf-set-selections && \
echo "slapd slapd/backend string HDB" |debconf-set-selections && \
echo "slapd slapd/purge_database boolean true" |debconf-set-selections && \
echo "slapd slapd/move_old_database boolean true" |debconf-set-selections && \
echo "slapd slapd/allow_ldap_v2 boolean false" |debconf-set-selections && \
echo "slapd slapd/no_configuration boolean false" |debconf-set-selections && \

DEBIAN_FRONTEND=noninteractive apt install -y slapd ldap-utils ssl-cert ca-certificates python

#ติดตั้ง tzdata เพื่อปรับ Timezone ให้ถูกต้อง
DEBIAN_FRONTEND=noninteractive apt install -y tzdata && echo "Asia/Bangkok" > /etc/localtime && dpkg-reconfigure -f noninteractive tzdata

service slapd stop
```

### จัดการ Certificate

### ติดตั้ง Trusted Root Certificate

```sh
#เพิ่ม resources/certificate/ca-cryptonomics.co.th.cer ลงใน Trusted Root
cp resources/certificate/ca-cryptonomics.co.th.cer /usr/local/share/ca-certificates/cacert.crt
chmod 640 /usr/local/share/ca-certificates/cacert.crt
update-ca-certificates
```

### Copy CA, KEY, Certificate ใส่ folder `/etc/ldap/sasl2`

> Certificate ของ `provider` ใช้ `ldap-provider`, `consumer` ใช้ `ldap-consumer`

```sh
cp resources/certificate/ca-cryptonomics.co.th.cer /etc/ldap/sasl2/ca-cryptonomics.co.th.cer
cp resources/certificate/ldap-(provider|consumer).cryptonomics.co.th.key /etc/ldap/sasl2/ldap-(provider|consumer).cryptonomics.co.th.key
cp resources/certificate/ldap-(provider|consumer).cryptonomics.co.th.crt /etc/ldap/sasl2/ldap-(provider|consumer).cryptonomics.co.th.crt

chmod -R 640 /etc/ldap/sasl2/* 
chown -R openldap:openldap /etc/ldap/sasl2/* 
```

### เตรียมไฟล์ data.ldif

>จากไฟล์ `resources/ldif/example.data.ldif` จะมีข้อมูลเบื้องต้นเพื่อใช้ในการจัดการ LDAP Database </br>
>โดยจะมี account ถูกสร้างมาแล้ว 2 account คือ
>
>- `cn=admin,dc=cryptonomics,dc=co,dc=th`
>- `cn=readonly,ou=people,dc=cryptonomics,dc=co,dc=th`

เราจะทำการคัดลอกไฟล์นี้

```sh
cp resources/ldif/example.data.ldif resources/ldif/data.ldif
```

และตั้ง Password ให้ 2 account นี้

```sh
# for `cn=admin,dc=cryptonomics,dc=co,dc=th` change <supersecurepasswordforadmin> to anythings you need, conform with password policy
slappasswd -h {SSHA} -s supersecurepasswordforadmin

slappasswd -h {SSHA} -s supersecurepasswordforreadonly

# it will output hashed password or similar 
# "{SSHA}n5zobTGg3Jzgo9wiFiJ1fp/XQ3bhsJ8x" (the output will change each time, even the input password is the same)
```

นำค่า hashed ที่ได้ไปใส่ในไฟล์ `resources/ldif/data.ldif` ที่ค่า `userPassword` ตามลำดับ

```ldif
#file: resources/ldif/data.ldif
-
dn: cn=admin,dc=cryptonomics,dc=co,dc=th
objectClass: simpleSecurityObject
objectClass: organizationalRole
cn: admin
description: LDAP administrator
userPassword: {SSHA}A9knXvgn+9qhmchqwtbf6kE6upgZ1yvO
structuralObjectClass: organizationalRole
entryUUID: fc3d07a4-9045-103a-95bf-a559b9a67a35
creatorsName: cn=admin,dc=cryptonomics,dc=co,dc=th
createTimestamp: 20200921110438Z
entryCSN: 20200921110438.321925Z#000000#000#000000
modifiersName: cn=admin,dc=cryptonomics,dc=co,dc=th
modifyTimestamp: 20200921110438Z
-
...
dn: cn=readonly,ou=people,dc=cryptonomics,dc=co,dc=th
objectClass: organizationalRole
objectClass: simpleSecurityObject
cn: readonly
userPassword: {SSHA}5Cd/ca6fmiex4PYp5617CR7cFAVD7NcB
description: Bind DN user for LDAP Operations
structuralObjectClass: organizationalRole
entryUUID: 2a3157b6-90da-103a-9cdf-7b83decb964a
creatorsName: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
createTimestamp: 20200922044520Z
entryCSN: 20200922044520.936504Z#000000#000#000000
modifiersName: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
modifyTimestamp: 20200922044520Z
-
```

### เตรียมไฟล์ config.ldif

>จากไฟล์ `resources/ldif/base.config.ldif` จะมี configuration ที่กำหนดไว้เป็นมาตฐานที่ต้องใช้แล้ว </br>
>เราจะทำการคัดลอกเป็นไฟล์ config.ldif และแก้ไขบางส่วนในเพื่อให้นำไปใช้ให้ Server ทั้ง 2 เครื่อง

```ldif
# file: resources/ldif/config.ldif

# เปลี่ยน path/file ของ certificate ให้ตรงกับ hostname
# provider ใช้ ldap-provider, consumer ใช้ ldap-consumer
-
olcTLSCACertificateFile: /etc/ldap/sasl2/ca-cryptonomics.cer
olcTLSCertificateFile: /etc/ldap/sasl2/ldap-(provider|consumer).cryptonomics.co.th.crt
olcTLSCertificateKeyFile: /etc/ldap/sasl2/ldap-(provider|consumer).cryptonomics.co.th.key
-

# นำค่า hashed ของ dn: cn=admin,dc=cryptonomics,dc=co,dc=th ไปใส่ใน olcRootPW
-
olcRootPW: {SSHA}A9knXvgn+9qhmchqwtbf6kE6upgZ1yvO
-
```

### เปลี่ยน config, data ของ openldap ด้วยไฟล์ที่เราแก้ไข

เราจะทำการใส่ไฟล์ config, data ใหม่ลงไป

```sh
# make sure slapd is stoped
service slapd stop

# เอา `/etc/ldap/slapd.d` `/var/lib/ldap` ปัจจุบันออก โดยคำสั่ง mv
mv /etc/ldap/slapd.d /etc/ldap/slapd.d.`date '+%Y-%m-%d'`
mv /var/lib/ldap /var/lib/ldap.`date '+%Y-%m-%d'`

# สร้าง folder `/etc/ldap/slapd.d` `/var/lib/ldap` ใหม่
mkdir /etc/ldap/slapd.d
mkdir -p /var/lib/ldap

# restore config, data
slapadd -n 0 -F /etc/ldap/slapd.d/ -l resources/ldif/config.ldif
slapadd -n 1 -F /etc/ldap/slapd.d/ -l resources/ldif/data.ldif

# update folder permision
chown -R openldap:openldap /etc/ldap/slapd.d/
chown -R openldap:openldap /var/lib/ldap

# start service back
service slapd start
```

ทดสอบ admin credential

```sh
# ldap-provider ใช้ ldap://ldap-provider.cryptonomics.co.th
# ldap-consumer ใช้ ldap://ldap-consumer.cryptonomics.co.th
ldapwhoami -H ldap://ldap-(provider|consumer).cryptonomics.co.th -x -ZZ -D "cn=admin,dc=cryptonomics,dc=co,dc=th" -x -W
# จะมี LDAP prompt รอ password ของเราอยู่
Enter LDAP Password:
```

> กรณีที่ password ถูก จะส่งค่า `dn:cn=admin,dc=cryptonomics,dc=co,dc=th` กลับมา

หรือ

> กรณีที่ password ผิด จะส่งค่า `ldap_bind: Invalid credentials (49)` กลับมา
> ซึ่งการแก้ไขอาจจะต้องย้อนกระบวนการการสสร้างไฟล์ data, config ใหม่อีกครั้ง </br> หรืออาจจะสร้างไฟล์เพื่อ modify `olcRootPW` ของ `olcDatabase={1}mdb,cn=config` </br> และ `userPassword` ของ user `dn: cn=admin,dc=cryptonomics,dc=co,dc=th` ให้ตรงกันก็ได้ _โดยใช้คำสั่ง ldapmodify_

&nbsp;

### ตั้งค่า Log ของ slapd

> ตั้งค่า log เป็น local4

```sh
# set local4
echo "local4.* /var/log/slapd.log" >> /etc/rsyslog.d/51-slapd.conf

# restart rsyslog slapd
systemctl restart rsyslog slapd
```

> ตั้งค่า logrotate โดยการสร้างไฟล์ `/etc/logrotate.d/slapd` แล้วใส่ config ในไฟล์ดังนี้

```sh
/var/log/slapd.log
{ 
  rotate 7
  daily
  missingok
  notifempty
  delaycompress
  compress
  postrotate
    /usr/lib/rsyslog/rsyslog-rotate
  endscript
}
```

```sh
# restart logrotate
systemctl restart logrotate
```

```sh
# ตั้งค่า slapd service ให้เป็น enable ผ่าน systemd
systemctl enable slapd.service
```

&nbsp;

## Provider Configuration (ทำที่เครื่อง ldap-provider)

การทำ replicate จะต้องสร้าง account เอาไว้บน Provider เพื่อให้ Consumer สามารถ Login เข้าไปอ่านข้อมูลได้

> เราจะใช้ไฟล์ `resources/ldif/replicator.ldif` เป็นข้อมูลเพื่อสร้าง account บน Provider

### ใช้ ldapadd เพื่อ save ลง LDAP Database แล้วใช้ ldappasswd เพื่อ update password

```sh
#ใช้ ldapadd เพื่อ save ลง LDAP Database
ldapadd -Q -Y EXTERNAL -H ldapi:/// -f resources/ldif/replicator.ldif

#ใช้ ldappasswd เพื่อ update password
ldappasswd -S -Y EXTERNAL -H ldapi:/// cn=replicator,dc=cryptonomics,dc=co,dc=th

# Note password เก็บไว้ เพราะเราต้องใช้เพื่อ setup sync config ที่เครื่อง consumer
```

### ใช้ ldapmodify เพื่อ setup ACL Limit

```sh
ldapmodify -Q -Y EXTERNAL -H ldapi:/// -f resources/ldif/replicator-acl-limits.ldif
```

### Setup `syncprov` Overlay ให้เครื่อง Provider

> เราจะใช้ไฟล์ `resources/ldif/provider_simple_sync.ldif` เพื่อสร้าง overlay

```sh
# สร้าง Overlay
ldapadd -Q -Y EXTERNAL -H ldapi:/// -f resources/ldif/provider_simple_sync.ldif

# ตรวจสอบ contextCSN
ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -s base -b dc=cryptonomics,dc=co,dc=th contextCSN
```

---

&nbsp;

## Consumer Configuration (ทำที่เครื่อง ldap-consumer)

### ตั้งค่าการ Sync

&nbsp;

> เราจะใช้ไฟล์ `resources/ldif/base.consumer_sync.ldif` เป็นต้นแบบเพื่อสร้าง `resources/ldif/consumer_sync.ldif`

เราจะเปลี่ยนรายละเอียดในไฟล์ `resources/ldif/consumer_sync.ldif` ให้ตรงกับการตั้งค่าบน Provider ดังนี้

```ldif
dn: cn=module{0},cn=config
changetype: modify
add: olcModuleLoad
olcModuleLoad: syncprov

dn: olcDatabase={1}mdb,cn=config
changetype: modify
add: olcDbIndex
olcDbIndex: entryUUID eq
-
add: olcSyncrepl
olcSyncrepl: rid=0
  # provider=ldap://ldap01.example.com
  provider=ldap://ldap-provider.cryptonomics.co.th
  bindmethod=simple
  # binddn="cn=replicator,dc=example,dc=com" credentials=<secret>
  binddn="cn=replicator,dc=cryptonomics,dc=co,dc=th" credentials=<secret>
  # searchbase="dc=example,dc=com"
  searchbase="dc=cryptonomics,dc=co,dc=th"
  logbase="cn=accesslog"
  logfilter="(&(objectClass=auditWriteObject)(reqResult=0))"
  schemachecking=on
  type=refreshAndPersist retry="60 +"
  syncdata=accesslog
  starttls=critical tls_reqcert=demand
-
add: olcUpdateRef
# olcUpdateRef: ldap://ldap01.example.com
olcUpdateRef: ldap://ldap-provider.cryptonomics.co.th
```

> เปลี่ยน `<secret>` ใน `credentials=<secret>` ให้ตรงกับ password ที่เราสร้างให้ `cn=replicator,dc=cryptonomics,dc=co,dc=th`

แล้วใช้ `ldapadd` เพื่อ update config ของ `ldap-consumer`

```sh
ldapadd -Q -Y EXTERNAL -H ldapi:/// -f resources/ldif/consumer_sync.ldif
```

### ตรวจสอบการ Sync

ตรวจสอบ `contextCSN` เมื่อมีการ `add`, `modify` ทั้ง 2 เครื่อง

```sh
# ตรวจสอบ contextCSN 
ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -s base -b dc=cryptonomics,dc=co,dc=th contextCSN

# LDAP จะแสดงข้อมูลของ contextCSN 
dn: dc=cryptonomics,dc=co,dc=th
contextCSN: 20211115083250.179160Z#000000#000#000000
```

> `contextCSN` ระหว่าง Provider และ Consumer จะตรงกันทุกครั้งที่มีการเปลี่ยนแปลง Data ใน Database

หาก `contextCSN` ไม่ตรงกับ แปลว่า ไม่สามารถ Sync ข้อมูลกันได้ ให้ตรวจสอบไฟล์ log ใน `/var/log/syslog` </br>
ตรวจสอบให้แน่ใจว่า Firewall ของเครื่อง Provider เปิดให้เชื่อมต่อที่ Port 389 ได้

&nbsp;

## Appendix

&nbsp;

เข้าไปดูรายละเอียดเพิ่มเติมที่

- [OpenLDAP Replication](https://ubuntu.com/server/docs/service-ldap-replication)
