---
title: "Exporting Pods Resource Allocations"
date: 2020-01-15T10:00:00+03:00
classes: wide
categories:
  - Openshift
tags:
  - openshift
---

You can export pod resource allocations using the script below.

```bash
#!/bin/bash 

project=$1
backupDir=c:/openshift_backup/
exportDir=$backupDir/$(date +'%d-%m-%Y-%H-%M-%S')
exportFile=$exportDir/resources.csv
ocpURL=https://api.os.yourdomain.com:6443
ocpUser=USER
ocpUserPass=PASS

function convert_to_core() {
	if [[ $1 = "<no value>" ]]
	then
		echo "0"
	elif [[ $1 == *m ]]
	then
		milicoreValue=${1//"m"/""}
		echo $(node -pe $milicoreValue/1000)
	else 
		echo $1
	fi
}

function convert_to_gb() {
	if [[ $1 = "<no value>" ]]
	then
		echo "0"
	elif [[ $1 == *Gi ]]
	then
		gbValue=${1//"Gi"/""}
		echo $gbValue
	elif [[ $1 == *Mi ]]
	then
		mbValue=${1//"Mi"/""}
		echo $(node -pe $mbValue/1000)
	else 
		echo $1
	fi
}

function process_project() {
	echo "Exporting $1 project's resources..."

	for i in $(oc get pods -n $1 --no-headers | awk '/Running/{ print $1 }')
	do
		nodeName=$(oc get pod $i -n $1 -o=go-template='{{.spec.nodeName}}')
		
		IFS=$'\n'
		
		for j in $(oc get pod $i -n $1 -o=go-template='{{range .spec.containers}}{{.name}}{{";"}}{{.resources.requests.cpu}}{{";"}}{{.resources.limits.cpu}}{{";"}}{{.resources.requests.memory}}{{";"}}{{.resources.limits.memory}}{{"\n"}}{{end}}')
		do
			IFS=';'
			
			read -ra podResources <<< "$j"
			
			IFS=$'\n'
			
			podCpuRequest=$(convert_to_core ${podResources[1]})
			podCpuLimit=$(convert_to_core ${podResources[2]})
			podMemoryRequest=$(convert_to_gb ${podResources[3]})
			podMemoryLimit=$(convert_to_gb ${podResources[4]})
			
			echo "$1;$nodeName;$i;${podResources[0]};${podResources[1]};$podCpuRequest;${podResources[2]};$podCpuLimit;${podResources[3]};$podMemoryRequest;${podResources[4]};$podMemoryLimit" >> $exportFile
		done
		
		IFS=$' \t\n'
	done
}

oc login -u $ocpUser -p $ocpPass --insecure-skip-tls-verify=true $ocpURL

mkdir -p $exportDir

echo "Namespace;Node Name;Pod Name;Container Name;CPU Request;CPU Request (Core);CPU Limit;CPU Limit (Core);Memory Request;Memory Request (GB);Memory Limit;Memory Limit (GB)" >> $exportFile

if [ -z $project ]
then
	echo "Project is empty. All projects will be exported."

	for i in $(oc get projects --no-headers | awk '{ print $1 }')
	do
		process_project $i
	done
else
	process_project $project
fi

exit 0
```
Update your export directory and cluster access information according to your environment.
{: .notice--info}