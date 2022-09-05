# Curso-DevOps
#Atividade  Grafana/InfluxDB/Telegraf
Vagrant.configure("2") do |config|
  config.vm.define "ubuntu1" do |ubuntu1|
    ubuntu1.vm.box = "ubuntu/bionic64"
    ubuntu1.vm.network "public_network", bridge: "Realtek 8811CU Wireless LAN 802.11ac USB NIC", ip: "192.168.0.200" 
    ubuntu1.vm.provider "virtualbox" do |vb|
        vb.memory = 1024
        vb.cpus = 2
  vb.name = "ubuntu1"
    end
    $script=<<-SCRIPT
    apt update -y && apt upgrade -y
    wget -qO- https://repos.influxdata.com/influxdb.key | apt-key add - 
    echo "deb https://repos.influxdata.com/debian buster stable" | tee /etc/apt/sources.list.d/influxdb.list
    apt update -y && apt upgrade -y
    apt -y install telegraf -y
    systemctl start telegraf && systemctl enable telegraf && systemctl status telegraf
    SCRIPT
    ubuntu1.vm.provision "shell",inline:$script
  end
  config.vm.define "ubuntu2" do |ubuntu2|
    ubuntu2.vm.box = "ubuntu/bionic64"
    ubuntu2.vm.network "public_network", bridge: "Realtek 8811CU Wireless LAN 802.11ac USB NIC", ip: "192.168.0.201"
    ubuntu2.vm.network "forwarded_port", guest: 80, host: 8086
    ubuntu2.vm.provider "virtualbox" do |vb|
        vb.memory = 1024
        vb.cpus = 2
  vb.name = "ubuntu2"
    end
    $script=<<-SCRIPT
    apt update -y && apt upgrade -y
    curl -s https://repos.influxdata.com/influxdb2.key | gpg --import -
    wget https://dl.influxdata.com/influxdb/releases/influxdb2-2.0.9-linux-amd64.tar.gz.asc
    gpg --verify influxdb2-2.0.9-linux-amd64.tar.gz.asc influxdb2-2.0.9-linux-amd64.tar.gz
    echo "deb https://repos.influxdata.com/debian buster stable" | tee /etc/apt/sources.list.d/influxdb.list
    apt update -y && apt upgrade -y
    apt install influxdb -y
    systemctl start influxdb && systemctl enable influxdb && systemctl status influxdb
    SCRIPT
    ubuntu2.vm.provision "shell",inline:$script
  end
  
  config.vm.define "ubuntu3" do |ubuntu3|
    ubuntu3.vm.box = "ubuntu/bionic64"
    ubuntu3.vm.network "public_network", bridge: "Realtek 8811CU Wireless LAN 802.11ac USB NIC", ip: "192.168.0.202"
    ubuntu3.vm.network "forwarded_port", guest: 80, host: 3000
    ubuntu3.vm.provider "virtualbox" do |vb|
        vb.memory = 1024
        vb.cpus = 2
  vb.name = "ubuntu3"
    end
    $script=<<-SCRIPT
    sudo apt-get install -y apt-transport-https
    sudo apt-get install -y software-properties-common wget
    sudo wget -q -O /usr/share/keyrings/grafana.key https://packages.grafana.com/gpg.key
    echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
    sudo apt-get update -y
    sudo apt-get install grafana -y
    systemctl start grafana-server && systemctl enable grafana-server && systemctl status grafana-server
    SCRIPT
    ubuntu3.vm.provision "shell",inline:$script
  end
end
