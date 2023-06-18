pipeline {
    agent any
    //tools {
    //    "org.jenkinsci.plugins.terraform.TerraformInstallation" "terraform"
    //}
    parameters {
        string(name: 'WORKSPACE', defaultValue: 'DEV', description:'workspace to use in Terraform')
    }

    environment {
        TF_HOME = tool('terraform')
        TF_INPUT = "0"
        TF_IN_AUTOMATION = "TRUE"
        TF_LOG = "WARN"
        AWS_ACCESS_KEY_ID = credentials('aws_access_key')
        AWS_SECRET_ACCESS_KEY = credentials('aws_secret_access_key')
        PATH = "$TF_HOME:$PATH"
    }

    stages {
        stage('Init'){
            steps {
                //dir('tf/networking/'){
                    sh 'terraform --version'
                    sh "terraform init"
                //}
            }
        }
        stage('Validate'){
            steps {
                //dir('tf/networking/'){
                    sh 'terraform validate'
                //}
            }
        }
        stage('Plan'){
            steps {
                //dir('tf/networking/'){
                    script {
                        try {
                           sh "terraform workspace new ${params.WORKSPACE}"
                        } catch (err) {
                            sh "terraform workspace select ${params.WORKSPACE}"
                        }
                        sh "terraform plan -out terraform-networking.tfplan;echo \$? > status"
                        stash name: "terraform-networking-plan", includes: "terraform-networking.tfplan"
                    }
                //}
            }
        }
        stage('Apply'){
            steps {
                script{
                    def apply = false
                    try {
                        input message: 'confirm apply', ok: 'Apply Config'
                        apply = true
                    } catch (err) {
                        apply = false
                        //dir('tf/networking/'){
                            sh "terraform destroy -auto-approve"
                        //}
                        currentBuild.result = 'UNSTABLE'
                    }
                    if(apply){
                        //dir('tf/networking/'){
                            unstash "terraform-networking-plan"
                            sh 'terraform apply terraform-networking.tfplan'
                        //}
                    }
                }
            }
        }
    }
}
