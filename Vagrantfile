# vi: set ft=ruby :

BASE_IP = "192.168.34"
VM_NAME = 'ctf'

# Pick one of the flavors below
BOXES = [
# =========================
# Headless servers:
# =========================
    # Ubuntu (x64)
    # -------------------------
    #{:box => "puppetlabs/ubuntu-16.04-64-nocm", :name => "ubuntu16"},
    #{:box => "puppetlabs/ubuntu-14.04-64-nocm", :name => "ubuntu14"},
    #{:box => "puppetlabs/ubuntu-12.04-64-nocm", :name => "ubuntu12"},

    # Ubuntu (x32)
    # -------------------------
    #{:box => "puppetlabs/ubuntu-16.04-32-nocm", :name => "ubuntu16"},
    #{:box => "puppetlabs/ubuntu-14.04-32-nocm", :name => "ubuntu14"},
    #{:box => "puppetlabs/ubuntu-12.04-32-nocm", :name => "ubuntu12"},

    # Centos (x64)
    # -------------------------
    {:box => "puppetlabs/centos-7.0-64-nocm", :name => "centos7"},
    #{:box => "puppetlabs/centos-6.6-64-nocm", :name => "centos6"},
    #{:box => "puppetlabs/centos-5.11-64-nocm", :name => "centos5"},

    # Debian (x64)
    # -------------------------
    #{:box => "debian/jessie64", :name => "debian8"},
    #{:box => "puppetlabs/debian-7.8-64-nocm", :name => "debian7"},
    #{:box => "puppetlabs/debian-6.0.10-64-nocm", :name => "debian6"},

    # Fedora (x64)
    # -------------------------
    #{:box => "box-cutter/fedora22", :name => "fedora22"},
    #{:box => "boxcutter/fedora21", :name => "fedora21"},

# =========================
# GUI Environments (Untested... There may be dragons here.)
# =========================
    # Mint (x64)
    # -------------------------
    #{:box => "npalm/mint17-amd64-cinnamon", :name => "mint17", :type => :gui},
]

#===============================================================================
# Box Types (Do not modify unless you know what you are doing)
#===============================================================================
VM_TYPES = [
    "#{VM_NAME}-sm",
    "#{VM_NAME}-md",
    "#{VM_NAME}-lg",
    "#{VM_NAME}-xl",
]

#===============================================================================
# Helper Functions
#===============================================================================

def create_vm(config, box, name)
    cpus = 1
    memory = 512
    boxname = box[:name]
    box = box[:box]
    type = box[:type] || :headless
    hostname = name.to_s
    scripts = []

    case name.to_s
    when /.*-sm/
        cpus = 1
        memory = 512
    when /.*-md/
        cpus = 1
        memory = 1024
    when /.*-lg/
        cpus = 2
        memory = 2048
    when /.*-xl/
        cpus = 4
        memory = 4096
    end

    hostname = "#{name}.vm"

    ip_addr = get_vm_ip(name)

    name = "#{name}-#{boxname}.vm"
    vboxname = "#{name}#{Dir.pwd}".gsub('/', '-')


    scripts.push get_ip_addresses()
    scripts.push install_hacking_packages(boxname)
    if script = get_platform_scripts(boxname)
        scripts.push script
    end


    config.vm.define name.to_sym do |vbox|
        vbox.vm.box = box
        vbox.vm.hostname = hostname
        vbox.vm.network "private_network", ip: ip_addr
        vbox.vm.provider "virtualbox" do |v|
            v.cpus = cpus
            v.name = vboxname
            v.memory = memory
        end
        scripts.each do |script|
            vbox.vm.provision "shell", inline: script
        end
    end
end

def get_vm_ip(hostname)
    case hostname.to_s

    when /.*-sm/
        ip = 0
    when /.*-md/
        ip = 1
    when /.*-lg/
        ip = 2
    when /.*-xl/
        ip = 3
    end

    "#{BASE_IP}.#{ip + 10}"
end

def get_ip_addresses()
    VM_TYPES.map do |vm|
        "echo '#{get_vm_ip(vm)} #{vm}.vm' >> /etc/hosts"
    end.join("\n")
end


def get_platform_scripts(boxname)
    case boxname
    when /fedora.*|centos7.*|el.*/
        <<-SCRIPT
        sudo service firewalld stop
        SCRIPT
    when /centos6.*/
        <<-SCRIPT
        sudo service iptables stop
        SCRIPT
    else
        nil
    end
end

def install_hacking_packages(boxname)
    global_packages = ['bash', 'vim']
    case boxname
    when /fedora.*|centos.*|el.*/
        packages = ['glibc.i686', 'gdb', 'tree']
        packages += global_packages
        install_commands = packages.map do |pkg|
            "yum install #{pkg} -y"
        end
        install_commands.join("\n")
    else
        ''
    end
end

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    BOXES.each do |box|
        VM_TYPES.each do |type|
            create_vm(config, box, type)
        end
    end
    config.vm.synced_folder Dir.pwd(), "/vagrant", type: "nfs"
end
