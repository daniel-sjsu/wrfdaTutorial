
# WRFDA Tutorial

## Installation

#### Cloning Repository
I've personally found it beneficial to have a parent directory to contain my WRF/WRFDA installations, especially since 4Dvar and 3Dvar are two separate installations.
```bash
mkdir WRFSFIRE
cd WRFSFIRE
```

Clone WRF-SFIRE by using one of the following commands

```bash
git clone git://github.com/openwfm/WRF-SFIRE.git WRF-SFIRE-DA
# Note: This command has never worked for me.
# Use the following if it doesn't
git clone git://repo.or.cz/WRF-SFIRE.git WRF-SFIRE-DA
```
Some more information about WRF-SFIRE located [here](https://wiki.openwfm.org/wiki/How_to_get_WRF-SFIRE).


---

#### Compiling 3D-VAR 
More information found in the [UCAR Guide](https://www2.mmm.ucar.edu/wrf/users/wrfda/Docs/user_guide_V3.7.1/users_guide_chap6.htm). When copying commands from that link, be careful since they sometimes use a different "-" which causes problems.

Compilation requires netCDF to be installed, refer to this [tutorial](https://www2.mmm.ucar.edu/wrf/OnLineTutorial/compilation_tutorial.php#STEP2) for instructions on installing.

```bash
export NETCDF="/path_to_directory"
```
However, it is best to load in other libraries used for WRF in general. This avoids messages and limitations to the installation like this:<br> 
`HDF5 not set in environment. Will configure WRF for use without.`<br>
`$JASPERLIB or $JASPERINC not found in environment, configuring to build without grib2 I/O...`<br>
Assuming those modules have been installed to run WRF or WRF-SFIRE previously, a script like this can be used to load in all the modules. 

```bash
module purge
module load slurm
module load gcc8
export DIR="$HOME/Utility_WRF/Library_gcc8"
export PATH="$DIR/bin:$PATH"
export LD_LIBRARY_PATH="$DIR/lib:$LD_LIBRARY_PATH"
export CC="gcc"
export CXX="g++"
export FC="gfortran"
export FCFLAGS="-m64"
export F77="gfortran"
export FFLAGS="-m64"
export LDFLAGS="-L$DIR/lib"
export CPPFLAGS="-I$DIR/include"
export NETCDF="$DIR"
export JASPERLIB="$DIR/lib"
export JASPERINC="$DIR/include"
export OMPI_MCA_btl_openib_allow_ib=1
export WRFIO_NCD_LARGE_FILE_SUPPORT=1


export WRF_EM_CORE=1
export WRF_NMM_CORE=0
```

In order to be able to assimilate BUFR or PREPBUFR files. (Uneeded in case of lidars etc)
```bash
export BUFR=1
```
Satellite radiance can also be used, refer to users guide if this is wanted.

```bash
cd WRF-SFIRE-DA
./configure wrfda
```
```
Please select from among the following Linux x86_64 options:

  1. (serial)   2. (smpar)   3. (dmpar)   4. (dm+sm)   PGI (pgf90/gcc)
  5. (serial)   6. (smpar)   7. (dmpar)   8. (dm+sm)   PGI (pgf90/pgcc): SGI MPT
  9. (serial)  10. (smpar)  11. (dmpar)  12. (dm+sm)   PGI (pgf90/gcc): PGI accelerator
 13. (serial)  14. (smpar)  15. (dmpar)  16. (dm+sm)   INTEL (ifort/icc)
                                         17. (dm+sm)   INTEL (ifort/icc): Xeon Phi (MIC architecture)
 18. (serial)  19. (smpar)  20. (dmpar)  21. (dm+sm)   INTEL (ifort/icc): Xeon (SNB with AVX mods)
 22. (serial)  23. (smpar)  24. (dmpar)  25. (dm+sm)   INTEL (ifort/icc): SGI MPT
 26. (serial)  27. (smpar)  28. (dmpar)  29. (dm+sm)   INTEL (ifort/icc): IBM POE
 30. (serial)               31. (dmpar)                PATHSCALE (pathf90/pathcc)
 32. (serial)  33. (smpar)  34. (dmpar)  35. (dm+sm)   GNU (gfortran/gcc)
 36. (serial)  37. (smpar)  38. (dmpar)  39. (dm+sm)   IBM (xlf90_r/cc_r)
 40. (serial)  41. (smpar)  42. (dmpar)  43. (dm+sm)   PGI (ftn/gcc): Cray XC CLE
 44. (serial)  45. (smpar)  46. (dmpar)  47. (dm+sm)   CRAY CCE (ftn $(NOOMP)/cc): Cray XE and XC
 48. (serial)  49. (smpar)  50. (dmpar)  51. (dm+sm)   INTEL (ftn/icc): Cray XC
 52. (serial)  53. (smpar)  54. (dmpar)  55. (dm+sm)   PGI (pgf90/pgcc)
 56. (serial)  57. (smpar)  58. (dmpar)  59. (dm+sm)   PGI (pgf90/gcc): -f90=pgf90
 60. (serial)  61. (smpar)  62. (dmpar)  63. (dm+sm)   PGI (pgf90/pgcc): -f90=pgf90
 64. (serial)  65. (smpar)  66. (dmpar)  67. (dm+sm)   INTEL (ifort/icc): HSW/BDW
 68. (serial)  69. (smpar)  70. (dmpar)  71. (dm+sm)   INTEL (ifort/icc): KNL MIC
 72. (serial)  73. (smpar)  74. (dmpar)  75. (dm+sm)   FUJITSU (frtpx/fccpx): FX10/FX100 SPARC64 IXfx/Xlfx

Enter selection [1-75] : 34
```
Select option 34, it should return "Configuration successful!" and then a bunch of other configuration information.

```bash
./compile all_wrfvar >& compile.out
```
Upon completion of this compilation, there should be 44 executables. To quickly view count
```bash
ls -l var/build/*exe var/obsproc/src/obsproc.exe | wc -l
```
If this does not return 44, look into the compile.out file for errors, happened to me originally but I forget what the errors were.

---
#### Running 3D-Var Test Case 
The tutorial website has a test case but it is for an older version of WRF so numerous issues arise from this. To avoid that problem, download test case files.<br>
You can skip this if you want to make your own case or are using a newer version of WRFSFIRE(V4.4). <br>
Change directories to where you'd like to run your testing (For example back into the WRFSFIRE parent directory)
```bash
wget https://github.com/daniel-sjsu/wrfdaTutorial/archive/refs/tags/v1.0.0.zip
unzip v1.0.0.zip
mv wrfdaTutorial-1.0.0 testCaseWRFDA
ls testCaseWRFDA
```
This should contain the following files
```
19:00:00_fake.txt  
wrfbdy_d01  
wrfinput_d01
namelist.obsproc
namelist.input
parame.in
```

---

##### Obsproc
You could also navigate to the obsproc folder itself and copy 19:00:00_fake.txt but this method is cleaner. For more information on what obsproc does, refer to the ucar guide but it basically takes in the little_r filepath taken from namelist.obsproc, processes it and returns an ascii file.<br>
Change the path to WRFDA_DIR to lead to your WRF-SFIRE-DA installation. 
```bash
cd testCaseWRFDA
export WRFDA_DIR="/path_to_directory"
ln -sf $WRFDA_DIR/var/obsproc/obsproc.exe .
ln -sf $WRFDA_DIR/var/obsproc/obserr.txt .
ln -sf $WRFDA_DIR/var/obsproc/msfc.tbl .
./obsproc.exe >& obsproc.out
```
Look through the obsproc.out and verify execution was successful. This test case should have 48 multi-level reports (PROFL Reports)
```bash
tail obsproc.out
```
```
OTHER reports:      0
 Total reports:     48 =       0 single +      48 multi-level reports.

------------------------------------------------------------------------------
Write 3DVAR GTS observations in file obs_gts_2022-10-24_19:00:00.3DVAR (WRFDA V4.4)

Wrote    16080 lines of data in file: obs_gts_2022-10-24_19:00:00.3DVAR
 
No SSMI observations available.
```
There should also be this file listed below, this is your ascii file.
```
obs_gts_2022-10-24_19:00:00.3DVAR
```
---
##### Data Assimilation(3D-VAR)
```bash
ln -sf $WRFDA_DIR/var/run/be.dat.cv3 be.dat
ln -sf $WRFDA_DIR/var/da/da_wrfvar.exe .
ln -sf $WRFDA_DIR/var/da/da_update_bc.exe .
ln -sf $WRFDA_DIR/run/*.TBL .
ln -sf wrfinput_d01 ./fg
ln -sf obs_gts_2022-10-24_19:00:00.3DVAR ./ob.ascii
./da_wrfvar.exe
tail rsl.out.0000
```
```*** WRF-Var completed successfully ***```<br>
I'd also recommend looking into rsl.out.0000 to confirm that the "Total number of obs." is greater than 0.<br>
Next step is to update the boundary conditions, this will override the wrfbdy file so if you want to verify a change is made, make sure to have a copy. This looks at parame.in and requires wrfvar_output and wrfbdy_01
```bash
./da_update_bc.exe
```
This should create fort.11 and fort.12 with information about changes made. Once again, if more information is needed, please look at the ucar guide.

Using ncdiff you can check the differences between wrfvar_output and wrfinput_d01 to confirm that assimilation has occurred as expected.

---
#### Running WRF
The wrfvar_output would be the wrfinput_d01 provided to your wrf simulation alongside the new wrfbdy_d01 you created. From here you would just run ```./wrf.exe```
<br>This would not work in this case since you do not have the met_em files