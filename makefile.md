BINARY_NAME=gogs

build:
 go build -o gogs
 
run: build
 ./gogs web &

clean:
 go clean
 rm ${BINARY_NAME}-darwin
 rm ${BINARY_NAME}-linux
 rm ${BINARY_NAME}-windows



pipeline {
    agent any
    tools { go '1.18' }

    stages {
        stage('go_unit_test') {
            steps {
                sh '''
                    xGO_TEST_RESULTS_FILE=/tmp/go_test_result
                    go get -x -t ./...
                    go test -cover ./... | tee $xGO_TEST_RESULTS_FILE 
                    xGO_TEST_EXIT_CODE=$?
                    xGO_TEST_FAILED="$(grep '^FAIL' $xGO_TEST_RESULTS_FILE | wc -l)"
                    xGO_TEST_OK="$(grep '^ok' $xGO_TEST_RESULTS_FILE | wc -l)"
                    xGO_TEST_NO="$(grep '^?' $xGO_TEST_RESULTS_FILE | wc -l)"
                '''
            }
        }
        stage('generar_artefacto') {
            steps {
                sh '''
                    go build -o gogs
                '''
            }
        }
        stage('pruebas_funcionales') {
            steps {
                sh '''
                    ./gogs web &
                    sleep 10
                    curl -I http://localhost:3000/
                    xCURL_TEST=$?
                    if [ $xCURL_TEST -eq 0 ]; then
                    	echo "TEST SUCCESSFUL"
                    else
                    	echo "TEST FAIL"
                    fi                    	  
                '''
            }
        }
    }
}
