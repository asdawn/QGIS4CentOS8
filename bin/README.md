*These binaries (with source codes) are compiled in a virtual machine. The CPU model is AMD R9 3950X (CPU might affect the binaries even in virtual machines).
I suggest you compile it yourself, unless time is not enough.*

```bash
7za x D.7z.001
cd cd libspatialindex/  
sudo make install
cd ..
cd qgis3.14.1
cd build
sudo make install
```

To make it work, please refer to README.md of this project. Step 4, 5 and 6 can be skipped.



