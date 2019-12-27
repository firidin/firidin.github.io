---
title: "Openshift Kaynak Tanımlarını Yedeklemek"
date: 2019-12-27T13:00:00+03:00
categories:
  - Openshift
tags:
  - openshift
---

Openshift kaynak tanımlarını yedeklemek için aşağıdaki scripti kullanabilirsiniz. 

```bash
#!/bin/bash 

project=$1
backupDir=c:/openshift_backup
exportDir=$backupDir/$(date +'%d-%m-%Y-%H-%M-%S')
ocpURL=https://api.os.yourdomain.com:6443
ocpUser=USER
ocpUserPass=PASS

mkdir -p $exportDir

function export_other() {
	for i in $(oc api-resources --no-headers | awk '/false/{ print $1 }')
	do
		echo "Exporting $i "
		oc get $i -o yaml  > $exportDir/$i.yaml
	done
}

function export_project() {
	echo "Exporting $1 project's resources..."

	projectDir=$exportDir/$1
	
	mkdir -p ${projectDir}
	
	for i in $(oc api-resources --no-headers | awk '/true/{ print $1 }')
	do
		echo "Exporting $i "
		oc get $i -n $1 -o yaml  > $projectDir/$i.yaml
	done
}

oc login -u $ocpUser -p $ocpUserPass --insecure-skip-tls-verify=true $ocpURL

if [ -z $project ]
then
	echo "Project is empty. All projects will be exported."

	for i in $(oc get projects --no-headers | awk '{ print $1 }')
	do
		export_project $i
	done
else
	export_project $project
fi

export_other

exit 0
```

Yedekleme alacağınız dizin ve cluster erişim bilgilerini ortamınıza göre güncellemeyi unutmayınız.
{: .notice--info}