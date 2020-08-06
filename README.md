# QGIS4CentOS8
A tutorial on compiling QGIS latest for CentOS 8

Currently QGIS is not available in the epel-release repository(CentOS 8), so we have to compile one if needed. I tried several times and finally got the correct way.

*This is not a minimal instruction, you will install extra packages, however it works. I will comment out unnecessary lines as possible as I can.*

0. CentOS GUI

Make sure you are using the GUI version. QGIS is a desktop GIS, that means it runs under a desktop environment. Unless you are running QGIS purely as a server side tool, you have to install GUI if the system do not have one. With command `dnf grouplist` and `dnf groupinstall` you can easily add GUI to your system. If you want to use *Remote Desktop*, please

```bash
sudo dnf install epel-release
sudo dnf update
sudo dnf install xrdp net-tools nano screen krdc
sudo systemctl enable xrdp
sudo systemctl start xrdp
sudo firewall-cmd --zone=public --add-port=3389/tcp --permanent
sudo firewall-cmd --reload
```

However remote GUI is optional unless it is a server without monitor. CentOS is a good OS for servers, so I guess you need it.

1. Install programming languages

```bash
#sudo dnf install -y python2 python2-pip nodejs npm 
sudo dnf install -y gcc gcc-c++ golang python3 python3-pip
sudo dnf install -y java-1.8.0-openjdk-devel java-11-openjdk-devel java-latest-openjdk-devel ant swig
#sudo dnf install -y R-core
sudo dnf install -y git svn
```

2. Install postgres

Execute commands line by line, you have to type "y" while installing some package.

```bash
sudo dnf install https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo dnf -qy module disable postgresql
sudo dnf install postgresql12-server
sudo /usr/pgsql-12/bin/postgresql-12-setup initdb
sudo systemctl enable postgresql-12
sudo systemctl start postgresql-12
sudo dnf install gdal
sudo dnf install postgis30_12
#sudo dnf install pgrouting_12
```

3. Install dependencies

Dozens of packages. Though QGIS's readme has given a list of packages to install, it is for Debian, not Centos. There are differences.

```bash
sudo dnf install -y bison ca-certificates cmake cmake-gui ccache doxygen expect flex
sudo dnf install -y graphviz exiv2-devel expat-devel geos-devel gsl-devel proj-devel
sudo dnf install -y postgresql12-devel libpq-devel gdal-devel
sudo dnf install -y qca-qt5*
sudo dnf install -y qscintilla-qt5-devel libzip libzip-devel
sudo dnf install -y protobuf-devel qt5-qtserialport-devel qt5-qtsvg-devel qt5-qtwebkit-devel
sudo dnf install -y qt5-qtbase-gui qt5-qttools-devel qt5-qtconfiguration-devel
sudo dnf install -y gpsbabel-gui qt5-qtwebengine-devtools qt5-qtlocation-devel qt5-qttools-static
sudo dnf install -y python3-qt5* sip python3-sip-devel python3-pyqt5-sip
sudo dnf install -y netcdf-devel libxml2-devel 
sudo dnf install -y qscintilla-qt5-devel python3-qscintilla-qt5 python3-qscintilla-qt5-devel
sudo dnf install -y etckeeper qtkeychain-qt5-devel protobuf-lite
sudo dnf install -y libspatialite-devel qwt-qt5-devel pcp-pmda-slurm
sudo dnf install ocl-icd ocl-icd-devel opencl-headers opencl-filesystem
sudo dnf install -y gdal30 gdal30-devel geos38-devel
```

4. Compile libspatialindex
**Suppose your home directory is /home/d**

```bash
cd ~
git clone https://github.com/libspatialindex/libspatialindex.git

cd libspatialindex/
cmake .
make
sudo make install
```

5. Compile QGIS
**Suppose your home directory is /home/d**

```bash
cd ~
wget https://qgis.org/downloads/qgis-latest.tar.bz2
tar -xvf qgis-latest.tar.bz2

#maybe the version will be different to yours, please correct it
cd qgis-3.14.1
mkdir build
cd build
cmake ..

**Yes, you got errors!** That's common. I solved this problem by edit the CMakeCaches.txt.
Correct these lines:

```
POSTGRES_CONFIG:FILEPATH=/usr/pgsql-12/bin/pg_config
POSTGRES_LIBRARY:FILEPATH=/usr/pgsql-12/lib/libpq.so
GDAL_CONFIG:FILEPATH=/usr/gdal30/bin/gdal-config
GDAL_CONFIG_PREFER_PATH:STRING=/usr/gdal30/bin
GDAL_INCLUDE_DIR:PATH=/usr/gdal30/includ
GDAL_LIBRARY:FILEPATH=/usr/gdal30/lib/libgdal.so
GEOS_CONFIG:FILEPATH=/usr/geos38/bin/geos-config
GEOS_CONFIG_PREFER_PATH:STRING=/usr/geos38/bin
GEOS_INCLUDE_DIR:PATH=/usr/geos38/include
GEOS_LIBRARY:FILEPATH=/usr/geos38/lib64/libgeos_c.so
SPATIALINDEX_INCLUDE_DIR:PATH=/usr/local/include/spatialindex
SPATIALINDEX_LIBRARY:FILEPATH=/usr/local/lib/libspatialindex.so
OpenCL_INCLUDE_DIR:PATH=/usr/include/CL
OpenCL_LIBRARY:FILEPATH=/usr/lib64/libOpenCL.so
Protobuf_LITE_LIBRARY_RELEASE:FILEPATH=/usr/lib64/libprotobuf-lite.so.15
```

If things have not changed so much, just copy the values above is okay. Some directories are not quite the same if your home directory is not /home/d, or the packages have new version. The most important thing is, **use the postgresql, gdal, geos versions you installed for the PostgreSQL database and PostGIS you are running.** With the most recent versions, there might be errors while compiling. To find which is the right version (for example gdal-libs), you can run command (*DO NOT TYPE Y*, we just want to make clear the dependencies, not really uninstall a package) `dnf remove gdal-libs`, and postgis is not in the list. Then try `dnf remove gdal30-libs`, postgis will also be removed if we continue, so gdal30 is the right version to use. Its installation directory can be found with command `rpm -ql gdal30-libs`.


6. Restart compilation

After correct CMakeCaches.txt, try again:

```bash
cmake ..
make -j4
sudo make install
```

If your computer have N cores, for example 16 cores, you can replace `make -j4` with `make -j16` to make it faster. After all, there will be a shortcut for QGIS in your start menu. 

**May be there will be errors, later I will tell my experiences in dealing with the errors. **

7. Correct the libraries

Some libraries are not correctly linked or added to the system lib dir.  So you can not start it. Here I just copy them to /usr/lib64, you may use ln or modify the environment variables.

```bash
sudo cp /usr/local/lib/libqgis* /usr/lib64/
sudo cp ~/libspatialindex/bin/*.* /usr/lib64/
```

Then you can start it, but with some warnings. We have to install some python plugins.

8. Python plugins

```bash
sudo dnf install -y python3-gdal
sudo pip3 install  OSGeo-Easy
sudo pip3 install  pyows OWSlib jinja2 pygments
```

Now we can use QGIS. Enjoy it!


**9. Dealing with errors**

+ cmacke errors

If cmake complains some packages is not found or some variable is not defined, find out which package is needed and install it. Be patient and careful, it cost me hours to get the full list of packages to install.

+ packages installed but not recognized

If cmake can not find a package automatically, correct CMakeCaches.txt by hand.

+ make errors

If incompatible versions of geos-gdal-postgis-postgresql are used, some variables can not be found in the library. Correct CMakeCaches.txt with the right path.

+ QGIS do not start

Run it in terminal, then there will be extra information.

+ Library not found

Link or copy them to the system library directory /usr/lib64.

+ Python plugin not found

Yes you have to install them with pip3, however sometimes the name is not quite the same with the name used in warning messages. Just search and try, it cost me less than one hour.

+ GRASS not found

Now grass relies on some python package, that package relies on something can not installed with pip3 or dnf, so I do not install it, and most functions are okay. If necessary, you have to try it yourself.

+ Binaries

I'll share my compilations of QGIS and libspatialindex. However you have to install all dependencies, and I'm not sure it can run on your machine.
 
