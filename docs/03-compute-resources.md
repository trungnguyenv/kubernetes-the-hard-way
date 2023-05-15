# Provisioning Compute Resources
## Hyper-V Network
Run these commands as Administrator to create the 
```powershell
New-VMSwitch -SwitchName "multipass" -SwitchType Internal
$adapter=$(Get-NetAdapter | Where-Object -Property Name -Value ".+multipass.+" -Match | Select-Object -Property "ifIndex")
New-NetIPAddress -IPAddress 192.168.66.1 -PrefixLength 24 -InterfaceIndex $adapter.ifIndex
New-NetNat -Name multipassnat -InternalIPInterfaceAddressPrefix 192.168.66.0/24
```
## Compute Instances

The compute instances in this lab will be provisioned using [Ubuntu Server](https://www.ubuntu.com/server) 20.04, which has good support for the [containerd container runtime](https://github.com/containerd/containerd). Each compute instance will be provisioned with a fixed private IP address to simplify the Kubernetes bootstrapping process.

### Kubernetes Controllers
```
multipass launch --name k8s-cp-01 --network "name=multipass,mode=manual,mac=60:54:00:0C:6D:00" --cpus 2 --memory 2G jammy

multipass exec -n k8s-cp-01 -- sudo bash -c 'cat << EOF > /etc/netplan/10-custom.yaml
network:
    version: 2
    ethernets:
        extra0:
            dhcp4: no
            match:
                macaddress: "60:54:00:0C:6D:00"
            addresses: [192.168.66.2/24]
EOF'

multipass exec -n k8s-cp-01 -- sudo netplan apply

multipass launch --name k8s-cp-02 --network "name=multipass,mode=manual,mac=60:54:00:0C:6D:01" --cpus 2 --memory 2G jammy

multipass exec -n k8s-cp-02 -- sudo bash -c 'cat << EOF > /etc/netplan/10-custom.yaml
network:
    version: 2
    ethernets:
        extra0:
            dhcp4: no
            match:
                macaddress: "60:54:00:0C:6D:01"
            addresses: [192.168.66.3/24]
EOF'

multipass exec -n k8s-cp-02 -- sudo netplan apply
```
### Kubernetes Workers

```
multipass launch --name k8s-work-01 --network "name=multipass,mode=manual,mac=60:54:00:0C:6D:02" --cpus 2 --memory 2G jammy

multipass exec -n k8s-work-01 -- sudo bash -c 'cat << EOF > /etc/netplan/10-custom.yaml
network:
    version: 2
    ethernets:
        extra0:
            dhcp4: no
            match:
                macaddress: "60:54:00:0C:6D:02"
            addresses: [192.168.66.4/24]
EOF'

multipass exec -n k8s-work-01 -- sudo netplan apply

multipass launch --name k8s-work-02 --network "name=multipass,mode=manual,mac=60:54:00:0C:6D:03" --cpus 2 --memory 2G jammy

multipass exec -n k8s-work-02 -- sudo bash -c 'cat << EOF > /etc/netplan/10-custom.yaml
network:
    version: 2
    ethernets:
        extra0:
            dhcp4: no
            match:
                macaddress: "60:54:00:0C:6D:03"
            addresses: [192.168.66.5/24]
EOF'

multipass exec -n k8s-work-02 -- sudo netplan apply
```

## Config /etc/hosts
```
192.168.66.2 k8s-cp-01
192.168.66.3 k8s-cp-02
192.168.66.4 k8s-work-01
192.168.66.5 k8s-work-02
```

```
multipass exec -n k8s-cp-01 -- sudo bash -c 'cat << EOF >> /etc/hosts
# Kubernetes the hard way
192.168.66.2 k8s-cp-01
192.168.66.3 k8s-cp-02
192.168.66.4 k8s-work-01
192.168.66.5 k8s-work-02
EOF'

multipass exec -n k8s-cp-02 -- sudo bash -c 'cat << EOF >> /etc/hosts
# Kubernetes the hard way
192.168.66.2 k8s-cp-01
192.168.66.3 k8s-cp-02
192.168.66.4 k8s-work-01
192.168.66.5 k8s-work-02
EOF'

multipass exec -n k8s-work-01 -- sudo bash -c 'cat << EOF >> /etc/hosts
# Kubernetes the hard way
192.168.66.2 k8s-cp-01
192.168.66.3 k8s-cp-02
192.168.66.4 k8s-work-01
192.168.66.5 k8s-work-02
EOF'

multipass exec -n k8s-work-02 -- sudo bash -c 'cat << EOF >> /etc/hosts
# Kubernetes the hard way
192.168.66.2 k8s-cp-01
192.168.66.3 k8s-cp-02
192.168.66.4 k8s-work-01
192.168.66.5 k8s-work-02
EOF'
```

Next: [Provisioning a CA and Generating TLS Certificates](04-certificate-authority.md)
