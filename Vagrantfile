Vagrant.require_version ">= 1.5.3"
unless Vagrant.has_plugin?("vagrant-vbguest")
  raise "Please install the vagrant-vbguest plugin by running `vagrant plugin install vagrant-vbguest`"
end

VAGRANTFILE_API_VERSION = "2"

MEMORY = 4096
CPU_COUNT = 2

$script = <<SCRIPT
if [ ! -d /edx/app/edx_ansible ]; then
    echo "Error: Base box is missing provisioning scripts." 1>&2
    exit 1
fi
OPENEDX_RELEASE=$1
export PYTHONUNBUFFERED=1
source /edx/app/edx_ansible/venvs/edx_ansible/bin/activate
cd /edx/app/edx_ansible/edx_ansible/playbooks

# Did we specify an openedx release?
if [ -n "$OPENEDX_RELEASE" ]; then
  EXTRA_VARS="-e edx_platform_version=$OPENEDX_RELEASE \
    -e certs_version=$OPENEDX_RELEASE \
    -e forum_version=$OPENEDX_RELEASE \
    -e xqueue_version=$OPENEDX_RELEASE \
  "
  CONFIG_VER=$OPENEDX_RELEASE
  # Need to ensure that the configuration repo is updated
  # The vagrant-devstack.yml playbook will also do this, but only
  # after loading the playbooks into memory.  If these are out of date,
  # this can cause problems (e.g. looking for templates that no longer exist).
  /edx/bin/update configuration $CONFIG_VER
else
  CONFIG_VER="master"
fi

ansible-playbook -i localhost, -c local vagrant-devstack.yml -e configuration_version=$CONFIG_VER $EXTRA_VARS

SCRIPT

edx_platform_mount_dir = "edx-platform"
themes_mount_dir = "themes"
forum_mount_dir = "cs_comments_service"
ora_mount_dir = "ora"
insights_mount_dir = "insights"
analytics_api_mount_dir = "analytics_api"
xblocks_dir = "xblocks"
edx_microsite_dir = "edx-microsite"

if ENV['VAGRANT_MOUNT_BASE']

  edx_platform_mount_dir = ENV['VAGRANT_MOUNT_BASE'] + "/" + edx_platform_mount_dir
  themes_mount_dir = ENV['VAGRANT_MOUNT_BASE'] + "/" + themes_mount_dir
  forum_mount_dir = ENV['VAGRANT_MOUNT_BASE'] + "/" + forum_mount_dir
  ora_mount_dir = ENV['VAGRANT_MOUNT_BASE'] + "/" + ora_mount_dir
  insights_mount_dir = ENV['VAGRANT_MOUNT_BASE'] + "/" + insights_mount_dir
  analytics_api_mount_dir = ENV['VAGRANT_MOUNT_BASE'] + "/" + analytics_api_mount_dir
  xblocks_dir = ENV['VAGRANT_MOUNT_BASE'] + "/" + xblocks_dir
  edx_microsite_dir  = ENV['VAGRANT_MOUNT_BASE'] + "/" +  edx_microsite_dir 
end

# map the name of the git branch that we use for a release
# to a name and a file path, which are used for retrieving
# a Vagrant box from the internet.
openedx_releases = {
  "openedx/rc/aspen-2014-09-10" => {
    :name => "aspen-devstack-rc1", :file => "20141009-aspen-devstack-rc1.box",
  },
  "aspen.1" => {
    :name => "aspen-devstack-1", :file => "20141028-aspen-devstack-1.box",
  },
  "named-release/aspen" => {
    :name => "aspen-devstack-1", :file => "20141028-aspen-devstack-1.box",
  },
  "named-release/birch.rc1" => {
    :name => "birch-devstack-rc1", :file => "20150203-birch-devstack-rc1.box",
  },
  "named-release/birch.rc2" => {
    :name => "birch-devstack-rc2", :file => "20150211-birch-devstack-rc2.box",
  },
  "named-release/birch.rc3" => {
    :name => "birch-devstack-rc3", :file => "20150213-birch-devstack-rc3.box",
  },
  "named-release/birch" => {
    :name => "birch-devstack", :file => "20150224-birch-devstack.box",
  },
  "named-release/birch.1" => {
    :name => "birch-devstack-1", :file => "birch-1-devstack.box",
  },
  "named-release/birch.2" => {
    :name => "birch-devstack-2", :file => "birch-2-devstack.box",
  },
  "named-release/cypress.rc1" => {
    :name => "cypress-devstack-rc1", :file => "20150714-cypress-devstack-rc1.box",
  },
  "named-release/cypress.rc2" => {
    :name => "cypress-devstack-rc2", :file => "20150717-cypress-devstack-rc2.box",
  },
  "named-release/cypress.rc3" => {
    :name => "cypress-devstack-rc3", :file => "20150724-cypress-devstack-rc3.box",
  },
  "named-release/cypress.rc4" => {
    :name => "cypress-devstack-rc4", :file => "cypress-rc4-devstack.box",
  },
  "named-release/cypress" => {
    :name => "cypress-devstack", :file => "cypress-devstack.box",
  },
}
openedx_releases.default = {
  :name => "cypress-devstack", :file => "cypress-devstack.box",
}
openedx_releases_vmware = {
  "named-release/birch" => {
    :name => "birch-devstack-vmware", :file => "20150610-birch-devstack-vmware.box",
  },
}
openedx_releases_vmware.default = {
  :name => "kifli-devstack-vmware", :file => "20140829-kifli-devstack-vmware.box",
}
rel = ENV['OPENEDX_RELEASE']

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Creates an edX devstack VM from an official release
  config.vm.box     = openedx_releases[rel][:name]
  config.vm.box_url = "http://files.edx.org/vagrant-images/#{openedx_releases[rel][:file]}"

  config.vm.network :private_network, ip: "192.168.33.10"
  config.vm.network :forwarded_port, guest: 8000, host: 8000
  config.vm.network :forwarded_port, guest: 8001, host: 8001
  config.vm.network :forwarded_port, guest: 18080, host: 18080
  config.vm.network :forwarded_port, guest: 8765, host: 8765
  config.vm.network :forwarded_port, guest: 9200, host: 9200
  config.vm.network :forwarded_port, guest: 8100, host: 8100  # Analytics Data API
  config.vm.network :forwarded_port, guest: 8110, host: 8110  # Insights
  config.vm.network :forwarded_port, guest: 50070, host: 50070  # HDFS Admin UI
  config.vm.network :forwarded_port, guest: 8088, host: 8088  # Hadoop Resource Manager
  config.ssh.insert_key = true

  config.vm.synced_folder  ".", "/vagrant", disabled: true

  # Enable X11 forwarding so we can interact with GUI applications
  if ENV['VAGRANT_X11']
    config.ssh.forward_x11 = true
  end

  if ENV['VAGRANT_USE_VBOXFS'] == 'true'
    config.vm.synced_folder "#{edx_platform_mount_dir}", "/edx/app/edxapp/edx-platform",
      create: true, owner: "edxapp", group: "www-data"
    config.vm.synced_folder "#{themes_mount_dir}", "/edx/app/edxapp/themes",
      create: true, owner: "edxapp", group: "www-data"
    config.vm.synced_folder "#{forum_mount_dir}", "/edx/app/forum/cs_comments_service",
      create: true, owner: "forum", group: "www-data"
    config.vm.synced_folder "#{ora_mount_dir}", "/edx/app/ora/ora",
      create: true, owner: "ora", group: "www-data"
    config.vm.synced_folder "#{insights_mount_dir}", "/edx/app/insights/edx_analytics_dashboard",
      create: true, owner: "insights", group: "www-data"
    config.vm.synced_folder "#{analytics_api_mount_dir}", "/edx/app/analytics_api/analytics_api",
      create: true, owner: "analytics_api", group: "www-data"
    config.vm.synced_folder "#{xblocks_dir}", "/edx/app/edxapp/xblocks",
      create: true, owner: "xblocks_dir", group: "www-data"
    config.vm.synced_folder "#{edx_microsite_dir }", "/edx/app/edxapp/edx-microsite",
      create: true, owner: "edx_microsite_dir ", group: "www-data"
  else
    config.vm.synced_folder "#{edx_platform_mount_dir}", "/edx/app/edxapp/edx-platform",
      create: true, nfs: true
    config.vm.synced_folder "#{themes_mount_dir}", "/edx/app/edxapp/themes",
      create: true, nfs: true
    config.vm.synced_folder "#{forum_mount_dir}", "/edx/app/forum/cs_comments_service",
      create: true, nfs: true
    config.vm.synced_folder "#{ora_mount_dir}", "/edx/app/ora/ora",
      create: true, nfs: true
    config.vm.synced_folder "#{insights_mount_dir}", "/edx/app/insights/edx_analytics_dashboard",
      create: true, nfs: true
    config.vm.synced_folder "#{analytics_api_mount_dir}", "/edx/app/analytics_api/analytics_api",
      create: true, nfs: true
    config.vm.synced_folder "#{xblocks_dir}", "/edx/app/edxapp/xblocks",
      create: true, nfs: true
     config.vm.synced_folder "#{edx_microsite_dir }", "/edx/app/edxapp/edx-microsite",
      create: true, nfs: true
  end

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", MEMORY.to_s]
    vb.customize ["modifyvm", :id, "--cpus", CPU_COUNT.to_s]

    # Allow DNS to work for Ubuntu 12.10 host
    # http://askubuntu.com/questions/238040/how-do-i-fix-name-service-for-vagrant-client
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
  end

  ["vmware_fusion", "vmware_workstation"].each do |vmware_provider|
    config.vm.provider vmware_provider do |v, override|
      override.vm.box     = openedx_releases_vmware[rel][:name]
      override.vm.box_url = "http://files.edx.org/vagrant-images/#{openedx_releases_vmware[rel][:file]}"
      v.vmx["memsize"] = MEMORY.to_s
      v.vmx["numvcpus"] = CPU_COUNT.to_s
    end
  end

  # Use vagrant-vbguest plugin to make sure Guest Additions are in sync
  config.vbguest.auto_reboot = true
  config.vbguest.auto_update = true

  # Assume that the base box has the edx_ansible role installed
  # We can then tell the Vagrant instance to update itself.
  config.vm.provision "shell", inline: $script, args: rel
end
