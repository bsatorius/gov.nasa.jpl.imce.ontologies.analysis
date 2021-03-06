#!/usr/bin/env groovy

pipeline {
    agent any

    parameters {
        string(name: 'ONT_METADATA', defaultValue: '../../exporter-results/exportedOMFMetadata.owl', description: 'Ontology metadata file location. Should probably be moved into the build.sbt script and derived from input.')
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out project from SCM..."

                checkout scm
            }
        }

        stage('Setup') {
            steps {
                echo "Setting up environment..."

                sh "cd workflow; . env.sh; /usr/bin/make clean"

                sh "${tool name: 'default-sbt', type: 'org.jvnet.hudson.plugins.SbtPluginBuilder$SbtInstallation'}/bin/sbt setupTools setupExportResults"
                sh ". workflow/env.sh"
            }
        }

        stage('Build') {
            steps {
                sh "${tool name: 'default-sbt', type: 'org.jvnet.hudson.plugins.SbtPluginBuilder$SbtInstallation'}/bin/sbt compile test:compile"
                //archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }

        stage('Validate-Ontologies') {
            environment {
                METADATA = "${params.ONT_METADATA}"
            }
            steps {
                echo "Validating ontologies..."
                sh "echo 'METADATA plain: \$METADATA'"
                echo "METADATA env: ${env.METADATA}"

                sh "cd workflow; . env.sh; echo \$JENA_PORT"

                sh "cd workflow; . env.sh; /usr/bin/make \$WORKFLOW/Makefile"
                sh "cd workflow; . env.sh; /usr/bin/make location-mapping"
                sh "cd workflow; . env.sh; /usr/bin/make validate-roots"

                junit '**/target/workflow/tests/**/*.xml'
            }
        }
    }
}
