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
            #if statement waits for the vms to be ready before running ansible provisioner
            if i == N
                node.vm.provision "ansible-node", type: "ansible" do |ansible|
                    ansible.playbook = "k8s_setup/k8s_node.yml"
                    #since ansible support parallism we're filtering the hosts in the dynamic inventory to apply the playbook against
                    #rather than passing "kuben0#[i]" which meant it provisioned one at a time since it is looping over 'i'
                    ansible.limit = "kuben*"

                    ansible.extra_vars = {
                        node_ip: "192.168.50.#{i + 10}",
                    }
                end
            end
        end
    end

    # config.vm.define "kubem00" do |argocd|
    #     argocd.vm.provision "ansible-master", type: "ansible" do |ansible|
    #         ansible.playbook = "deploy_argocd/main.yml"
    #         ansible.limit = "kubem00"
    #         ansible.extra_vars = {
    #             node_ip: "192.168.50.10",
    #         }
    #     end
    # end   

    #externalise provisioniners due to outside in ordering mentioned here https://developer.hashicorp.com/vagrant/docs/multi-machine
    
        # Add a step to run Ansible playbook on "kubem00" after all VMs are provisioned




    # (1..N).each do |i|    
    #     config.vm.provision "ansible-node", type: "ansible" do |node|
    #         node.playbook = "k8s_setup/k8s_node.yml"
    #         node.limit = "kuben0\#{i}"
    #         node.extra_vars = {
    #             node_ip: "192.168.50.#{i + 10}",
    #         }
    #     end
    # end

#     config.vm.provision "argocd", after: "ansible-node", type: "ansible" do |argocd|
#         argocd.playbook = "deploy_argocd/main.yml"
#         argocd.limit = "kubem00"
#         argocd.galaxy_role_file = "deploy_argocd/collections/requirements.yml"
#     end
end
