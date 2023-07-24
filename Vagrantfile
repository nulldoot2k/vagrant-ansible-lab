NodeScale = 50
IP_IW = "172.16.1."
NodeCount = 2
NodeControllCount = 1

def setup_dns(node)
  node.vm.provision "setup-hosts", :type => "shell", :path => "setup/setup-hosts.sh" do |s|
    s.args = ["eth1", node.vm.hostname]
  end
  node.vm.provision "setup-dns", :type => "shell", :path => "setup/setup-dns.sh"
end

Vagrant.configure("2") do |config|
  config.vm.boot_timeout = 300
  config.vm.synced_folder ".", "/vagrant"
  config.vm.provision "shell", path: "setup/bootstrap.sh"
  config.vm.provision "shell" do |s|
    ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
    s.inline = <<-SHELL
      echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
      echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
    SHELL
  end
  
  # Node
  (1..NodeCount).each do |i|
    config.vm.define "node#{i}" do |node|
      node.vm.box = "bento/ubuntu-20.04"
      node.vm.hostname = "node#{i}"
      node.vm.network :private_network, ip: IP_IW + "#{NodeScale + i}"
      node.vm.network :forwarded_port, guest: 22, host: "#{2200 + i}", id: "ssh"
      node.vm.provider :virtualbox do |vb|
        vb.memory = 512
        vb.cpus = 1
      end
      setup_dns node
    end
  end

  # Control
  (1..NodeControllCount).each do |i|
    config.vm.define "control" do |control|
      control.vm.box = "bento/ubuntu-20.04"
      control.vm.hostname = "control"
      control.vm.network :private_network, ip: "172.16.1.50"
      control.vm.network :forwarded_port, guest: 22, host: "2200", id: "ssh"
      control.vm.provider :virtualbox do |vb|
        vb.memory = 512
        vb.cpus = 1
      end
      setup_dns control
      control.vm.provision "ansible" do |ansible|
        ansible.verbose = "v"
        ansible.playbook = "ansible/playbook.yml"
        ansible.limit = "nodes"
        ansible.inventory_path = "ansible/hosts"
      end
    end
  end

end
