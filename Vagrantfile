Vagrant.configure("2") do |config|
  config.vm.box = "generic/rhel8"
  config.vm.box_check_update = false

  config.vm.provider :libvirt do |libvirt|
    libvirt.memory = 4096
  end

  config.vm.provision "file", source: "common/root", destination: "/tmp/common_root"
  config.vm.provision "shell", inline: <<-SHELL
    cp -r /tmp/common_root/. /

    systemctl stop firewalld
    dnf install -y tree pcp-zeroconf
  SHELL

  config.vm.define "collector" do |node|
    node.vm.hostname = "collector.local"
    node.vm.network "public_network", dev: "virbr0", mode: "bridge", type: "bridge", ip: "192.168.122.100"

    node.vm.provision "file", source: "collector/root", destination: "/tmp/collector_root"
    node.vm.provision "shell", inline: <<-SHELL
      cp -r /tmp/collector_root/. /
      dnf install -y redis grafana grafana-pcp pcp-pmda-bpftrace
      systemctl enable --now grafana-server redis pmproxy
      systemctl restart pmlogger
    SHELL
  end

  (1..3).each do |i|
    config.vm.define "node#{i}" do |node|
      node.vm.hostname = "node#{i}.local"
      node.vm.network "public_network", dev: "virbr0", mode: "bridge", type: "bridge", ip: "192.168.122.#{100+i}"

      node.vm.provision "file", source: "node/root", destination: "/tmp/node_root"
      node.vm.provision "shell", inline: <<-SHELL
        cp -r /tmp/node_root/. /
        systemctl enable pmcd
        systemctl restart pmcd
      SHELL
    end
  end
end
