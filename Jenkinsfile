def osp_rhel_list = [
    '10': '7.7',
    '13': '7.8',
    '14': '7.7',
    '15': '8.2',
    '16': '8.1',
    '16.1': '8.2',
    '16.1.1': '8.2'
]

def OS_REL = osp_rhel_list[params.release]

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
        choice(name: 'deployment', choices: ['virtual', 'hybrid'], description: 'Pick something')
        string(name: 'temp_git', defaultValue: 'https://github.com/yogananth-subramanian/tht-dpdk.git', description: 'git repo hosting the THT template')
        string(name: 'temp_path', defaultValue: 'osp16_ref', description: 'path to THT templates in above repo')
        string(name: 'build_update', defaultValue: 'passed_phase2', description: 'passed_phase2,passed_phase1')
    }
    stages {
        stage('update fix') {
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
