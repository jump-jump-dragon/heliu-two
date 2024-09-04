## Ubuntu下配置 MFEM 环境

**MFEM: 模块化有限元方法库**

打开 Ubuntu 终端

新建工作目录
```bash
mkdir mfem-slepc
```

进入工作目录
```bash
cd mfem-slepc
```

用户的路径
```bash
\home\testuser1\mfem-slepc
```

传输五个文件，查看文件列表
```bash
ls
```

搭建编译环境（常用开发工具和C/C++, MPI开发环境）
```bash
sudo apt update
sudo apt install build-essential cmake git 
sudo apt install g++ make
sudo apt install unzip
sudo apt install mpich
```

### 编译 metis-5.1.0

```bash
tar -zxvf metis-5.1.0.tar.gz
mv metis-5.1.0 metis
cd metis
make BUILDDIR=lib config  # (默认使用gcc编译，若想指定MPI编译器，请添加cc=mpicc, 查看编译选项请直接在命令行输入make,若要编译多线程版本，指定openmp=-fopenmp)
make BUILDDIR=lib
cp lib/libmetis/libmetis.a lib
cd ..
```

### 编译 hypre-master

```bash
unzip hypre-master.zip
cd hypre-master/src
./configure --disable-fortran --with-openmp --with-MPI
make -jx # 其中x是“cpu的核数-1”，防止死机
cd ../..
ln -s hypre-master hypre
```

### 编译 petsc-3.21.4

```bash
tar -zxvf petsc-3.21.4.tar.gz
mv petsc-3.21.4 petsc  # 改文件名
cd petsc
# 打开浏览器，搜索框中输入：https://bitbucket.org/petsc/pkg-fblaslapack/getv3.4.2-p3.tar.gz，文件为 petsc-pkg-fblaslapack-e8a03f57d64c.tar.gz
# 在 home/testuser1/mfem-slepc 目录下新建目录 
mkdir blaslapack # 将 petsc-pkg-fblaslapack-e8a03f57d64c.tar.gz 复制到blaslapack文件夹下 
./configure --download-fblaslapack=/home/testuser1/mfem-slepc/blaslapack/petsc-pkg-fblaslapack-e8a03f57d64c.tar.gz
make PETSC_DIR=/home/testuser1/mfem-slepc/petsc PETSC_ARCH=arch-linux-c-debug all
make PETSC_DIR=/home/testuser1/mfem-slepc/petsc PETSC_ARCH=arch-linux-c-debug check
cd ..
```

### 编译 slepc-3.21.1

```bash
tar -zxvf slepc-3.21.1.tar.gz
mv slepc-3.21.1 slepc
cd slepc
export PETSC_DIR=/home/testuser1/mfem-slepc/petsc
export PETSC_ARCH=arch-linux-c-debug
./configure
make SLEPC_DIR=/home/testuser1/mfem-slepc/slepc PETSC_DIR=/home/testuser1/mfem-slepc/petsc PETSC_ARCH=arch-linux-c-debug
make SLEPC_DIR=/home/testuser1/mfem-slepc/slepc PETSC_DIR=/home/testuser1/mfem-slepc/petsc check
cd ..
```

### 编译 mfem-master 并运行 examples/petsc/ex11p 特征值问题

```bash
unzip mfem-master.zip
cd mfem-master
mv mfem-master mfem
make config MFEM_USE_SIMD=YES MFEM_USE_OPENMP=YES MFEM_USE_MPI=YES MFEM_USE_METIS_5=YES METIS_DIR=@MFEM_DIR@/../metis MFEM_USE_PETSC=YES PETSC_LIB="-L@MFEM_DIR@/../petsc/arch-linux-c-debug/lib -lpetsc" PETSC_OPT="-I@MFEM_DIR@/../petsc/include -I@MFEM_DIR@/../petsc/arch-linux-c-debug/include" MFEM_USE_SLEPC=YES SLEPC_LIB="-L@MFEM_DIR@../slepc/arch-linux-c-debug/lib -lslepc" SLEPC_OPT="-I@MFEM_DIR@/../slepc/include -I@MFEM_DIR@/../slepc/arch-linux-c-debug/include"

make -jx # 其中x是“cpu的核数-1”，防止死机

cd /home/testuser1/mfem-slepc/slepc/arch-linux-c-debug/lib

sudo cp libslepc.so.3.21.1 /usr/local/lib

cd /usr/local/lib

sudo ln -s libslepc.so.3.21.1 libslepc.so.3.21
sudo ln -s libslepc.so.3.21.1 libslepc.so
cd /home/testuser1/mfem-slepc/mfem/examples/petsc
export LD_LIBRARY_PATH=/usr/local/lib:/home/testuser1/mfem-slepc/petsc/arch-linux-c-debug/lib
make ex11p
mpirun -np 4 ./ex11p -m ../../data/star.mesh --slepcopts rc_ex11p_lobpcg
mpirun -np 4 ./ex11p -m ../../data/star.mesh --slepcopts rc_ex11p_gd
```

