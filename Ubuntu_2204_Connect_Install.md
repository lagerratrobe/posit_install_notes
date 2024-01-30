## New Connect Server setup on Ubuntu 22.04 EC2

### 1. Install R (https://docs.posit.co/resources/install-r/)

```
sudo apt update
sudo apt install gdebi-core
export R_VERSION=4.2.3
curl -O https://cdn.rstudio.com/r/ubuntu-2204/pkgs/r-${R_VERSION}_1_amd64.deb
sudo gdebi r-${R_VERSION}_1_amd64.deb
curl -O https://cdn.rstudio.com/r/ubuntu-2204/pkgs/r-${R_VERSION}_1_amd64.deb 
sudo gdebi r-${R_VERSION}_1_amd64.deb
sudo ln -s /opt/R/${R_VERSION}/bin/R /usr/local/bin/R
sudo ln -s /opt/R/${R_VERSION}/bin/Rscript /usr/local/bin/Rscript

# Should end up with this
$ ll /usr/local/bin | grep R
lrwxrwxrwx  1 root root   18 Jan 30 19:20 R -> /opt/R/4.2.3/bin/R*
lrwxrwxrwx  1 root root   24 Jan 30 19:21 Rscript -> /opt/R/4.2.3/bin/Rscript*
```

### 2. Install Optional system libraries https://docs.posit.co/connect/admin/r/dependencies/  (Jan 30, 2024)

```
sudo apt install -y tcl tk tk-dev tk-table default-jdk cmake git libpng-dev libjpeg-dev make imagemagick libmagick++-dev gsfonts libssl-dev libfreetype6-dev libfribidi-dev libharfbuzz-dev libfontconfig1-dev python3 libsodium-dev libglu1-mesa-dev libgl1-mesa-dev zlib1g-dev libcairo2-dev libssh2-1-dev libudunits2-dev unixodbc-dev libxml2-dev libmysqlclient-dev libnode-dev libcurl4-openssl-dev libtiff-dev libicu-dev libglpk-dev libgmp3-dev libgdal-dev gdal-bin libgeos-dev libproj-dev libsqlite3-dev
```

Test that the optional libs installed correctly

```
gdalinfo --version
GDAL 3.4.1, released 2021/12/27

geos-config --version
3.10.2
```

### 3. Install Python (https://docs.posit.co/resources/install-python/)

```
export PYTHON_VERSION="3.8.15"
curl -O https://cdn.rstudio.com/python/ubuntu-2204/pkgs/python-${PYTHON_VERSION}_1_amd64.deb
sudo gdebi python-${PYTHON_VERSION}_1_amd64.deb

sudo /opt/python/"${PYTHON_VERSION}"/bin/pip install --upgrade \
    pip setuptools wheel
```

### 4. Install Connect (https://docs.posit.co/connect/admin/getting-started/local-install/manual-install/)

```
curl -O https://cdn.posit.co/connect/2024.01/rstudio-connect_2024.01.0~ubuntu22_amd64.deb && 
sudo gdebi rstudio-connect_2024.01.0~ubuntu22_amd64.deb

sudo systemctl status rstudio-connect
```

### 5. Activate your Connect lic. (https://docs.posit.co/connect/admin/getting-started/local-install/manual-install/#step-2.-verify-and-activate-your-license)

I'm using a license file here, if you have a key, follow the instructions for using that.

```
sudo chown root connect.lic \
sudo chmod 0600 connect.lic \
sudo cp -a connect.lic /var/lib/rstudio-connect/

# Restart the service

# Check the status of the license
$ sudo /opt/rstudio-connect/bin/license-manager status

-- License file status --

Status: Activated
Product-Key: XXXX-XXXX-XXXX-XXXX-XXXX-XXXX-9NTA
Has-Key: Yes
Has-Trial: No
Enable-Launcher: 1
Users: 20
User-Activity-Days: 365
Shiny-Users: 20
Allow-APIs: 1
Licensee: RStudio (Test Account)
License-File: /var/lib/rstudio-connect/connect.lic
Expiration: 2024-09-01 00:00:00
Days-Left: 215
License-Engine: 1.0.0.0
License-Scope: System
```	

### 6. Install Quarto (https://docs.posit.co/resources/install-quarto/#quarto-deb-file-install)

```
sudo curl -LO https://quarto.org/download/latest/quarto-linux-amd64.deb
sudo gdebi quarto-linux-amd64.deb

/usr/local/bin/quarto check
```

NOTE:  Ignore the Jupyter, knitr and rmarkdown errors.


### 7. Update Connect config to enable Quarto

```
# /etc/rstudio-connect/rstudio-connect.gcfg
[Quarto]
Enabled = true
Executable = "/usr/local/bin/quarto"
```

### 8. Complete Connect Configuration
At this point, you need to setup your auth and MAIL, etc. https://docs.posit.co/connect/admin/getting-started/local-install/manual-install/#step-3.-initial-configuration
