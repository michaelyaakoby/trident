# Setting up development environment for Trident

## Using Centos
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

