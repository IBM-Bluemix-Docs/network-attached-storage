---

copyright:
  years: 2014, 2018
lastupdated: "2018-07-31"

---
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}

# Mounting {{site.data.keyword.cos_short}} with s3fuse

You might want to use {{site.data.keyword.cos_full}} for various reasons. One of them is the ability to store or backup data in a different data center, other than the one where your server is located. For more information about {{site.data.keyword.cos_short}}, see [About {{site.data.keyword.cos_full_notm}}](https://console.bluemix.net/docs/services/cloud-object-storage/about-cos.html)

You can use the `s3fs-fuse`utility in Linux and Mac OS X to mount an {{site.data.keyword.cos_short}} bucket through FUSE. `s3fs` preserves the original object format for files. For more information about `s3fs-fuse`, see the [S3sf Wiki](https://github.com/s3fs-fuse/s3fs-fuse/wiki/FAQ).

1. Update the OS.
   - RHEL/CentOS
     ```
     yum -y update
     ```
     {: pre}
     
   - Debian/Ubuntu
     ```
     apt-get upgrade
     ```
     {: pre}
   
2. Install dependencies.
   - RHEL/CentOS
     ```
     yum -y install automake fuse fuse-devel gcc-c++ git libcurl-devel libxml2-devel make openssl-devel
     ```
     {: pre}
     
   - Debian/Ubuntu
     ```
     apt-get install automake autotools-dev fuse g++ git libcurl4-openssl-dev libfuse-dev libssl-dev libxml2-dev make pkg-config
     ```
     {: pre}
     
3. Install s3fs-fuse.
   - RHEL/CentOS
     ```
     git clone https://github.com/s3fs-fuse/s3fs-fuse.git
     ```
     {: pre}
      
   - Debian/Ubuntu 
     ```
     apt-get install s3fs
     ```
     {: pre}
   
4. Get into the new directory.
   ```
   cd s3fs-fuse/
   ```
   {: pre}
   
5. Start building the source.
   ```
   ./autogen.sh
   ```
   {: pre}
   
   ```
   ./configure
   ```
   {: pre}
   
   ```
   make
   ```
   {: pre}
   
   ```
   make all-recursive
   ```
   {: pre}
   
   ```
   make install
   ```
   {: pre}
   
6. Place the values of your `Access Key ID` and `Secret Access Key` into a file called `~/.passwd-s3fs`, or `/etc/passwd-s3fs`. The two entries must be separated by a colon.     
   - You can create the file with the following command.
     ```
     echo Access_Key_ID:Secret_Access_Key > ~/.passwd-s3fs
     ```
     {: pre}
   - You can also use `vi(m)` or `nano` to create the file.<br>
   
     **Note** - You can get the `Access Key ID` and `Secret Access Key` from the {{site.data.keyword.BluSoftlayer_full}} console.
     1. Click your {{site.data.keyword.cos_full_notm}} instance.
     2. In the navigation panel, click **Service Credentials**.
     3. Click **New credential** to generate credential information.
     {:tip}
         
7. For security reasons, set owner-only permissions for the password file.
   ```
   chmod 600 ~/.passwd-s3fs
   ```
   {: pre}
   
8. Create the directory that you want to mount the {{site.data.keyword.cos_short}} bucket to.
   ```
   mkdir /mnt/s3test
   ```
   {: pre}

9. Start s3fs fuse. Use the endpoint that corresponds to the location of the bucket that is used. The following example uses the US Cross Region service, public endpoint, and the bucket ‘s3fss3fusetest’. The bucket must already exist in your {{site.data.keyword.cos_short}}.<br>
   **Note** - You can find endpoint and bucket information in the [{{site.data.keyword.slportal}}](https://control.softlayer.com/){:new_window}. For more information, see [Select regions and endpoints](https://console.bluemix.net/docs/services/cloud-object-storage/basics/endpoints.html){:new_window} and [Creating buckets to store your data](https://console.bluemix.net/docs/services/cloud-object-storage/getting-started.html#create-buckets){:new_window}.
   {:tip}

   Syntax
   ```
   s3fs examplebucket /mnt/s3test -o passwd_file=~/.passwd-s3fs -o url=endpint -o use_path_request_style -o dbglevel=info
   ```
   {: pre}
   
   Example
   ```
   [root@s3fuse s3fs-fuse]# s3fs s3fusetest /mnt/s3test/ -o passwd_file=~/.passwd-s3fuse -o url=http://s3-api.us-geo.objectstorage.softlayer.net/ -o use_path_request_style -o dbglevel=info
   ```
   {: pre}
   
   **Note** - Since `-o dbglevel=info` was used, you can check `/var/log/messages` for s3fs entries. These logs help with debugging when something isn't working correctly.
   
10. Use `df -h` to verify that the {{site.data.keyword.cos_short}} is mounted.

11. Use `ls` to confirm that the information saves in the {{site.data.keyword.cos_short}}.
   ```
   [root@s3fuse s3fs-fuse]# ls /mnt/s3test/
   2018-07-08  2018-07-10  2018-07-12  2018-07-14  2018-07-15  holiday event.pptx  test  test2.file  test.file
   ```
   {: pre}
   
12. If you want the {{site.data.keyword.cos_short}} bucket to mount upon boot, you can add it to the `/etc/fstab`. Make sure that the credentials are in `/etc/passwd-s3fs`. The following line goes into the `/etc/fstab` file.
    ```
    s3fs#examplebucket /mnt/s3test fuse _netdev,allow_other,use_path_request_style,url=https://s3-api.us-geo.objecstorage.softlayer.net 0 0
    ```
    {: pre}