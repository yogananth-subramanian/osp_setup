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
        string(name: 'beaker_user', defaultValue: 'ysubrama', description: 'Beaker username')
    }
    stages {
        stage('Install Baremetal') {
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
    }
}
