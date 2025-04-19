@Library("jenkins-shared-library@main") _   // Import the Shared Libraryd

def secrets = [
  [path: 'secret/jenkins/ssh-private-keys/manish/ssh-private-key', engineVersion: 2, secretValues: [
    [envVar: 'SSH_PRIVATE_KEY', vaultKey: 'id_rsa']]],
]

def secrets2 = [
  [path: 'secret/jenkins/ansible/ansible-vault-pass', engineVersion: 2, secretValues: [
    [envVar: 'VAULT_PASS', vaultKey: 'password']]],
]

def configuration = [vaultUrl: 'https://vault.tritec.in',  vaultCredentialId: 'vault', engineVersion: 2]

pipeline {
   agent {
       kubernetes {
           yaml getAnsibleYaml()                   // Retrieve agent YAML manifest
           defaultContainer getDefaultContainer() // Retrieve default container configuration
       }
   }

   parameters {
      choice(name: 'PLAYBOOK', choices: ['site.yml', 'reset.yml'], description: 'select install or uninstall playbook to run')
   }

    stages {
        stage('Run Ansible to Install k3s cluster') {
            steps {
                container('ansible') {

                withVault([configuration: configuration, vaultSecrets: secrets]) {
                            script {
                                writeFile file: 'id_rsa', text: "$SSH_PRIVATE_KEY"
                            }
                withVault([configuration: configuration, vaultSecrets: secrets2]) {
                    script {
                        writeFile file: 'vault_pass.txt', text: "$VAULT_PASS"
                    }
                }
                    sh 'ansible --version'     
                    sh 'ansible-playbook --private-key=id_rsa -u manish -i inventory.yml playbooks/$PLAYBOOK --vault-password-file vault_pass.txt'
                    sh 'rm -f vault_pass.txt'  // Cleanup
                    sh 'rm -f id_rsa'  // Cleanup
                }
                }
            }
        
    }

    }
}