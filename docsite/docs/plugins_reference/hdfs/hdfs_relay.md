# hdfs_relay

## Synopsis

Allow definition of how all HDFS commands will be performed.

There should be only one entry of this type in the HADeploy definition file.

## Attributes

hdfs_relay is a map with the following attributes:

Name | req?	| Default |	Description
--- | --- | --- | ---
host|yes||The host on which all HDFS commands will be pushed for execution.
cache_folder|no|/var/cache/hadeploy/files|A folder on this host, which will be used by HADeploy as cache storage. Mainly, all files targeted to HDFS will be first pushed in this cache. And will remains in it, to optimize idempotence.<br>Changing this value is required if you intend to deploy under non-root account.
user|no|hdfs|The user account HADeploy will use to perform all HDFS related operation. Must have enough rights to do so.<br>Not to be defined when using Kerberos authentication.
hadoop_conf_dir|no|/etc/hadoop/conf|Where HADeploy will lookup Hadoop configuration file.
webhdfs_endpoint|no|`<hadoop_conf_dir>/hdfs-site.xml`|HADeploy will perform several actions through WebHDFS REST interface. You can specify corresponding endpoint, if it is not defined in the usual configuration way
principal|no||A Kerberos principal allowing all HDFS related operation to be performed. See below
keytab_path|no||A path to the associated keytab file on the relay host. See below

## Kerberos authentication

When `principal` and `keytab_path` variables are defined, Kerberos authentication will be activated for all HDFS folders, files and trees operations. This means a `kinit` will be issued with provided values before any HDFS access, and a `kdestroy` issued after. This has the following consequences:

* All HDFS operations will be performed on behalf of the user defined by the provided principal. The user parameter become irrelevant and providing it is an error.

* The `kinit` will be issued under the relay host ssh_user account (`root` by default). This means any previous ticket own by this user on this node will be lost. 

* And `ssh_user` must have read access on the provided keytab file.

Note also this keytab file must exists on the relay host. If it is not the case, one may copy it using file copy of HADeploy. This wills works as all file copy on the nodes are performed before any HDFS operation (See Execution order in Miscellaneous chapter) (LINK).

## Example

The simplest case:
```yaml
hdfs_relay:
  host: en1
```
Same, with default value sets:
```yaml
hdfs_relay:
  host: en1
  user: hdfs
  hadoop_conf_dir: "/etc/hadoop/conf"
  webhdfs_endpoint: "namenode.mycluster.myinfra.com:50070"
```
The simplest case with Kerberos activated:
```yaml
hdfs_relay:
  host: en1
  principal: hdfs-mycluster
  keytab_path: /etc/security/keytabs/hdfs.headless.keytab
```