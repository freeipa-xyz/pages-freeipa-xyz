---
layout: chapter
chapter_id: 1020
chapter_title: "Certificate profile"
date:   2024-10-26
theme:  8021x-workstations
permalink: /a/a1cff461-5579-4cfd-9c41-d60911bae228
---

## Create special certificate profile for computer authentication 

Export FreeIPA default host certificate profile to file and create a copy for editing:

```shell
ipa certprofile-show caIPAserviceCert --out caIPAserviceCert.txt
cp caIPAserviceCert.txt ca802_1xCert.txt 
```

Replase some namings: 
```
sed -i 's/.serverCertSet./.set1./g' ca802_1xCert.txt
sed -i 's/policyset.list=serverCertSet/policyset.list=set1/g' ca802_1xCert.txt
sed -i 's/caIPAserviceCert/ca802_1xCert/g' ca802_1xCert.txt
sed -i 's/server certificates/802.1x certificates/g' ca802_1xCert.txt
sed -i 's/Server Certificate Enrollment/802.1x Certificate Enrollment/g' ca802_1xCert.txt
```

#### Certificate validity

Open `ca802_1xCert.txt` file with text editor. 
Replace `policyset.set1.2` group with
```text
policyset.set1.2.constraint.class_id=validityConstraintImpl
policyset.set1.2.constraint.name=Validity Constraint
policyset.set1.2.constraint.params.rangeUnit=day
policyset.set1.2.constraint.params.range=60
policyset.set1.2.constraint.params.notBeforeGracePeriod=3600
policyset.set1.2.constraint.params.notBeforeCheck=true
policyset.set1.2.constraint.params.notAfterCheck=true
policyset.set1.2.default.class_id=validityDefaultImpl
policyset.set1.2.default.name=Validity Default
policyset.set1.2.default.params.range=60
policyset.set1.2.default.params.startTime=0
```

#### Certificate key options

Find `policyset.set1.3.constraint.params.keyParameters=` and remove value `1024` from the list:

```text
policyset.set1.key.constraint.params.keyParameters=2048,3072,4096,8192
```

#### Certificate Extended Key Usages:

Find `policyset.set1.7.default.params.exKeyUsageOIDs=` and replace value with  `1.3.6.1.5.5.7.3.2,1.3.6.1.5.5.7.3.14`. which stands for `ClientAuth, EAP-over-LAN`.
* `1.3.6.1.5.5.7.3.14` is `EAP over LAN` or `id-kp.14`, which is a way for us to know if this certificate passed to server by client is expected to be used for 802.1x authentication and not just default certificate

#### Certificate signing algorithms

Find `policyset.set1.8.constraint.params.signingAlgsAllowed` and remove all Algs except SHA-2 algs:

```text
policyset.set1.8.constraint.params.signingAlgsAllowed=SHA256withRSA,SHA384withRSA,SHA512withRSA
```

#### AIA records
Comment out whole AIA section. This disables revocation checks using OSCP.: 
```
# policyset.set1.5.constraint.class_id=noConstraintImpl
# policyset.set1.5.constraint.name=No Constraint
# policyset.set1.5.default.class_id=authInfoAccessExtDefaultImpl
# policyset.set1.5.default.name=AIA Extension Default
# policyset.set1.5.default.params.authInfoAccessADEnable_0=true
# policyset.set1.5.default.params.authInfoAccessADLocationType_0=URIName
# policyset.set1.5.default.params.authInfoAccessADLocation_0=http://ipa-ca.od.freeipa.xyz/ca/ocsp
# policyset.set1.5.default.params.authInfoAccessADMethod_0=1.3.6.1.5.5.7.48.1
# policyset.set1.5.default.params.authInfoAccessCritical=false
# policyset.set1.5.default.params.authInfoAccessNumADs=1
```

and remove `5` from the list: 

```
policyset.set1.list=1,2,3,4,6,7,8,9,10,11,12
```

#### Save the file
Save the file and exit

### Import certificate profile to FreeIPA
Import back file we just created as a new certificate profile to FreeIPA:

#### Important note
If you want to revoke certificates created using this profile with normal 
certificate revocation mechanisms of FreeIPA, you should probably use `--store=true`. 

```shell
ipa certprofile-import ca802_1xCert \
  --file=ca802_1xCert.txt \
  --store=false \
  --desc="This certificate profile is for enrolling 802.1x certificates with IPA-RA agent authentication."
```

Add permission rule for CA ACL subsystem which fits all hosts
```shell
ipa caacl-add hosts__ca802_1xCert --hostcat='all'
```

Bind this ACL to our new certificate profile: 

```shell
ipa caacl-add-profile hosts__ca802_1xCert --certprofiles=ca802_1xCert
```
