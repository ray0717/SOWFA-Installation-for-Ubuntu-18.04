# SOWFA-Installation-for-Ubuntu-18.04
## OpenFOAM-2.4.0 Install
### Install the necessary packages
```shell
$ sudo apt-get update
$ sudo apt-get install build-essential cmake flex bison zlib1g-dev qt4-dev-tools libqt4-dev libqtwebkit-dev gnuplot \
libreadline-dev libncurses5-dev libxt-dev libopenmpi-dev openmpi-bin libboost-system-dev libboost-thread-dev libgmp-dev \
libmpfr-dev python python-dev libcgal-dev gcc-5 g++-5 -y
$ sudo apt-get install libglu1-mesa-dev libqt4-opengl-dev -y
```
### Download and unpack
```shell
$ cd ~ && mkdir OpenFOAM && cd OpenFOAM
$ wget "http://downloads.sourceforge.net/foam/OpenFOAM-2.4.0.tgz?use_mirror=mesh" -O OpenFOAM-2.4.0.tgz && tar -xzf OpenFOAM-2.4.0.tgz
$ wget "http://downloads.sourceforge.net/foam/ThirdParty-2.4.0.tgz?use_mirror=mesh" -O ThirdParty-2.4.0.tgz && tar -xzf ThirdParty-2.4.0.tgz
```
### MPI symbolic links
```shell
$ ln -s /usr/bin/mpicc.openmpi OpenFOAM-2.4.0/bin/mpicc
$ ln -s /usr/bin/mpirun.openmpi OpenFOAM-2.4.0/bin/mpirun
```
### Change the default Boost, CGAL versions and GCC version <br>- Change WM_NCOMPPROCS=4 to number of cores
```shell
$ sed -i -e 's/\(cgal_version=\)CGAL-4.6/\1cgal-system/' OpenFOAM-2.4.0/etc/config/CGAL.sh
$ sed -i -e 's=\-lmpfr=-lmpfr -lboost_thread=' OpenFOAM-2.4.0/wmake/rules/General/CGAL
$ sed -i -e 's/gcc/\$(WM_CC)/' OpenFOAM-2.4.0/wmake/rules/linux*Gcc/c
$ sed -i -e 's/g++/\$(WM_CXX)/' OpenFOAM-2.4.0/wmake/rules/linux*Gcc/c++
$ source $HOME/OpenFOAM/OpenFOAM-2.4.0/etc/bashrc WM_NCOMPPROCS=4 WM_MPLIB=SYSTEMOPENMPI
$ export WM_CC='gcc-5'
$ export WM_CXX='g++-5'
```
### Save alias, run of240 when starting new terminal
```shell
$ echo "alias of240='source \$HOME/OpenFOAM/OpenFOAM-2.4.0/etc/bashrc $FOAM_SETTINGS; export WM_CC=gcc-5; export WM_CXX=g++-5'" >> $HOME/.bashrc
```
### Build ThirdParty folder
```shell
$ cd $WM_THIRD_PARTY_DIR
$ export QT_SELECT=qt4
$ sed -i -e 's|\(^if.*CGAL_ARCH_PATH.*\)]|\1 -a "${CGAL_ARCH_PATH##*/}" != "cgal-system" ]|' Allwmake
$ ./Allwmake > log.make 2>&1
$ of240
```
### Build ParaView
```shell
$ export QT_SELECT=qt4
$ cd $WM_THIRD_PARTY_DIR
$ sed -i -e 's=MPI_ARCH_PATH/include=MPI_ARCH_PATH/include;$MPI_INCLUDE=' etc/tools/ParaView4Functions
$ sed -i -e 's=//#define GLX_GLXEXT_LEGACY=#define GLX_GLXEXT_LEGACY=' \
  ParaView-4.1.0/VTK/Rendering/OpenGL/vtkXOpenGLRenderWindow.cxx
$ cd $WM_THIRD_PARTY_DIR/ParaView-4.1.0 && wget http://www.paraview.org/pipermail/paraview/attachments/20140210/464496cc/attachment.bin -O Fix.patch && patch -p1 < Fix.patch
$ cd VTK && wget https://github.com/gladk/VTK/commit/ef22d3d69421581b33bc0cd94b647da73b61ba96.patch -O Fix2.patch && patch -p1 < Fix2.patch
$ cd ../..
$ ./makeParaView4 -python -mpi -python-lib /usr/lib/x86_64-linux-gnu/libpython2.7.so.1.0 > log.makePV 2>&1
$ of240
```
### Build OpenFOAM
```shell
$ cd $WM_PROJECT_DIR
$ find src applications -name "*.L" -type f | xargs sed -i -e 's=\(YY\_FLEX\_SUBMINOR\_VERSION\)=YY_FLEX_MINOR_VERSION < 6 \&\& \1='
$ cd $WM_PROJECT_DIR
$ export QT_SELECT=qt4
$ ./Allwmake > log.make 2>&1
```
### Check OpenFOAM installation
```shell
$ icoFoam -help
$ of240
```
## SOWFA install
### SOWFA code download
```shell
$ cd ${HOME} && git clone https://github.com/NREL/SOWFA.git && cd SOWFA
```
### SOWFA installation
```shell
$ export SOWFA_DIR="${HOME}/SOWFA"
$ cd ${SOWFA_DIR}
$ ./Allwmake > log.make 2>&1
```
### Set variables
#### Add below 2 lines to ~/.bashrc
```
export LD_LIBRARY_PATH=${SOWFA_DIR}/lib/${WM_OPTIONS}/:${LD_LIBRARY_PATH}
export PATH="${PATH}:${SOWFA_DIR}/applications/bin/${WM_OPTIONS}"
```
```shell
$ source ~/.bashrc
```
## Reference
https://github.com/pablo-benito/SOWFA-installation <br>
https://openfoamwiki.net/index.php/Installation/Linux/OpenFOAM-2.4.0/Ubuntu#Ubuntu_18.04 <br>
https://github.com/NREL/SOWFA/blob/master/README.SOWFA
