# Simple Backup Maria DB/MySQL DB

## Overview

The maria DB backup uses the `cronjob` functionality of OpenShift to start a pod in regular intervals. The pod dumps the database to a persistent volume and exits.

This tool uses an existing RedHat Maria DB container and overrides its command. There is no need to build a container.

The project contains two templates:

* mariadb-backup-template.yaml


## How to deploy the maria DB backup pod

### Prequisits

* Log in using `oc login`
* Switch to the right project using `oc project <yourproject>`

### Create a pv for the backup

I would recommend to use the GUI for this part.

### Take a look at the parameters of the template

```bash
oc process --parameters -f mariadb-backup-template.yaml

```

**The following parameters are mandatory:**

* DATABASE_USER
* DATABASE_PASSWORD
* DATABASE_HOST
* DATABASE_PORT
* DATABASE_NAME
* DATABASE_BACKUP_VOLUME_CLAIM

### Create the cronjob

You can store the template in the project using and `oc process` afterwards

```bash
oc create -f mariadb-backup-template.yaml # project template
oc process mariadb-backup-template DATABASE_USER=<dbuser> DATABASE_PASSWORD=<dbpassword> ... | oc create -f -
```

```bash
oc create -f mariadb-backup-template.yaml -n openshift # global template
oc process -n openshift mariadb-backup-template DATABASE_USER=<dbuser> DATABASE_PASSWORD=<dbpassword> ... | oc create -f -
```

To check if the cronjob is present:

````bash
oc get cronjob
````

### Housekeeping

To disable the backup, you can simply suspend the cronjob:

`oc edit cronjob mariadb-backup`

Find the attribute `suspend` and set the value to `true`

To restore the backup you start a backup pod (e.g. in debug mode) connect to the pod and use:

````bash
oc rsh mariadb-backup-[xyz]-debug
mysql -u<db-user> -p<db-user-password> -h<host> < <path-to-backupfile> (the backupfile has to be unpacked)
````
