default: \
  ocp_install \
  cnv_install \
  olm
  #osp_operators_install \
  #osp_tests_run

all: default

hosts:
	echo -e "localhost ansible_connection=local\n\n[convergence_base]\nlocalhost" > $@

local-defaults.yaml:
	echo -e "vars:" > $@

local-deps:
	ansible-galaxy install -r requirements.yml

ocp_install:
	$(MAKE) host_prep
	$(MAKE) dev_scripts
	$(MAKE) post_install

host_prep: hosts local-deps local-defaults.yaml
	ANSIBLE_FORCE_COLOR=true ansible-playbook -i hosts -e @local-defaults.yaml \
	host_prep.yaml \
	ocp_dev_scripts_prep.yaml

dev_scripts: hosts local-defaults.yaml
	ANSIBLE_FORCE_COLOR=true ansible-playbook -i hosts -e @local-defaults.yaml \
	ocp_dev_scripts.yaml

post_install: hosts local-defaults.yaml
	ANSIBLE_FORCE_COLOR=true ansible-playbook \
	-i hosts -e @local-defaults.yaml \
	local_oc_client.yaml \
	ocp_default_storageclass.yaml \
	ocp_image_registry.yaml \
	ocp_vm_setup_extra_nics.yaml

cnv_install: hosts local-defaults.yaml
	ANSIBLE_FORCE_COLOR=true ansible-playbook \
	-i hosts -e @local-defaults.yaml \
	ocp_cnv.yaml

ctlplane: hosts local-defaults.yaml
	# NOTE: requires 'make olm'
	ANSIBLE_FORCE_COLOR=true ansible-playbook \
	-i hosts -e @local-defaults.yaml \
	install_ctlplane.yaml

osp_content: hosts local-defaults.yaml
	ANSIBLE_FORCE_COLOR=true ansible-playbook \
	-i hosts -e @local-defaults.yaml \
	osp_content.yaml

osp_tests_run: hosts local-defaults.yaml
	ANSIBLE_FORCE_COLOR=true ansible-playbook \
	-i hosts -e @local-defaults.yaml \
	osp_tempest_ocp.yaml

osp-director-operator: hosts local-defaults.yaml
	# NOTE: requires 'make olm'
	ANSIBLE_FORCE_COLOR=true ansible-playbook \
	-i hosts -e @local-defaults.yaml \
	install_namespace.yaml

olm: hosts local-defaults.yaml
	ANSIBLE_FORCE_COLOR=true ansible-playbook \
	-v -i hosts -e @local-defaults.yaml \
	install_namespace.yaml \
	olm.yaml

openstackclient: hosts local-defaults.yaml
	ANSIBLE_FORCE_COLOR=true ansible-playbook \
	-v -i hosts -e @local-defaults.yaml \
	install_openstackclient.yaml \

olm_cleanup: hosts local-defaults.yaml
	ANSIBLE_FORCE_COLOR=true ansible-playbook \
	-v -i hosts -e @local-defaults.yaml \
	olm_cleanup.yaml

nfs_cleanup: hosts
	ANSIBLE_FORCE_COLOR=true ansible -i hosts convergence_base \
	--become -m shell -a \
	"set -x; \
	 rm -f /etc/exports.d/host-nfs-storageclass.exports; \
	 exportfs -ra; \
	 umount -d /home/nfs/pv-*; \
	 rm -rf /home/nfs; \
	 temp=$$(mktemp); \
	 grep -v "^/home/nfs/data/pv-" /etc/fstab > \$$temp; \
	 cat \$$temp > /etc/fstab; \
	 rm \$$temp"

destroy_ocp: hosts
	ANSIBLE_FORCE_COLOR=true ansible -i hosts convergence_base \
	--become --become-user ocp \
	-m shell -a "cd /home/ocp/dev-scripts && make clean"
	$(MAKE) nfs_cleanup
