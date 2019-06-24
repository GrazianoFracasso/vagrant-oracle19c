VAGRANTFILE_API_VERSION = "2"

# define hostname
$ip = "192.168.50.4"
NAME = "oracle-19c-vagrant"

unless Vagrant.has_plugin?("vagrant-proxyconf")
  puts 'Installing vagrant-proxyconf Plugin...'
  system('vagrant plugin install vagrant-proxyconf')
end

# get host time zone for setting VM time zone, if possible
# can override in env section below
offset_sec = Time.now.gmt_offset
if (offset_sec % (60 * 60)) == 0
  offset_hr = ((offset_sec / 60) / 60)
  timezone_suffix = offset_hr >= 0 ? "-#{offset_hr.to_s}" : "+#{(-offset_hr).to_s}"
  SYSTEM_TIMEZONE = 'Etc/GMT' + timezone_suffix
else
  # if host time zone isn't an integer hour offset, fall back to UTC
  SYSTEM_TIMEZONE = 'UTC'
end

$reset_password = <<-SCRIPT
su - oracle
#echo "export ORACLE_BASE=/opt/oracle" >> /home/vagrant/.bash_profile
#echo "export ORACLE_HOME=/opt/oracle/product/19c/dbhome_1" >> /home/vagrant/.bash_profile
#echo "export ORACLE_SID=ORCLCDB" >> /home/vagrant/.bash_profile
#echo "export ORACLE_PDB=ORCLPDB1" >> /home/vagrant/.bash_profile
#echo "export PATH=$PATH:$ORACLE_HOME/bin" >> /home/vagrant/.bash_profile
/home/oracle/setPassword.sh oracle
SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ol7-latest"
  config.vm.box_url = "https://yum.oracle.com/boxes/oraclelinux/latest/ol7-latest.box"
  config.vm.define NAME
  
  config.vm.box_check_update = false
  
  # change memory size
  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.name = NAME
  end

  # VM hostname
  config.vm.hostname = NAME
  config.vm.network "private_network", ip: $ip
  # Oracle port forwarding
  config.vm.network "forwarded_port", guest: 1521, host: 1521
  config.vm.network "forwarded_port", guest: 5500, host: 5500
  #config.vm.synced_folder "/F/app", "/app", type: "virtualbox"
  # Provision everything on the first run
  config.vm.provision "shell", path: "scripts/install.sh", env:
    {
       "ORACLE_BASE"         => "/opt/oracle",
       "ORACLE_HOME"         => "/opt/oracle/product/19c/dbhome_1",
       "ORACLE_SID"          => "ORCLCDB",
       "ORACLE_PDB"          => "ORCLPDB1",
       "ORACLE_CHARACTERSET" => "AL32UTF8",
       "ORACLE_EDITION"      => "EE",
       "SYSTEM_TIMEZONE"     => SYSTEM_TIMEZONE
    }

  config.vm.provision "shell", inline: <<-SHELL
        #{$reset_password}
  SHELL

end
