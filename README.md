# install-openstack-icehouse-centos6

This is the steps that I did to isnatll Openstack Icehouse on Centos 6.7.

You may need libgmp >5 to run glance. So you can install it before start(Centos 6 has gmp 4.3).

You can Google  and download the source file of GMP. Then run:
  >./configure --prefix=/usr --libdir=/usr/lib64  (if you have 2 64bit machine)
  >
  >gmake
  >
  >gmake install
  
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

2.

If you met error at the stage of installing Keystone:

>Error: /Stage[main]/Keystone::Roles::Admin/Keystone_role[_member_]: Could not evaluate: Expected 2 columns for role row, found 0. Line

You can add two lines in: /usr/lib/python2.6/site-packages/packstack/modules/puppet.py

>re_ignore = re.compile(

>...

    'NetworkManager is not running|'
    #feng1
    'Expected|'
    #feng2
    'Keystone'
>)

Then, clear the mysql database root password and try again(Not sure what's the root cause, while the workaround works fine for me).

Done.

P.S.

If you failed at some point and openstack-glance-api started already, you may need to manully kill the processes related to it and start again. Each time you need to clean the mysql database's root password.

For example, you can stop all openstack related service:

>for script in /etc/init.d/openstack-*;

>do

 >   $script stop;

>done

>for script in /etc/init.d/neutron-*;

>do

>    $script stop;

>done


