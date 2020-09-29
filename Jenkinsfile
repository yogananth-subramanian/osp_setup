def branches = [:]
def img_list = [
    '8.2':'http://download.devel.redhat.com/rhel-8/rel-eng/RHEL-8/latest-RHEL-8.2.0/compose/BaseOS/x86_64/images/rhel-guest-image-8.2-290.x86_64.qcow2',
    '8.1':'http://download.devel.redhat.com/rhel-8/rel-eng/RHEL-8/latest-RHEL-8.1.0/compose/BaseOS/x86_64/images/rhel-guest-image-8.1-263.x86_64.qcow2',
    '8.0':'http://download.devel.redhat.com/rhel-8/rel-eng/RHEL-8/latest-RHEL-8.0.0/compose/BaseOS/x86_64/images/rhel-guest-image-8.0-1854.x86_64.qcow2',
    '7.9':'http://download.devel.redhat.com/rhel-7/rel-eng/RHEL-7/latest-RHEL-7.9/compose/Server/x86_64/images/rhel-guest-image-7.9-9.x86_64.qcow2',
    '7.8':'http://download.devel.redhat.com/rhel-7/rel-eng/RHEL-7/latest-RHEL-7.8/compose/Server/x86_64/images/rhel-guest-image-7.8-41.x86_64.qcow2',
    '7.7':'http://download.devel.redhat.com/rhel-7/rel-eng/RHEL-7/latest-RHEL-7.7/compose/Server/x86_64/images/rhel-guest-image-7.7-261.x86_64.qcow2',
    '7.6':'http://download.devel.redhat.com/rhel-7/rel-eng/RHEL-7/latest-RHEL-7.6/compose/Server/x86_64/images/rhel-guest-image-7.6-210.x86_64.qcow2',
    '7.5':'http://download.devel.redhat.com/rhel-7/rel-eng/RHEL-7/RHEL-7.5-RC-1.3/compose/Server/x86_64/images/rhel-guest-image-7.5-146.x86_64.qcow2',
    '7.4':'http://download.devel.redhat.com/rhel-7/rel-eng/RHEL-7/RHEL-7.4-RC-1.2/compose/Server/x86_64/images/rhel-guest-image-7.4-191.x86_64.qcow2',
    '7.3':'http://download.devel.redhat.com/pub/rhel/released/RHEL-7/7.3/Server/x86_64/images/rhel-guest-image-7.3-33.x86_64.qcow2',
]
def osp_rhel_list = [
    '10': '7.7',
    '13': '7.8',
    '14': '7.7',
    '15': '8.2',
    '16': '8.1',
    '16.1': '8.2',
    '16.1.1': '8.2'
]
def net_config = [
    'rhos-nfv-10.lab.eng.rdu2.redhat.com': ' --topology-nodes undercloud:1,controller:1 --topology-network 4_nets_3_bridges_hybrid -e override.networks.net1.nic=em4 -e override.networks.net2.nic=em1 -e override.networks.net3.nic=em2',
    'dell-r640-oss-13.lab.eng.brq.redhat.com': ' --topology-nodes undercloud:1,controller:1 --topology-network 4_nets_3_bridges_hybrid -e override.networks.net1.nic=em2 -e override.networks.net2.nic=em3 -e override.networks.net3.nic=em4'
]
def IR_VIRSH_IMAGE = img_list[osp_rhel_list[params.release]]
def OS_REL = osp_rhel_list[params.release]
def IR_NET_CONFIG = net_config[params.server]
def stagelst='baremetal,undercloud,overcloud,update-fix,update-undercloud,update-overcloud'
def excludelst=[]
def excludestr="${params.exclude}"
for (i in excludestr.split(',')){
  for (j in stagelst.split(',')){
  if ( j.matches(i) ){
    excludelst.add(j)
  
  }
 }
}
pipeline {
    agent {
        node {
            label 'local'
            customWorkspace "${JENKINS_HOME}/workspace/${JOB_NAME}/${BUILD_NUMBER}"
        }
    }
    options {
        lock resource: "${params.server}"
    }
    parameters {
        choice(name: 'server', choices: ['rhos-nfv-10.lab.eng.rdu2.redhat.com', 'dell-r640-oss-13.lab.eng.brq.redhat.com'], description: 'Pick something')
        choice(name: 'release', choices: ['16.1', '16', '13'], description: 'Pick something')
        string(name: 'build', defaultValue: 'GA', description: 'passed_phase2,passed_phase1,GA')
        string(name: 'beaker_user', defaultValue: 'ysubrama', description: 'Beaker username')
        choice(name: 'deployment', choices: ['virtual', 'hybrid'], description: 'Pick something')
        string(name: 'instack_git', defaultValue: 'https://github.com/yogananth-subramanian/cluster-mgt.git', description: 'git repo hosting instackenv.json file')
        string(name: 'instack_path', defaultValue: 'templates/instack/dell-r640-oss-13-nodes.json', description: 'path to instackenv.json in above repo')
        string(name: 'temp_git', defaultValue: 'https://github.com/yogananth-subramanian/tht-dpdk.git', description: 'git repo hosting the THT template')
        string(name: 'temp_path', defaultValue: 'osp16_ref', description: 'path to THT templates in above repo')
        string(name: 'under_conf', defaultValue: 'osp16_ref/undercloud_hybrid.conf', description: 'path to undercloud.conf file in above THT template repo')
        string(name: 'overcloud_script', defaultValue: 'overcloud_deploy_regular.sh', description: 'path to overcloud_script in above THT template repo')
        string(name: 'build_update', defaultValue: 'passed_phase2', description: 'passed_phase2,passed_phase1')
        string(name: 'exclude', defaultValue: '.*', description: 'steps to exlude, takes on comma separated values such as baremetal,undercloud,overcloud,update-fix,update-undercloud,update-overcloud')
    }
    stages {
        stage('Install Baremetal') {
            when {
                expression { !('baremetal' in excludelst)  }
            }
            environment {
                SERVICE_CREDS = credentials("${params.beaker_user}")
            }
            steps {
                echo 'Building'
                echo 'Starting'
                  withCredentials([usernamePassword(credentialsId: "${params.beaker_user}", passwordVariable: 'beakerpass', usernameVariable: 'beakerusr')]) {
                        sh '''
                        git clone https://github.com/redhat-openstack/infrared.git;cd infrared/;virtualenv .venv;echo "export IR_HOME=`pwd`" >> .venv/bin/activate && source .venv/bin/activate; pip install -U pip;pip install .;infrared plugin add all
                        ir beaker --url='https://beaker.engineering.redhat.com' --beaker-user='ysubrama' --beaker-password="$beakerpass" --host-address="$server" --image='rhel-7.8'  --host-pubkey '/root/.ssh/id_rsa.pub' --host-privkey '/root//.ssh/id_rsa' --web-service 'rest' --host-password="$beakerpass"  --host-user='root'  --release='True'
                        ir beaker --url='https://beaker.engineering.redhat.com' --beaker-user='ysubrama' --beaker-password="$beakerpass" --host-address="$server" --image='rhel-7.8'  --host-pubkey '/root/.ssh/id_rsa.pub' --host-privkey '/root//.ssh/id_rsa' --web-service 'rest' --host-password="$beakerpass"  --host-user='root'
                        '''
                }
                echo 'Finish'
            }
        }
        stage('Configure Hypervisor') {
            when {
                expression { !('baremetal' in excludelst)  }
            }
            steps {
                echo 'Building'
                sh '''
                ssh  -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null root@${server}  'yum install -y git python3 libselinux-python3 python-virtualenv libselinux-python'
                keygen='yes|ssh-keygen -t rsa -q -f "$HOME/.ssh/id_rsa" -N ""; cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys '
                ssh  -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null root@${server} $keygen
                ssh  -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null root@${server} 'git clone https://github.com/redhat-openstack/infrared.git;cd infrared/;virtualenv .venv;echo "export IR_HOME=`pwd`" >> .venv/bin/activate && source .venv/bin/activate; pip install -U pip;pip install .;infrared plugin add all'
                '''
                echo 'Finish'
            }
        }
        stage('Setup Topology') {
            when {
                expression { !('undercloud' in excludelst)  }
            }
            environment {
                IR_VIRSH_IMAGE = "${IR_VIRSH_IMAGE}"
                IR_NET_CONFIG = "${IR_NET_CONFIG}"
            }
            steps {
                echo 'Building'
                 sh '''
                 net_ip=''
                 net_setup=''
                 cntrl_ip=''
                 ir_cli="infrared virsh -vv    --host-address $server  --host-key ~/.ssh/id_rsa     --image-url ${IR_VIRSH_IMAGE}  --host-memory-overcommit False --disk-pool /home/ -e override.controller.cpu=4 -e override.undercloud.cpu=4 "
                 if [ ${deployment} = 'virtual' ]
                 then 
                  IR_SETUP="cd infrared/;source .venv/bin/activate;$ir_cli --topology-nodes 'undercloud:1,controller:1,compute:1'   "
                 else  
                  if [ ! -z $temp_git ]
                    then
                    a=`basename $temp_git`
                    base_dir=${a%.git}
                    rm -rf $base_dir
                    git clone --depth=1 $temp_git
                    net_file=$(find $base_dir/$temp_path|grep -e network[-_]data)
                    cp $base_dir/$under_conf undercloud.conf
                    cp $base_dir/$temp_path/$overcloud_script overcloud_deploy.sh
                    a=`cat undercloud.conf |grep ^local_ip|awk 'BEGIN{FS=OFS="="} {print $2}'|sed "s/ //g"`
                    b=${a%/*}
                    under_ip=${b%.*}
                    if [ ! -z $under_ip ]
                      then
                      scp  -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null undercloud.conf root@${server}:/root/infrared/
                      scp  -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null overcloud_deploy.sh root@${server}:/root/infrared/

                      cntrl_ip=" -e  override.networks.net1.ip_address=$under_ip.150 "
                    fi
                    if [ ! -z $net_file ]
                      then
                      c=$(cat $net_file |awk 'BEGIN{RS="-";FS=""}{print}'|awk -v var="name_lower: external" 'BEGIN{RS="\\n\\n";FS="\\n"} $0~var{print }'|awk 'BEGIN{FS="";}/ip_subnet:/{print}'|awk 'BEGIN{FS=OFS=":"} {print $2}'|sed "s/'//g"|sed "s/ //g")
                      d=${c%/*}
                      net_ip=${d%.*}
                      net_setup=" -e  override.networks.net4.ip_address=$net_ip.1 -e  override.networks.net4.dhcp.range.start=$net_ip.2  -e  override.networks.net4.dhcp.range.end=$net_ip.100  -e  override.networks.net4.dhcp.subnet_cidr=$net_ip.0/24  -e  override.networks.net4.dhcp.subnet_gateway=$net_ip.1   -e  override.networks.net4.floating_ip.start=$net_ip.101 -e  override.networks.net4.floating_ip.end=$net_ip.151 "
                    fi
                    rm -rf $base_dir
                  fi
                  IR_SETUP="cd infrared/;source .venv/bin/activate;$ir_cli ${IR_NET_CONFIG} ${cntrl_ip} ${net_setup}"
                 fi
                 echo ${IR_SETUP}
                 ssh  -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null root@${server}  ${IR_SETUP}
                 '''
                echo 'Finish'
            }
        }
        stage('Setup Undercloud') {
            when {
                expression { !('undercloud' in excludelst)  }
            }
            steps {
                echo 'Testing'
                 sh '''
                 ir_cli="infrared tripleo-undercloud -vv --version ${release}  --build=${build} --images-task=rpm --images-update yes "
                 if [ ${deployment} = 'virtual' ]
                 then 
                  UN_SETUP="cd infrared/;source .venv/bin/activate;$ir_cli"
                 else
                  UN_SETUP="cd infrared/;source .venv/bin/activate;$ir_cli --config-file /root/infrared/undercloud.conf"
                 fi
                 echo ${UN_SETUP}
                 ssh  -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null root@${server}  ${UN_SETUP}
                 '''
            }
        }
        stage('Introspect Overcloud nodes') {
            when {
                expression { !('overcloud' in excludelst)  }
            }
            steps {
                echo 'Testing'
                 sh '''
                 ir_cli="infrared tripleo-overcloud -vv -o prepare_instack.yml     --version ${release}  --introspect=yes     --tagging=yes     --deploy=no "
                 if [ ${deployment} = 'virtual' ]
                 then 
                  OVER_SETUP="$ir_cli --deployment-files virt "
                 else
                    if [ ! -z $instack_git ]
                      then
                      a=`basename $instack_git`
                      base_dir=${a%.git}
                      rm -rf $base_dir
                      git clone --depth=1 $instack_git
                      cp $base_dir/$instack_path instack.json
                      rm -rf $base_dir
                      under_subnet=$(cat undercloud.conf |grep ^local_subnet|awk 'BEGIN{FS=OFS="="} {print $2}'|sed 's/ //'g)
                      a=`basename $temp_git`
                      dep_files=${a%.git}/$temp_path
                    fi
                    if [ -f instack.json  ]
                     then
                     echo "instack"
                     scp  -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null instack.json root@${server}:/root/infrared/
                    fi
                    if [ -z $under_subnet ]
                      then
                      under_subnet=br-ctlplane
                    fi
                  ssh  -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null root@${server} "cd infrared/;git clone $temp_git"
                  OVER_SETUP="$ir_cli  --deployment-files $dep_files     -e provison_virsh_network_name=br-ctlplane    --hybrid instack.json --vbmc-host undercloud "
                 fi
                 DEP_SETUP="cd infrared/;source .venv/bin/activate;${OVER_SETUP} "
                 echo ${DEP_SETUP}
                 ssh  -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null root@${server}  ${DEP_SETUP}
                 '''
            }
        }
        stage('Intall Overcloud') {
            when {
                expression { !('overcloud' in excludelst)  }
            }
            steps {
                echo 'Testing'
                 sh '''
                 net_back=' --network-ovn no --network-ovs yes --network-backend vxlan '
                 ir_cli="infrared tripleo-overcloud -vv -o prepare_instack.yml     --version ${release}  --introspect=no     --tagging=no  --tht-roles yes   --deploy=yes "
                 if [ ${deployment} = 'virtual' ]
                 then 
                  if [ ${release} = '16.1'  ]
                    then
                    net_back=' --network-backend=geneve '
                  fi
                  OVER_SETUP="$ir_cli --deployment-files virt $net_back"
                 else
                    if [ ! -z $instack_git ]
                      then
                      a=`basename $instack_git`
                      base_dir=${a%.git}
                      rm -rf $base_dir
                      git clone --depth=1 $instack_git
                      cp $base_dir/$instack_path instack.json
                      rm -rf $base_dir
                      under_subnet=$(cat undercloud.conf |grep ^local_subnet|awk 'BEGIN{FS=OFS="="} {print $2}'|sed 's/ //'g)
                      a=`basename $temp_git`
                      dep_files=${a%.git}/$temp_path
                    fi
                    scp  -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null overcloud_deploy.sh  root@${server}:/root/infrared/
                    if [ -z $under_subnet ]
                      then
                      under_subnet=br-ctlplane
                    fi                 
                  OVER_SETUP="$ir_cli  --deployment-files $dep_files     -e provison_virsh_network_name=$under_subnet    --hybrid instack.json  --overcloud-script  /root/infrared/overcloud_deploy.sh "
                 fi
                 OC_SETUP="cd infrared/;source .venv/bin/activate;${OVER_SETUP} "
                 echo ${OC_SETUP}
                 ssh  -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null root@${server}  ${OC_SETUP}
                 '''
            }
        }
        stage('update fix') {
            when {
                expression { !('update-fix' in excludelst)  }
            }
            steps {
                echo 'Testing'
                 sh '''
                 ir_cli="infrared plugin remove tripleo-upgrade;infrared plugin add --revision stable/train tripleo-upgrade;mkdir -p /root/infrared/plugins/tripleo-upgrade/infrared_plugin/roles/tripleo-upgrade;find /root/infrared/plugins/tripleo-upgrade -maxdepth 1 -mindepth 1 -not -name infrared_plugin -exec mv '{}' /root/infrared/plugins/tripleo-upgrade/infrared_plugin/roles/tripleo-upgrade \\; "
                 UN_SETUP="cd infrared/;source .venv/bin/activate;$ir_cli"
                 echo ${UN_SETUP}
                 ssh  -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null root@${server}  ${UN_SETUP}
                 '''
            }
        }
        stage('Update repo and containers') {
            when {
                expression { !('update-undercloud' in excludelst)  }
            }
            environment {
                OS_REL = "${OS_REL}"
            }
            steps {
                echo 'Testing'
                 sh '''
                 ir_cli="infrared tripleo-undercloud --update-undercloud yes --version ${release} --build ${build_update} --ansible-args='tags=upgrade_repos,undercloud_containers'  --osrelease ${OS_REL} "
                 UN_SETUP="cd infrared/;source .venv/bin/activate;$ir_cli"
                 echo ${UN_SETUP}
                 ssh  -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null root@${server}  ${UN_SETUP}
                 '''
            }
        }
        stage('Update undercloud') {
            when {
                expression { !('update-undercloud' in excludelst)  }
            }
            steps {
                echo 'Testing'
                 sh '''
                 ir_cli="infrared tripleo-upgrade --undercloud-update yes --overcloud-stack overcloud "
                 UN_SETUP="cd infrared/;source .venv/bin/activate;$ir_cli"
                 echo ${UN_SETUP}
                 ssh  -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null root@${server}  ${UN_SETUP}
                 '''
            }
        }
        stage('Update overcloud repo containers') {
            when {
                expression { !('update-overcloud' in excludelst)  }
            }
            environment {
                OS_REL = "${OS_REL}"
            }
            steps {
                echo 'Testing'
                 sh '''
                 ir_cli="infrared tripleo-overcloud --ocupdate True  --build ${build_update} --ansible-args='tags=update_collect_info,update_undercloud_validation,update_repos,update_prepare_containers;skip-tags=container-prepare-reboot' --osrelease ${OS_REL} "
                 if [ ${deployment} = 'virtual' ]
                 then 
                  OVER_SETUP="$ir_cli --deployment-files virt "
                 else
                  a=`basename $temp_git`
                  dep_files=${a%.git}/$temp_path
                  OVER_SETUP="$ir_cli  --deployment-files $dep_files "
                 fi
                 OC_SETUP="cd infrared/;source .venv/bin/activate;${OVER_SETUP} "
                 echo ${OC_SETUP}
                 ssh  -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null root@${server}  ${OC_SETUP}
                 '''
            }
        }
        stage('Update overcloud') {
            when {
                expression { !('update-overcloud' in excludelst)  }
            }
            steps {
                echo 'Testing'
                 sh '''
                 ir_cli="infrared tripleo-upgrade  --upgrade-floatingip-check no  --upgrade-workload yes --overcloud-update yes --overcloud-stack overcloud  "
                 if [ ${deployment} = 'virtual' ]
                 then 
                  OVER_SETUP="$ir_cli --deployment-files virt "
                 else  
                  a=`basename $temp_git`
                  dep_files=${a%.git}/$temp_path
                  OVER_SETUP="$ir_cli  --deployment-files $dep_files "
                 fi
                 OC_SETUP="cd infrared/;source .venv/bin/activate;${OVER_SETUP} "
                 echo ${OC_SETUP}
                 ssh  -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null root@${server}  ${OC_SETUP}
                 '''
            }
        }
        stage('Post update reboot and check') {
            when {
                expression { !('update-overcloud' in excludelst)  }
            }
            steps {
                echo 'Testing'
                 sh '''
                 net_back='--network-ovn no --network-ovs yes'
                 ir_cli="echo -e 'cleanup_services: []' > cleanup_services.yml;infrared tripleo-overcloud --postreboot True --postreboot_evacuate no --overcloud-stack overcloud -e @cleanup_services.yml "
                 if [ ${deployment} = 'virtual' ]
                 then
                  if [ ${release} = '16.1'  ]
                    then
                    net_back=' --network-ovn yes '
                  fi
                  OVER_SETUP="$ir_cli --deployment-files virt $net_back "
                 else  
                  a=`basename $temp_git`
                  dep_files=${a%.git}/$temp_path
                  OVER_SETUP="$ir_cli  --deployment-files $dep_files $net_back "
                 fi
                 OC_SETUP="cd infrared/;source .venv/bin/activate;${OVER_SETUP} "
                 echo ${OC_SETUP}
                 ssh  -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null root@${server}  ${OC_SETUP}
                 '''
            }
        }
        
    }
}
