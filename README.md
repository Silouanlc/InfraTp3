# TP3 Service Orchestration & Cloud-native environment


VagrantFile :
~~~
$script = <<-SCRIPT
echo I am provisioning...
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce docker-ce-cli containerd.io
systemctl disable firewalld
systemctl stop firewalld
SCRIPT


servers=[
  {
    :hostname => "vm1",
    :ip => "192.168.100.10",
    :box => "centos/7",
    :ram => 2048,
    :cpu => 1,
    :nbdisk => 2
  },
  {
    :hostname => "vm2",
    :ip => "192.168.100.11",
    :box => "centos/7",
    :ram => 2048,
    :cpu => 1,
    :nbdisk => 2
  },
  {
    :hostname => "vm3",
    :ip => "192.168.100.12",
    :box => "centos/7",
    :ram => 2048,
    :cpu => 1,
    :nbdisk => 3
  }
]

Vagrant.configure(2) do |config|
    servers.each do |machine|
        config.vm.define machine[:hostname] do |node|
            node.vm.box = machine[:box]
            node.vm.hostname = machine[:hostname]
            node.vm.network "private_network", ip: machine[:ip]
            node.vm.provider "virtualbox" do |vb|
            node.vm.provision "shell", inline: $script

            #vb.customize [ "createmedium", "disk", "--filename", "mydisk.vmdk", "--format", "vmdk", "--size", 1024 * 10 ]
            #vb.customize [ "storageattach", "myvm" , "--storagectl", "IDE Controller", "--port", "1", "--device", "0", "--type", "hdd", "--medium", "mydisk.vmdk"]
            #if nbdisk: > 2
            #  vb.customize [ "createmedium", "disk", "--filename", "disktwo.vmdk", "--format", "vmdk", "--size", 1024 * 10 ]
            #  vb.customize [ "storageattach", "myvm" , "--storagectl", "IDE Controller", "--port", "1", "--device", "0", "--type", "hdd", "--medium", "disktwo.vmdk"]
            #end
            end
        end
    end
end
~~~

## Mise en place de Docker Swarm

### 1. Setup

🌞 Utilisez des commandes docker afin de créer votre cluster Swarm
~~~
[vagrant@vm1 ~]$ sudo docker swarm init --advertise-addr 192.168.100.10
Swarm initialized: current node (zchp0q4qizzhe47tpjoz327tn) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-2snf3qrfdi8da64rx9ikq85r1dxj8mg71bcxe7dxzakueqjd6f-7drmd9rhi58cv78df4cqo73yb 192.168.100.10:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
~~~



docker swarm join pour qu'un noeud rejoigne un swarm existant :
~~~
[vagrant@vm2 ~]$ sudo docker swarm join --token SWMTKN-1-2snf3qrfdi8da64rx9ikq85r1dxj8mg71bcxe7dxzakueqjd6f-7drmd9rhi58cv78df4cqo73yb 192.168.100.10:2377
This node joined a swarm as a worker.
~~~
~~~
[vagrant@vm3 ~]$ sudo docker swarm join --token SWMTKN-1-2snf3qrfdi8da64rx9ikq85r1dxj8mg71bcxe7dxzakueqjd6f-7drmd9rhi58cv78df4cqo73yb 192.168.100.10:2377
This node joined a swarm as a worker.
~~~

vos 3 machines doivent être des managers Swarm

Join as manager 
VM2 :
~~~
[vagrant@vm2 ~]$ sudo docker swarm leave
Node left the swarm.
[vagrant@vm2 ~]$ sudo docker swarm join --token SWMTKN-1-2snf3qrfdi8da64rx9ikq85r1dxj8mg71bcxe7dxzakueqjd6f-3a5n7sy6jixi3zwlfi1qzecdy 192.168.100.10:2377
This node joined a swarm as a manager.
~~~

VM3 :
~~~
[vagrant@vm3 ~]$ sudo docker swarm leave
Node left the swarm.
[vagrant@vm3 ~]$ sudo docker swarm join --token SWMTKN-1-2snf3qrfdi8da64rx9ikq85r1dxj8mg71bcxe7dxzakueqjd6f-3a5n7sy6jixi3zwlfi1qzecdy 192.168.100.10:2377
This node joined a swarm as a manager.
~~~



