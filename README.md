# personal use of sspanel backend
install with the following shell command


### 1. install libsodium to support high-level encrypt algorithm

For Debian/Ubuntu
```shell
apt install -y libsodium-dev
```

For CentOS/Rocky/Alma/RHEL

```shell
dnf install -y epel-release
dnf makecache
dnf install -y libsodium-devel
```

### 2. configure other environments

For Python2.x

```shell
cd /root
curl -sSL https://bootstrap.pypa.io/pip/2.7/get-pip.py -o get-pip.py
python get-pip.py
git clone https://github.com/innocentiuss/ss_pxy_server_mion.git
cd ss_pxy_server_mion/shadowsocks
pip install -r requirements.txt
cp apiconfig.py userapiconfig.py
cp config.json user-config.json
```

For Python3.x
```shell
cd /root
dnf -y install python3-pip
git clone https://github.com/innocentiuss/ss_pxy_server_mion.git
cd ss_pxy_server_mion/shadowsocks
pip install -r requirements.txt
cp apiconfig.py userapiconfig.py
cp config.json user-config.json
```

### 3. configure web backend API configuration

```shell
vim userapiconfig.py
```

### 4. test/start

```shell
python server.py
./run.sh
```


### 5. configure start on boot with systemd

The following example applies to Debian/Ubuntu and RHEL/CentOS/Rocky/AlmaLinux systems that use `systemd`.

> **Notes**
>
> - The example assumes that the project is installed at `/root/ss_pxy_server_mion`.
>   Replace this path with the actual installation path if it is different.
> - The service runs `server.py m` directly in the foreground. Do not use `./run.sh`
>   as `ExecStart`, because `run.sh` starts the process with `nohup` in the background.
> - The example preserves the existing root-based installation. If `python` is not
>   available at `/usr/bin/python`, replace the Python path in `ExecStart` with the
>   result of `command -v python`, `command -v python2`, or `command -v python3`.

Check the Python executable path first:

```shell
command -v python
python --version
```

Create the systemd service file:

```shell
sudo vim /etc/systemd/system/ss-pxy-server.service
```

Use the following contents:

```ini
[Unit]
Description=ss_pxy_server_mion
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
WorkingDirectory=/root/ss_pxy_server_mion/shadowsocks
ExecStart=/usr/bin/python /root/ss_pxy_server_mion/shadowsocks/server.py m
Restart=on-failure
RestartSec=5
LimitNOFILE=512000

[Install]
WantedBy=multi-user.target
```

Before the first switch from manual startup, stop the existing `run.sh` process to avoid duplicate processes or port conflicts:

```shell
cd /root/ss_pxy_server_mion/shadowsocks
./stop.sh
```

#### Debian/Ubuntu

```shell
sudo systemctl daemon-reload
sudo systemctl enable --now ss-pxy-server.service
sudo systemctl status --no-pager ss-pxy-server.service
```

#### RHEL/CentOS/Rocky/AlmaLinux

```shell
sudo systemctl daemon-reload
sudo systemctl enable --now ss-pxy-server.service
sudo systemctl status --no-pager ss-pxy-server.service
```

`enable --now` enables the service at boot and starts it immediately. Useful maintenance commands:

```shell
# View service logs
sudo journalctl -u ss-pxy-server.service -f

# Restart the service after changing the configuration
sudo systemctl restart ss-pxy-server.service

# Stop the service
sudo systemctl stop ss-pxy-server.service

# Stop the service and disable startup on boot
sudo systemctl disable --now ss-pxy-server.service
```

