#### Spec

- OS : amazon linux2
- RPM Packages : gcc openssl-devel lz4-devel lzo lzo-devel pam-devel automake

~~~bash
# Install required packages
yum -y install gcc openssl-devel lz4-devel lzo lzo-devel pam-devel automake git python-pip unix2dos

pip install markdown

# Download openvpn release file
wget https://swupdate.openvpn.org/community/releases/openvpn-2.5.0.tar.xz
xz -d openvpn-2.5.0.tar.xz
tar xvf openvpn-2.5.0.tar

cd openvpn-2.5.0
./configure
./make
./make install


timedatectl set-timezone Asia/Seoul

git clone https://github.com/OpenVPN/easy-rsa.git
cd easy-rsa
./build/build-dist.sh --version=3.2.1


./easyrsa import-req /tmp/path/to/import.req 13.124.233.171









~~~







repo.

https://openvpn.net/community-resources/how-to/#openvpn-quickstart

