
# Abaqus 2018 installation instructions for Ubuntu 18.04

## 1. Install prerequisites

```
sudo apt install csh tcsh ksh gcc g++ gfortran libstdc++5 build-essential make libjpeg62 libmotif-dev
```

## 2. Alter all `Linux.sh` files
Installation folders contain a bash file called `Linux.sh`. Search your installation folders and locate all of them. The locations may vary but at the time of writing they were found inside the software folders at: `~/1/inst/common/init`. These files make particular settings, two of them were causing an issue at the time of writing which is why some changes to this file are required. The full file, after changes, is given at the [bottom of this document](#linuxSH). The changes are described below.

__1) Force the distribution/release to be compatible__  
The following line in the bash file checks the particular Linux distribution used:
```sh
DSY_OS_Release=`lsb_release --short --id |sed 's/ //g'`
```
On Ubuntu this will cause it to be "Ubuntu". It appears however Ubuntu is not officially supported and the installation will not proceed because of this. The solution is to manually override the distribution. To do this open this file (e.g. `sudo gedit Linux.sh`) and comment out the above line (by placing a `#` in front of it). Once the line is commented out, add a new line which forces this parameter to denote a supported distribution, e.g. `CentOS`:
```sh
DSY_OS_Release="CentOS"
```
Note, judging from the bash file, setting other distributions may be equivalent:
```sh
"RedHatEnterpriseServer"|"RedHatEnterpriseClient"|"RedHatEnterpriseWorkstation"|"CentOS"
```
In some cases installation stalls when prerequisites are not detected (or not detected properly). After installing the `motif` package for instance the Abaqus installation may still claim it is not installed. Therefore, some users have reported that "system checks" should be disabled by also adding the line:   
```sh
export DSY_Skip_CheckPrereq=1
```
If there are many `Linux.sh` files to alter users may wish to edit them using something like:
```sh
for f in $(find //home/kevin/Downloads/AM_SIM_Abaqus_Extend.AllOS -name "Linux.sh" -type f); do
        sudo gedit $f
done
```

---

If you use openSUSE 15, It is easier to change /usr/bin/lsb_release from
```sh
MSG_DISTRIBUTOR="openSUSE project"
```
to
```sh
MSG_DISTRIBUTOR="SUSE"
```
First make a backup of lsb_release
```sh
sudo cp /usr/bin/lsb_release /usr/bin/lsb_release_bak
```
Then change the line in question with vim
```sh
sudo vim /usr/bin/lsb_release
```
Once the installation is complete (you may need to install motif with ```sudo zypper install motif```), then you can restore the lsb_release information.
```sh
sudo cp /usr/bin/lsb_release_bak /usr/bin/lsb_release
```

## 3. Run installation GUIs
From the relevant folders run:
```
sudo ./StartGUI.sh
```
## 4. Make abaqus command available from any directory  
`sudo ln /var/DassaultSystemes/SIMULIA/Commands/abq2018 /usr/bin/abaqus`

---
### The `Linux.sh` file (as per 2018/08/16): <a name="linuxSH"></a>
Note this file contains two changes: 1) the release version was forced to be `"CentOS"`, and 2) checking of prerequisites was disabled.
```sh
DSY_LIBPATH_VARNAME=LD_LIBRARY_PATH

which lsb_release
if [[ $? -ne 0 ]] ; then
  echo "lsb_release is not found: check in the PDIR the list of installed packages for servers validation."
  exit 12
fi

DSY_OS_Release="CentOS" #Override release setting, old: DSY_OS_Release=`lsb_release --short --id |sed 's/ //g'`
echo "DSY_OS_Release=\""${DSY_OS_Release}"\""
export DSY_OS_Release=${DSY_OS_Release}
export DSY_Skip_CheckPrereq=1 #Added to avoid prerequisite check

if [[ -n ${DSY_Force_OS} ]]; then
  DSY_OS=${DSY_Force_OS}
  echo "DSY_Force_OS=\""${DSY_Force_OS}"\", use it for DSY_OS"
  return
fi

case ${DSY_OS_Release} in
    "RedHatEnterpriseServer"|"RedHatEnterpriseClient"|"RedHatEnterpriseWorkstation"|"CentOS")
        DSY_OS=linux_a64;;
    "SUSELINUX"|"SUSE")
        DSY_OS=linux_a64;;
    *)
        echo "Unknown linux release \""${DSY_OS_Release}"\""
        echo "exit 8"
        exit 8;;
esac
```
### Notes on the abq2018 solver

It has been pointed out that the *standard* process solver does not [terminate correctly](https://askubuntu.com/questions/1062058/process-hangs-before-termination-with-ubuntu-18-04/1111991#1111991). 

I would like to present my work around for this issue. I've made a python wrapper for the abq2018 solver which checks the .sta file for completeness. Once the .sta file is complete, any process named standard will be killed. I've found that the solver exits gracefully when standard is killed and the analysis is complete.   

**This work around is not a perfect solution. Current issues with this work around**:

 1. can't replace the abq2018 solver call directly
 2. will not work from GUI, must be run from the shell
 3. only parses job= argument
 4. you can only run one analysis at at time since all *standard* processes are killed
 5. abq will hang forever if .sta file is not created or modified

**How to use this workaround**:

 1. Create Python file named abq. Code for abq is detailed below. If you are using a solver other than abq2018, replace the line cmd = 'abq20xx.. with the solver that you are using.
 2. Make abq executable and available in your path. I placed abq in the Abaqus commands folder, then ran ```chmod +x abq```
 3. Run an Abaqus standard job by executing ```abq job=Job-1```. This will execute Job-1.inp, then this will kill the standard solver once Job-1.sta is completed.

take a look at the code for [abq](abq)

