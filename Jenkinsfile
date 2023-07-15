def envList = ['dev-env', 'qa-env', 'qa1-env', 'ts-env', 'ts2-env', 'ts3-env', 'perf1-env', 'perf2-env']
def orgList = ['APIGEE_ORGANISATION_NAME']

// Parameters Separated with Separator
properties([
    parameters([
        [
          $class: 'ChoiceParameter',
          choiceType: 'PT_SINGLE_SELECT',
          description: 'Select the Organisation',
          name: 'ORGANISATION',
          script: [
              $class: 'GroovyScript',
              script: [classpath: [], sandbox: true, script: "return ${orgList.inspect()}"]
          ]
        ],
        [
            $class: 'ChoiceParameter',
            choiceType: 'PT_CHECKBOX',
            description: 'Check the Environment Name from the List',
            name: 'ENVIRONMENT',
            script: [
                $class: 'GroovyScript',
                script: [ classpath: [], sandbox: true, script: "return ${envList.inspect()}"
                ]
            ]
        ],
        // Separator
        separator(name: "ADD_PRODUCT", sectionHeader: "Add Product",
          separatorStyle: "border-width: 0",
          sectionHeaderStyle: """
            background-color: #7ea6d3;
            text-align: left;
            padding: 4px;
            color: #343434;
            font-size: 22px;
            font-weight: normal;
            text-transform: uppercase;
            font-family: 'Orienta', sans-serif;
            letter-spacing: 1px;
            font-style: italic;
          """
        ),
        [
          $class: 'DynamicReferenceParameter', 
          choiceType: 'ET_FORMATTED_HTML', 
          description: 'Mention the Product file name to create/update',
          name: 'PRODUCT_FILE_NAME', 
          omitValueField: true,
          referencedParameters: 'ENVIRONMENT',
          script: [
              $class: 'GroovyScript', 
              fallbackScript: [
                  classpath: [],
                  sandbox: true,
                  script: 
                      'return [\'Error message\']'
              ], 
              script: [
                  classpath: [], 
                  sandbox: true,
                  script: 
                      """ 
                          html=""
                          if (ENVIRONMENT.contains('')){
                              html="<input name='value' value='' class='setting-input' type='text'>"
                          }
                          else {
                            
                              html="Enter value in PRODUCT_NAME to enter the value"
                          }
                          return html
                      """
              ]
          ]
        ]
    ])
])

def selectedEnvs = params.ENVIRONMENT.split(',')
def selectedProducts = params.PRODUCT_FILE_NAME.split(',')

pipeline {
    agent any
    environment {
          GCLOUD_DIR = "$JENKINS_HOME/google-cloud-sdk/bin"
          APIGEE_CLI_DIR = "$HOME/.apigeecli/bin"
    }
    stages {
        stage('Installing Dependencies') {
      steps {
          sh '''#!/bin/bash
                echo "Checking for pre-installed dependencies..."
                echo ""
                if [ ! -d "$GCLOUD_DIR" ]; then
                    echo "Installing GCloud CLI..."
                    echo ""
                    cd $JENKINS_HOME
                    curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-412.0.0-linux-x86_64.tar.gz
                    tar -xf google-cloud-cli-412.0.0-linux-*.tar.gz
                    ./google-cloud-sdk/install.sh -q
                    source $JENKINS_HOME/google-cloud-sdk/completion.bash.inc
                    source $JENKINS_HOME/google-cloud-sdk/path.bash.inc
                else--
                    echo "GCloud CLI is already Installed!"
                    echo ""
                fi

                if [ ! -d "$APIGEE_CLI_DIR" ]; then
                    echo "Installing Apigee CLI..."
                    echo ""
                    curl -L https://raw.githubusercontent.com/apigee/apigeecli/main/downloadLatest.sh | sh -
                    
                else
                    echo "Apigee CLI is already Installed!"
                    echo ""
                fi
             '''
      }
    }
        // Logging into GCloud
        stage('Logging into Google Cloud and Get Access Token') {
          steps {
            script {
                withCredentials([file(credentialsId: '<gcp_service_account>', variable: 'GOOGLE_SERVICE_ACCOUNT_KEY')]) {
                sh '${GCLOUD_DIR}/gcloud auth activate-service-account --key-file ${GOOGLE_SERVICE_ACCOUNT_KEY}'
                env.TOKEN = sh([script: "${GCLOUD_DIR}/gcloud auth print-access-token", returnStdout: true ]).trim()
              }
            }
          }
        }
        stage('Add products') {
          steps {
            script {
          // Clone the repository
          checkout([$class: 'GitSCM', branches: [[name: "*/main"]], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'Automation-to-add-products-in-Apigee-using-Jenkins']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'github', url: 'https://github.com/knoldus/Automation-to-add-products-in-Apigee-using-Jenkins.git']]])

          for (envs in selectedEnvs) {
            for (products in selectedProducts) {
              // List the product 
              def product = "${products}"
              def command = "$APIGEE_CLI_DIR/apigeecli products list -o ${params.ORGANISATION} -t ${env.TOKEN}"
              // Run the command and capture the output
              def output = sh(script: command, returnStdout: true).trim()

              // Check if the product exists in the output
              if (output.contains(product)) {

                echo "Product $product exists."
                sh "$APIGEE_CLI_DIR/apigeecli products import -o ${params.ORGANISATION} -t ${env.TOKEN} -f ${WORKSPACE}/${product}.json --upsert"
              } else {
                echo "Product $product does not exist!"
                sh "$APIGEE_CLI_DIR/apigeecli products import -o ${params.ORGANISATION} -f ${WORKSPACE}/${product}.json"
              }
            }
          }
        }
      }
    }
  }
}
  
