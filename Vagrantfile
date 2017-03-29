# -*- mode: ruby -*-
# vi: set ft=ruby :

class Vm
  def initialize(
    config:,
    box:,
    name:,
    cpus: 1,
    memory: 1024,
    gui: false,
    vram: 16,
    accelerate3d: false,
    accelerate2dvideo: false,
    shared_folders: {},
    nfs_version: 4,
    nfs_protocol: 'udp',
    ip: ,
    tcp_forwarded_ports: {},
    udp_forwarded_ports: {},
    provision_shell_paths: []
  )
    @config = config
    @box = box
    @name = name
    @cpus = cpus
    @memory = memory
    @gui = gui
    @vram = vram
    @accelerate3d = accelerate3d
    @accelerate2dvideo = accelerate2dvideo
    @shared_folders = shared_folders
    @nfs_version = nfs_version
    @nfs_protocol = nfs_protocol
    @ip = ip
    @tcp_forwarded_ports = tcp_forwarded_ports
    @udp_forwarded_ports = udp_forwarded_ports
    @provision_shell_paths = provision_shell_paths
  end

  def up()
    return self.configure()
  end

  def configure()

    if Vagrant.has_plugin?('vagrant-cachier')
      puts 'Configuring vagrant-cachier...'
      @config.cache.scope = :machine
      @config.cache.synced_folder_opts = {
        type: :nfs,
        mount_options: ['rw', 'tcp', 'nolock'],
      }
    end

    @config.vm.box = @box
    @config.vm.define @name do |c|
      c.vm.hostname = "#{@name}.local"

      @shared_folders.each_with_index do |(hfolder, gfolder), i|
        c.vm.synced_folder(
          hfolder.to_s,
          gfolder.to_s,
          id: "%s_share" % @name,
          nfs: true,
          mount_options: [
            'nolock',
            "vers=#{@nfs_version}",
            @nfs_protocol,
          ],
        )
      end

      c.vm.provider :virtualbox do |vb|
        vb.gui = @gui
        vb.cpus = @cpus
        vb.memory = @memory
        vb.customize [
          'modifyvm', :id,
          '--accelerate3d', @accelerate3d ? 'on': 'off'
        ]
        vb.customize [
          'modifyvm', :id,
          '--accelerate2dvideo', @accelerate2dvideo ? 'on': 'off'
        ]
        vb.customize ['modifyvm', :id, '--vram', @vram]
        vb.customize ['modifyvm', :id, '--nic1', 'nat']
        vb.customize ['modifyvm', :id, '--nictype1', 'virtio']
      end

      c.vm.network(
        :private_network,
        ip: @ip,
        nictype: 'virtio',
      )

      @tcp_forwarded_ports.each do |g, h|
        c.vm.network(
          :forwarded_port,
          guest: g,
          host: h,
          host_ip: @ip,
          protocol: 'tcp',
          auto_correct: true,
        )
      end

      @udp_forwarded_ports.each do |g, h|
        c.vm.network(
          :forwarded_port,
          guest: g,
          host: h,
          host_ip: @ip,
          protocol: 'udp',
          auto_correct: true,
        )
      end

      @provision_shell_paths.each do |f|
        c.vm.provision(
          :shell,
          path: f,
        )
      end
    end
  end
end

Vagrant.configure(2) do |config|
  #ssh_pubkey = File.readlines(
  #  File.expand_path('~/.ssh/id_rsa.pub')
  #).first.strip

  Vm.new(
    config: config,
    box: 'ubuntu/xenial64',
    name: 'flocker-client',
    ip: "172.20.54.50",
    provision_shell_paths: [
      'flocker-client.provision.sh',
    ]
  ).up()

  (1..3).each do |i|
    Vm.new(
      config: config,
      box: 'ubuntu/xenial64',
      name: 'flocker-node-%02d' % i,
      ip: "172.20.54.#{i+100}",
      provision_shell_paths: [
        'flocker-node.provision.sh',
      ]
    ).up()
  end

  #config.vm.provision "shell", inline: <<-SHELL
  #  set -e
#
#    # See https://bugs.launchpad.net/cloud-images/+bug/1621393
#    ln -svf /run/resolvconf/interface/enp0s3.dhclient /etc/resolv.conf
#
#    echo '#{ssh_pubkey}' | tee -a /home/ubuntu/.ssh/authorized_keys
#
#    apt-get update
#    apt-get install -y --no-install-recommends python
#    set +e
#  SHELL

end

