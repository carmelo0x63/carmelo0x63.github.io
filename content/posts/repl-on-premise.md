---
title: "Running a REPL on-premise"
date: 2020-03-29T12:00:00+01:00
draft: false
---

### Nodebook
Nodebook is a multi-language [REPL](https://en.wikipedia.org/wiki/Read–eval–print_loop) offering both a web UI and a CLI interface. Github: [https://github.com/netgusto/nodebook](https://github.com/netgusto/nodebook)

**NOTE**: REPL: [Read-Eval-Print Loop](https://en.wikipedia.org/wiki/Read–eval–print_loop)

#### Set up OS (e.g. CentOS 7)
```
sudo sed -i "s/^#PermitRootLogin yes/PermitRootLogin prohibit-password/" /etc/ssh/sshd_config

sudo sed -i "s/^#UseDNS yes/UseDNS no/" /etc/ssh/sshd_config

sudo systemctl restart sshd

sudo firewall-cmd --permanent --add-port=9001/tcp && sudo firewall-cmd --reload
```
**IMPORTANT**: be aware of any security implcations when running such an application. There's always a chance to run malicious code that can harm the guest OS. By default the application listens and responds to requests coming from localhost only. This setting can be overridden by using `--bindaddress`, see below.

#### Install Docker
```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

sudo yum install -y docker-ce docker-ce-cli containerd.io

sudo systemctl enable --now docker

sudo usermod -aG docker mellowiz
```

#### Deploy Nodebook
```
curl -LO https://github.com/netgusto/nodebook/releases/download/0.2.0/nodebook-linux

sudo mv nodebook-linux /usr/local/bin && sudo chown root:root /usr/local/bin/nodebook-linux && sudo chmod +x /usr/local/bin/nodebook-linux

mkdir -p Nodebooks/Golang Nodebooks/Python3
```

The directory above, `Nodebooks`, will be used to store any code we'll create and run on our REPL. To start with, let's create the following files:

`Python3/main.py`:
```
cat << EOF > Nodebooks/Python3/main.py
print('Hello, World!')
EOF
```

`Golang/main.go`:
```
cat << EOF > Nodebooks/Golang/main.go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
EOF
```

#### Run
```
nodebook-linux --bindaddress <ip_address> --port <port> --docker ~/Nodebooks &
```

Point your browser to `<ip_address>:<port>` and have fun!

