$msg = <<MSG
------------------------------------------------------
Nothing to say.
------------------------------------------------------
MSG

servers=[
  {
    :hostname      => "elasticsearchslave2",
    :box           => "debian/bullseye64",
    :ram           => 2048,
    :serverid      => 'slave',
    :ip            => '172.28.128.16',
  },
  {
    :hostname      => "elasticsearchmaster",
    :box           => "debian/bullseye64",
    :ram           => 4096,
    :serverid      => 'master',
    :ip            => '172.28.128.15',
    :redis_port    => 8001,
    :sentinel_port => 28001,
    :haproxy_port  => 38001,
  },
#  {
#    :hostname      => "elasticsearchslave3",
#    :box           => "debian/bullseye64",
#    :ram           => 2048,
#    :serverid      => 'slave',
#    :ip            => '172.28.128.17',
#  },
]

Vagrant.configure("2") do |config|

  servers.each do |machine|
    config.vm.define machine[:hostname] do |node|
      node.vm.box      = machine[:box]
      node.vm.hostname = machine[:hostname]
      node.vm.network "private_network", ip: machine[:ip]
      node.vm.provider "virtualbox" do |vb|
        vb.customize ["modifyvm", :id, "--memory", machine[:ram]]
      end

      node.vm.provision 'shell', inline: <<-SHELL
        /sbin/sysctl -w vm.swappiness=0
        /sbin/sysctl -w vm.overcommit_memory=1
        /sbin/sysctl -w net.core.somaxconn=65535
        echo never > /sys/kernel/mm/transparent_hugepage/enabled
        timedatectl set-ntp true
        timedatectl set-timezone "Africa/Johannesburg"
        timedatectl


        wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
        wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
        echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
        apt-get update
        export DEBIAN_FRONTEND=noninteractive
        DEBIAN_FRONTEND=noninteractive apt-get --no-install-suggests --no-install-recommends \
        install \
        chrony curl wget default-jdk zip python3-pip\
        apt-transport-https \
        gnupg2 \
        net-tools \
        gpg \
        vim \
        elasticsearch \
        filebeat metricbeat 

        systemctl enable elasticsearch
        systemctl start elasticsearch
        apt-get clean
      SHELL

      #if machine[:serverid] == "slave"
      #  node.vm.provision 'shell', inline: <<-SHELL
      #  SHELL
      #end

      if machine[:serverid] == "master"
        node.vm.provision 'shell', inline: <<-SHELL
          export DEBIAN_FRONTEND=noninteractive
          DEBIAN_FRONTEND=noninteractive apt-get --no-install-suggests --no-install-recommends install \
          logstash \
          kibana -y
          usermod -a -G adm logstash
          echo 'server.host: "0.0.0.0"' >> /etc/kibana/kibana.yml
          systemctl enable kibana
          systemctl restart kibana
          systemctl restart logstash
        SHELL
      end
    end
  end
end
