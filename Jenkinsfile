def someFunction(name, id, rec, test){
    echo "${name} ${id} ${rec} ${test}"
}
pipeline{
    agent any
   parameters {
  string defaultValue: '"pipeline"', name: 'GIT_BRANCH'
  booleanParam defaultValue: true, name: 'RUN_TEST'
  booleanParam defaultValue: true, name: 'SEND_MAIL'
}
    
    stages {
        stage("Download"){
            
            steps{
                cleanWs()
                echo (message: "Download")
                dir('pipeline'){
                    git(
                    branch: "${params.GIT_BRANCH}",
                    url: 'https://github.com/KLevon/jenkins-course'
                    )
                }
                rtDownload (
                    serverId: 'zaba',
                    spec: '''{
                        "files": [
                        {
                            "pattern": "generic-local/printer.zip",
                            "target": "printer_zip/"
                        }]
                    }'''
                    )
                    
                unzip(
                    zipFile: "printer_zip/printer.zip",
                    dir: "pipeline/"
                    )
            }
        }
        stage("Build"){
            steps{
                echo (message: "Build")
                bat(
                    script:"""
                        cd pipeline
                        Makefile.bat
                        
                    """
                    )
            }
        }
        stage("test"){
             when {
                equals(actual: params.RUN_TEST, expected: true)
            }
            steps{
                
                script{
                        env.output = ""
                        def niz = ["printer", "scanner", "main"]
                        for (element in niz) {
                                env.output += bat(script: "./pipeline/Tests.bat ${element}", returnStdout: true).trim()
                        }
                    
                }
                
            }
        }
         stage("Publish"){
            steps{
                echo (message: "Publish")
                script{
                    zip(
                    zipFile: "pipe_zip.zip", 
                    archive: true,
                    dir: "pipeline/"
                    )
                }
                
                rtUpload (
                    serverId: 'zaba',
                    spec: """{
                        "files": [
                        {
                            "pattern": "pipe_zip.zip",
                            "target": "generic-local/relese/${env.BUILD_ID}/"
                        }
                        ]
                       } """
                )
                
            }
        }
        
    }
    
    post {
        success{
             script{
                 someFunction("ime", env.BUILD_ID, "Ja", env.output)
                 
                if(params.SEND_MAIL){
                     mail to: 'beslic.alex@gmail.com',
                     from: 'levonkeleman@levonkeleman.com', 
                     subject: 'Jenkins',
                     body: "Radi! ${env.BUILD_ID} ${env.output}"
                     

                }
             }
                   }
        failure{
            echo "email: ${env.BUILD_ID} ${env.output}"
        }
        
        unstable {
            echo "email: ${env.BUILD_ID}"
        }
    }
}
