Posit Connect on RHEL 8 Setup Notes
---------------------------------------------------------

Please note that these notes were taken a few months ago and that the versions of all products should be checked and updated to the latest available, or whatever versions your data science team uses.

## Setup VM
- Software Image (AMI)
- Red Hat Enterprise Linux 8
- Virtual server type (instance type)
- t3.medium
- 1 volume(s) - 10 GiB

_NOTE: Inbound port 3939 needs to be opened in the network policies for the machine in order to test in HTTP mode._ 

### Prereqs
```
$ sudo dnf update 

# Enable the Extra Packages for Enterprise Linux (EPEL) repository
$ sudo dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
$ sudo dnf install -y dnf-plugins-core
```

### Install R
https://docs.posit.co/resources/install-r/

```
 $  export R_VERSION=4.2.3
 $  curl -O https://cdn.rstudio.com/r/centos-8/pkgs/R-${R_VERSION}-1-1.x86_64.rpm
 
 $  sudo dnf install -y R-${R_VERSION}-1-1.x86_64.rpm
 
 $  sudo ln -s /opt/R/${R_VERSION}/bin/R /usr/local/bin/R
 $  sudo ln -s /opt/R/${R_VERSION}/bin/Rscript /usr/local/bin/Rscript
```

### Install Python
https://docs.posit.co/resources/install-python/

```
$  export PYTHON_VERSION="3.8.15"
$  curl -O https://cdn.rstudio.com/python/centos-8/pkgs/python-${PYTHON_VERSION}-1-1.x86_64.rpm

$  sudo dnf install -y python-${PYTHON_VERSION}-1-1.x86_64.rpm

$  sudo /opt/python/"${PYTHON_VERSION}"/bin/pip install --upgrade     pip setuptools wheel
$  /opt/python/3.8.15/bin/python -m pip install --upgrade pip

$  sudo vi /etc/profile.d/python.sh

PATH=/opt/python/3.8.15/bin:$PATH
```

### Install Connect

https://docs.posit.co/rsc/manual-install/

```
$ curl -O https://cdn.rstudio.com/connect/2023.09/rstudio-connect-2023.09.0.el8.x86_64.rpm
$ sudo dnf install rstudio-connect-2023.09.0.el8.x86_64.rpm 

# Verify the rstudio-connect service is up
$ sudo systemctl status rstudio-connect
```

### Install System Dependencies

NOTE:  You need to make sure that you've enabled the `codeready-builder` repo ahead of time to install some of these.

`sudo subscription-manager repos --enable codeready-builder-for-rhel-8-x86_64-rpms`

```
$ sudo dnf install -y tcl tk tk-devel java-1.8.0-openjdk-devel libxml2-devel openssl-devel ImageMagick ImageMagick-c++ unixODBC-devel freetype-devel fribidi-devel harfbuzz-devel git make geos-devel libpng-devel udunits2-devel fontconfig-devel cairo-devel libgit2-devel libssh2 zlib-devel perl cmake libsodium-devel libicu-devel glpk-devel gmp-devel mesa-libGLU-devel mesa-libGL-devel libcurl-devel mariadb-devel libjpeg-turbo-devel libtiff-devel gdal-devel gdal sqlite-devel proj-devel
```

### Test that an app that requires those system libs works

```
$ gdal_translate --version
GDAL 3.0.4, released 2020/01/28

$ geos-config --version
3.7.2
```

### Install Quarto

See - https://docs.posit.co/resources/install-quarto/.

```
#prep
$ export QUARTO_VERSION="1.3.450"
$ sudo mkdir -p /opt/quarto/${QUARTO_VERSION}

# download
$  sudo curl -o quarto.tar.gz -L     "https://github.com/quarto-dev/quarto-cli/releases/download/v${QUARTO_VERSION}/quarto-${QUARTO_VERSION}-linux-amd64.tar.gz"

# install
$  sudo tar -zxvf quarto.tar.gz     -C "/opt/quarto/${QUARTO_VERSION}"     --strip-components=1

# test
$  /opt/quarto/"${QUARTO_VERSION}"/bin/quarto check
  ```

### Activate the License

Before Lic is active:

```
$ sudo /opt/rstudio-connect/bin/license-manager status

TTY detected. Printing informational message about logging configuration. Logging configuration loaded from '/etc/rstudio/logging.conf'. Logging to '/var/log/rstudio/rstudio-server/license-manager.log'.
RStudio License Manager 2022.11.0-daily+174.pro1

-- Local license status --

Trial-Type: Verified
Status: Evaluation
Days-Left: 45
License-Scope: System
License-Engine: 4.4.3.0

-- Floating license status --

License server not in use.
```

After Lic is activated:

```
$ sudo /opt/rstudio-connect/bin/license-manager activate XXXX-XXXX-XXXX-XXXX-XXXX-XXXX-XXXX

TTY detected. Printing informational message about logging configuration. Logging configuration loaded from '/etc/rstudio/logging.conf'. Logging to '/var/log/rstudio/rstudio-server/license-manager.log'.

Status: Activated
Product-Key: XXXX-XXXX-XXXX-XXXX-XXXX-XXXX-XXXX
Has-Key: Yes
Has-Trial: Yes
Users: 20
User-Activity-Days: 365
Shiny-Users: 20
Allow-APIs: 1
Expiration: 2024-06-09 00:00:00
Days-Left: 550
License-Engine: 4.4.3.0
License-Scope: System
```

### Complete Configuration

At this point, you need to setup your auth and MAIL, etc.  https://docs.posit.co/connect/admin/getting-started/local-install/manual-install/#step-3.-initial-configuration

### Basic config for testing

Connect has one config file located in `/etc/rstudio/rstudio-connect.gcfg`.  Below is a simple version of this file for a single-node server running without SSO and TLS.

```
[Server]
SenderEmail = roger.andre@posit.co
Address = http://ec2-18-222-224-92.us-east-2.compute.amazonaws.com:3939
DataDir = /data

[Licensing]
LicenseType = local

[Database]
Provider = sqlite

[HTTP]
Listen = :3939
NoWarning = TRUE

[Authentication]
Provider = password

[RPackageRepository "CRAN"]
URL = https://packagemanager.posit.co/cran/__linux__/centos8/latest

[Python]
Enabled = true
Executable = /opt/python/3.8.15/bin/python

[Quarto]
Enabled = true
Executable = /opt/quarto/1.3.450/bin/quarto

[Applications]
MostPermissiveAccessType = all
AdminMostPermissiveAccessType = all

[Logging]
ServiceLog = STDOUT
ServiceLogFormat = TEXT    ; TEXT or JSON
ServiceLogLevel = INFO     ; INFO, WARNING or ERROR
AccessLog = STDOUT
AccessLogFormat = COMMON   ; COMMON, COMBINED, or JSON
```
