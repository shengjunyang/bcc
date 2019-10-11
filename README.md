centos-7.6.64.1810 install bcc

yum install -y epel-release
yum -y update
yum groupinstall -y "Development tools"
yum install -y elfutils-libelf-devel iperf cmake3


mkdir build
cd build
如果bison版本低于3安装
curl -OL https://ftp.gnu.org/gnu/bison/bison-3.0.tar.xz
tar -xf bison-3.0.tar.xz
cd bison-3.0
./configure
make
make install


rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh https://www.elrepo.org/elrepo-release-7.0-4.el7.elrepo.noarch.rpm
yum -y --enablerepo=elrepo-kernel install kernel-ml kernel-ml-devel

cd ~/build
yum -y install yum-utils
yumdownloader --enablerepo=elrepo-kernel kernel-ml-headers
rpm2cpio kernel-ml-headers*.rpm | cpio -idmv

awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
0 : CentOS Linux (5.2.14-1.el7.elrepo.x86_64) 7 (Core)
1 : CentOS Linux (3.10.0-957.27.2.el7.x86_64) 7 (Core)
2 : CentOS Linux (3.10.0-514.el7.x86_64) 7 (Core)
3 : CentOS Linux (0-rescue-b7ab9add9a761df2e33b16b2038dbf9c) 7 (Core)
# 查看当前设置
grub2-editenv list
grub2-set-default 0
reboot 


cd ~/build
git clone https://github.com/iovisor/ply.git
cd ply
./autogen.sh
 export CFLAGS=-I${HOME}/build/usr/include
./configure
make
make install

cd ..
curl -LO http://releases.llvm.org/3.9.1/cfe-3.9.1.src.tar.xz
curl -LO http://releases.llvm.org/3.9.1/llvm-3.9.1.src.tar.xz
tar -xf cfe-3.9.1.src.tar.xz
tar -xf llvm-3.9.1.src.tar.xz
mkdir clang-build
mkdir llvm-build

cd llvm-build
cmake3 -G "Unix Makefiles" -DLLVM_TARGETS_TO_BUILD="BPF;X86" \
  -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr ../llvm-3.9.1.src
make
make install

cd ../clang-build
cmake3 -G "Unix Makefiles" -DLLVM_TARGETS_TO_BUILD="BPF;X86" \
  -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr ../cfe-3.9.1.src
make
make install

cd ..
git clone https://github.com/iovisor/bcc.git
mkdir bcc-build
cd bcc-build
cmake3 -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=/usr ../bcc
make
make install

export PATH=/usr/share/bcc/tools:$PATH

