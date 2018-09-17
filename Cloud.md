# Setting up development environment for Trident
This section documents the (currently manual) steps required to setup a development environment for Trident. 

## Using CentOS
Trident's building requires [docker](https://www.docker.com/) to be installed - follow the steps below to install docker, golang, and glide. Then clone and build Trident.
1. Installing golang, see https://golang.org/doc/install
   ```bash
   curl https://dl.google.com/go/go1.11.linux-amd64.tar.gz -o go1.11.linux-amd64.tar.gz
   tar -C /usr/local -xzf go1.11.linux-amd64.tar.gz
   cat <<EOF >> /etc/bashrc
   export PATH=\$PATH:/usr/local/go/bin:/go/bin
   export GOPATH=/go
   EOF
   ```
   * Make sure to pick the latest golang version
   * The `PATH` and `GOPATH` settings assume a single workspace 
2. Create a workspace and clone the git project, see https://golang.org/doc/code.html#Workspaces
   ```bash
   yum install git
   git clone https://github.com/netapp/trident.git /go/src/github.com/netapp/trident
   ```
3. Install Docker
   ```bash
   yum install -y docker
   systemctl enable docker 
   systemctl start docker
   ```
4. Build, see https://github.com/NetApp/trident/blob/master/BUILD.md
   ```bash
   mkdir -p /go/bin
   curl https://glide.sh/get | sh
   cd /go/src/github.com/netapp/trident/
   glide install -v
   setenforce 0
   make build
   ```
   * Note that the `setencforce` is workaround selinux, see https://stackoverflow.com/questions/24288616/permission-denied-on-accessing-host-directory-in-docker
5. Run tests
   ```bash
   yum install -y gcc
   make test
   ```

## Using Windows 10 with WSL
The goal of this section is to enable a developer to:
1. Git clone the Trident project onto a workspace in the `C:` drive, for example `C:\Users\yakobi\go\src\github.com\netapp\trident`
2. Use IDEs such as [GoLand](https://www.jetbrains.com/go/) locally (i.e. not using remote-desktop or VNC) 
3. Build and test locally (without the need to commit changes to a branch that is built remotely)

Given that Trident's build was built for Linux, this requires using the Windows Subsystem for Linux (WSL) in conjunction with docker, as described below:
1. Install the Windows Subsystem for Linux (WSL) as described in <https://docs.microsoft.com/en-us/windows/wsl/install-win10>
2. Install docker for windows, follow the instructions in <https://store.docker.com/editions/community/docker-ce-desktop-windows>
3. Add firewall rules that allow docker containers to access `C:` - from powershell-as-administrator issue the followin command, see https://stackoverflow.com/questions/42203488/settings-to-windows-firewall-to-allow-docker-for-windows-to-share-drive/43904051#43904051
   ```powershell
   Set-NetConnectionProfile -interfacealias "vEthernet (DockerNAT)" -NetworkCategory Private
   ```
4. From docker-settings, share your C drive with your containers as described in <https://nickjanetakis.com/blog/setting-up-docker-for-windows-and-wsl-to-work-flawlessly > (and in <https://blogs.msdn.microsoft.com/stevelasker/2016/06/14/configuring-docker-for-windows-volumes/>)
5. To complete the sharing, from the WSL bash prompt
   ```bash
   sudo ln -s /mnt/c/Program\ Files/Docker/Docker/resources/bin/docker.exe /usr/bin/docker
   sudo mkdir /c
   sudo mount --bind /mnt/c /c
   ``` 
6. Replace WSL's default golang distro (gccgo-go version 4.9 which implements go 1.2 and is too old) with the latest as described in <https://github.com/golang/go/wiki/Ubuntu>
   ```bash
   apt-get remove gccgo-go
   sudo add-apt-repository ppa:gophers/archive
   sudo apt-get update
   sudo apt-get install golang-1.10-go
   ```
7. Configure your golang workspace - in bash:
   ```bash
   cat <<EOF >> ~/.bashrc
   export GOPATH=/c/Users/yakobi/go
   export PATH=$PATH:/c/Users/yakobi/go/bin
   EOF
   ```
8. Build
   ```bash
   cd /go/src/github.com/netapp/trident
   glide install -v
   make build
   ```