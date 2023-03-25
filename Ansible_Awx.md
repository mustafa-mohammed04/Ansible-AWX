## AWX is an Open Source of Ansible Tower
## Install AWX Without Using Docker or Minikube >>  Locally

#### Ansible AWX Dashboard
![1](https://user-images.githubusercontent.com/128603198/227668544-50a56987-824a-4ace-8498-bbcf88d8d3b1.png)

#### Change User and make a password
![2](https://user-images.githubusercontent.com/128603198/227668571-fa1ba75f-6021-47a4-b2ef-0a312d9c407c.png)

#### Users of Dashboard
![3](https://user-images.githubusercontent.com/128603198/227668595-c5ed68e0-dcae-4676-ac52-f0bf7a21a26a.png)

#### Login with User (That you will create)
![4](https://user-images.githubusercontent.com/128603198/227668627-061a1c61-bd28-4bed-9152-d1d66c366ed2.png)

#### Verify User

![5](https://user-images.githubusercontent.com/128603198/227668635-0f0598e2-5b0e-46d8-883a-37d3e3b64280.png)

##### Commands that Used !
## register subscription
subscription-manager register --name <redhat_login_name> --password <redhat_password> --auto-attach

## initiate environment
hostnamectl set-hostname AWX
setenforce 0
systemctl disable --now firewalld
yum update 

## create all repos
yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -ivh http://mirror.centos.org/centos/7/extras/x86_64/Packages/centos-release-scl-rh-2-3.el7.centos.noarch.rpm
subscription-manager repos --enable rhel-7-server-optional-rpms --enable rhel-server-rhscl-7-rpms

## Repo of AWX

echo "[mrmeee-ansible-awx]
name=Copr repo for ansible-awx owned by mrmeee
baseurl=https://copr-be.cloud.fedoraproject.org/results/mrmeee/ansible-awx/epel-7-x86_64/
type=rpm-md
skip_if_unavailable=True
gpgcheck=1
gpgkey=https://copr-be.cloud.fedoraproject.org/results/mrmeee/ansible-awx/pubkey.gpg
repo_gpgcheck=0
enabled=1
enabled_metadata=1" > /etc/yum.repos.d/ansible-awx.repo

## install packages

yum install -y wget rabbitmq-server rh-postgresql10 memcached ansible nginx rh-python36-runtime rh-python36-python
yum -y install --disablerepo='*' --enablerepo='mrmeee-ansible-awx, rhel-7-server-rpms' -x *-debuginfo -x rh-python36-msgpack-0.6.2-1.x86_64 -x rh-python36-gitdb-4.0.2-1.noarch rh-python36*
yum install -y ansible-awx

## initialize services
cd /opt/awx/bin
scl enable rh-postgresql10 "postgresql-setup initdb"
wget -O /etc/nginx/nginx.conf https://raw.githubusercontent.com/MrMEEE/awx-build/master/nginx.conf

## enable and Start Services

systemctl enable --now memcached rabbitmq-server rh-postgresql10-postgresql.service nginx
scl enable rh-postgresql10 "su postgres -c \"createuser -S awx\""
scl enable rh-postgresql10 "su postgres -c \"createdb -O awx awx\""

## install missing python packages

scl enable rh-python36 bash
pip install --upgrade pip
pip install django==2.2.4
pip install channels==1.1.5

## initialize awx
sudo -u awx scl enable rh-python36 rh-postgresql10 "awx-manage migrate"
echo "from django.contrib.auth.models import User; User.objects.create_superuser('admin', 'root@localhost', 'password')" | sudo -u awx scl enable rh-python36 rh-postgresql10 "awx-manage shell"
sudo -u awx scl enable rh-python36 rh-postgresql10 "awx-manage create_preload_data"
sudo -u awx scl enable rh-python36 rh-postgresql10 "awx-manage provision_instance --hostname=$(hostname)"
sudo -u awx scl enable rh-python36 rh-postgresql10 "awx-manage register_queue --queuename=tower --hostnames=$(hostname)"
systemctl enable --now awx-cbreceiver awx-dispatcher awx-channels-worker awx-daphne awx-web awx

## To Verify 
firefox & 
your ip :8052 >>  192.168.80.129:8052
username: admin
password: password
