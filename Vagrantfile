IMAGE_NAME = "bento/ubuntu-20.04"
N = 2

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false

    config.vm.provider "virtualbox" do |v|
        v.memory = 2048
        v.cpus = 2
    end
      
    config.vm.define "kubem00" do |master|
        master.vm.box = IMAGE_NAME
        master.vm.network "private_network", ip: "192.168.50.10"
        master.vm.hostname = "kubem00"
        master.vm.provision "ansible-master", type: "ansible" do |ansible|
            ansible.playbook = "k8s_setup/k8s_master.yml"
            ansible.limit = "kubem00"
            ansible.extra_vars = {
                node_ip: "192.168.50.10",
            }
        end
    end

    (1..N).each do |i|
        config.vm.define "kuben0#{i}" do |node|
            node.vm.box = IMAGE_NAME
            node.vm.network "private_network", ip: "192.168.50.#{i + 10}"
            node.vm.hostname = "kuben0#{i}"
            #this if statement waits for the vms to be ready before running ansible provisioner
            if i == N
                node.vm.provision "ansible-node", type: "ansible" do |ansible|
                    ansible.playbook = "k8s_setup/k8s_node.yml"
                    #since ansible support parallism we're filtering the hosts in the dynamic inventory to apply the playbook against
                    #rather than previously passing "kuben0#[i]" which meant it provisioned one at a time since it is looping over 'i'
                    ansible.limit = "kuben*"

                    ansible.extra_vars = {
                        node_ip: "192.168.50.#{i + 10}",
                    }
                end
            end
        end
    end
end
