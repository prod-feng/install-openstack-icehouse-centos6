# install-openstack-icehouse-centos6

The following are the steps that I did to isnatll Openstack Icehouse on Centos 6.7.

You may need libgmp >5 to run glance. So you can install it before start(Centos 6 has gmp 4.3, the real problem I had was I had some juno packages installed. I had to clean all of them).

You can Google  and download the source file of GMP. Then run:
  >./configure --prefix=/usr --libdir=/usr/lib64  (if you have 2 64bit machine)
  >
  >gmake
  >
  >gmake check 
  >
  >gmake install 
  >
  >libtool --finish /usr/lib64
  >
  >pip install --ignore-installed PyCrypto 

  
OK, let's start:

1. 
> yum install https://repos.fedorapeople.org/repos/openstack/EOL/openstack-icehouse/rdo-release-icehouse-4.noarch.rpm

The repository is here: https://repos.fedorapeople.org/repos/openstack/EOL/openstack-icehouse/epel-6/

2. 
> yum install openstack-packstack

3.

>packstack --allinone

Packstack by defaule uses ssh port 22. If you use different port, it can be a problem. The workaround can be easy:
In /etc/ssh/ssh_config, add one line after line of "Host *"

>Host *
 
 >    Port your-port-number

The file /usr/lib/python2.6/site-packages/packstack/installer/validators.py also needs to be updated with the new ssh port number:

>def validate_ssh(param, options=None):
>>    """

>>    Raises ParamValidationError if provided host does not listen

>>    on port 22??.

>>    """

>>    options = options or []

>>    try:

>>        touch_port(param.strip(), $put-your-ssh-port-num-here)

Then it will be fine.

Besides, if you run the installation as root, you may want to set "PermitRootLogin=yes" in /etc/ssh/sshd_config, and then restart sshd.

You'd better start from a clean OS. If you have mysql, qpid, etc., running on your computer, you might need to stop them when you run "Allinone" Icehouse.


Note:

1. If you have the database error during installation(for example, the packstack failed at some poit, and when you try to run the installation again), like:

>>ERROR : Error appeared during Puppet run: X.X.X.X_mariadb.pp
>>Error: mysqladmin -u root  password 'fa5a74a6e6dd4ba4' returned 1 instead of one of [0]

You may need to clear the mysql "root" password, and try again.

>/etc/init.d/mysqld stop

>mysqld_safe --skip-grant-tables &

>mysql -u root

>>use mysql;

>>update user set password=PASSWORD("") where User='root';

>>flush privileges;

>>quit

find the running mysqld processid:

>ps -ef|grep my

>>kill -9 [processid]

make sure no mysqld is running.

Then run " packstack --allinone" again.

Or you can run yum to erase mysql and mysql-libs, and delete all files in /var/lib/mysql(hope packstack can be smarter).

The other way is to run packstack using the answer file generated from last run, so you do not need to worry about mysql password stuffs:

>packstack  --answer-file=packstack-answers-??????.txt

Done.

P.S.

1. If you failed at some point and openstack-glance-api started already, you may need to manully kill the processes related to it and start again. Each time you need to clean the mysql database's root password.

For example, you can stop all openstack related service:

>for script in /etc/init.d/openstack-*;

>do

 >   $script stop;

>done

>for script in /etc/init.d/neutron-*;

>do

>    $script stop;

>done

Make sure all related services/processes stopped. You may have to "kill -9" manually some remaining processes.

2. Also, if you have an error on Swift:

>Error: Execution of '/usr/bin/swift-ring-builder /etc/swift/object.builder add r1z1-x.x.x.x:6000/swift_loopback 10.00' returned 2: Device 0 already uses x.x.x.x:6000/swift.

You can do:

>yum erase openstack-swift

> rm -r /et/swift

and then run packstack again.

3. If you can not start nova_compute service, you might need to ucommnet one line in /etc/libvirt/libvirtd.conf:

> #auth_unix_rw = "none" 

4. Libvirtd. You may need to dowload the recent version of libvirtd and compiled it from the source:

>./configure --prefix=/usr --libdir=/usr/lib64  --with-xen   --with-xen-inotify --with-yajl
>
>gmake
>
>sudo gmake install
>
>sudo ldconfig

5 openstack-dashboard 1.5 seems doe not work well with Neutron, erased it, and installed an older version:

> yum install openstack-dashboard-2014.1.3-1.el6.noarch


6. Failed to create image and run instance using dashboard.
  
   Use virt-mamager creates image, and then:

>glance image-create --name='cent image' --is-public=true  --container-format=bare --disk-format=qcow2 >/var/lib/libvirt/images/vm01.img


  Then run instance OK.

  
