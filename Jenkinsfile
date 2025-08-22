pipeline {
  agent any

  parameters {
    choice(
      name: 'ACTION',
      choices: ['apply', 'destroy_budget', 'rmstate_budget'],
      description: 'apply = create VMSS + Budget; destroy_budget = delete only budget in Azure; rmstate_budget = remove budget from state only'
    )
  }

  environment {
    ARM_CLIENT_ID       = '370ae248-5a76-402d-a0bd-b6ac6c8c4dde'
    ARM_CLIENT_SECRET   = '1r68Q~Q6dSjN3uY6bcPYpq7hBi8AypV~Q-I28cgN'
    ARM_TENANT_ID       = '5e865d62-8ae0-4163-9923-646bf3b4ffa1'
    ARM_SUBSCRIPTION_ID = 'a01bd4dc-3157-43b6-a8f9-a9a3c40a0e8c'
    TF_VAR_subscription_id = '/subscriptions/a01bd4dc-3157-43b6-a8f9-a9a3c40a0e8c'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Terraform Init and Validate') {
      steps {
        sh '''
          terraform -chdir=terraform init -input=false
          terraform -chdir=terraform fmt -check -diff || true
          terraform -chdir=terraform validate
        '''
      }
    }

    stage('Execute Terraform') {
      steps {
        script {
          if (params.ACTION == 'apply') {
            sh '''
              terraform -chdir=terraform plan -out=tfplan
              terraform -chdir=terraform apply -auto-approve tfplan
            '''
          } else if (params.ACTION == 'destroy_budget') {
            sh '''
              terraform -chdir=terraform destroy -auto-approve -target=azurerm_consumption_budget_subscription.vmss_budget
            '''
          } else if (params.ACTION == 'rmstate_budget') {
            sh '''
              terraform -chdir=terraform state rm azurerm_consumption_budget_subscription.vmss_budget || true
            '''
          }
        }
      }
    }
  }
}

