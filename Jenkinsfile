#!groovy

docker.image('cloudbees/java-build-tools:0.0.6').inside {

    checkout scm

    def mavenSettingsFile = "${pwd()}/.m2/settings.xml"

    stage 'Compilation'
    wrap([$class: 'ConfigFileBuildWrapper',
        managedFiles: [[fileId: 'maven-settings-for-my-spring-boot-app', targetLocation: "${mavenSettingsFile}"]]]) {
        sh "mvn -s ${mavenSettingsFile} clean test-compile"
    }

    stage 'Unit tests'
    wrap([$class: 'ConfigFileBuildWrapper',
        managedFiles: [[fileId: 'maven-settings-for-my-spring-boot-app', targetLocation: "${mavenSettingsFile}"]]]) {
        sh "mvn -s ${mavenSettingsFile} test"
    }

    stage 'Reporting'
    wrap([$class: 'ConfigFileBuildWrapper',
        managedFiles: [[fileId: 'maven-settings-for-my-spring-boot-app', targetLocation: "${mavenSettingsFile}"]]]) {

        sh "mvn -s ${mavenSettingsFile} checkstyle:checkstyle pmd:pmd findbugs:findbugs"

        step([$class: 'ArtifactArchiver', artifacts: 'target/*.jar'])
        step([$class: 'WarningsPublisher', consoleParsers: [[parserName: 'Maven']]])
        step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
        step([$class: 'JavadocArchiver', javadocDir: 'target/site/apidocs/'])

        // Use fully qualified hudson.plugins.checkstyle.CheckStylePublisher if JSLint Publisher Plugin or JSHint Publisher Plugin is installed
        step([$class: 'hudson.plugins.checkstyle.CheckStylePublisher', pattern: '**/target/checkstyle-result.xml'])
        // In real life, PMD and Findbugs are unlikely to be used simultaneously
        step([$class: 'PmdPublisher', pattern: '**/target/pmd.xml'])
        step([$class: 'FindBugsPublisher', pattern: '**/findbugsXml.xml'])
        step([$class: 'AnalysisPublisher'])
    }

    stage 'Package and deploy'
    wrap([$class: 'ConfigFileBuildWrapper',
        managedFiles: [[fileId: 'maven-settings-for-my-spring-boot-app', targetLocation: "${mavenSettingsFile}"]]]) {

        sh "mvn -s ${mavenSettingsFile} source:jar javadoc:javadoc spring-boot:repackage deploy:deploy"

    }
    
}
