#!groovy

docker.image('cloudbees/java-build-tools:0.0.6').inside {

    checkout scm

    stage 'Compile and Test'
    def mavenSettingsFile = "${pwd()}/.m2/settings.xml"
    wrap([$class: 'ConfigFileBuildWrapper',
        managedFiles: [[fileId: 'maven-settings-for-my-spring-boot-app', targetLocation: "${mavenSettingsFile}"]]]) {

        sh "mvn -s ${mavenSettingsFile} clean source:jar javadoc:javadoc checkstyle:checkstyle pmd:pmd findbugs:findbugs deploy"

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
}
