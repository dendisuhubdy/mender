pipeline {
  agent { label 'ubuntu-18.04' }

  triggers {
    upstream(
      upstreamProjects: 'hioa-cs-org-test/IncludeOS/dev', threshold: hudson.model.Result.SUCCESS
      )
  }

  environment {
    PROFILE_x86_64 = 'clang-6.0-linux-x86_64'
    PROFILE_x86 = 'clang-6.0-linux-x86'
    CPUS = """${sh(returnStdout: true, script: 'nproc')}"""
    CC = 'clang-6.0'
    CXX = 'clang++-6.0'
    PACKAGE = 'mender'
    USER = 'includeos'
    CHAN = 'test'
  }

  stages {
    stage('Pull Request pipeline') {
      when { changeRequest() }
      stages {
        /*stage('Unit tests') {
          steps {
            //cmake cache is bad
            sh script: "mkdir -p unittests && rm -rf unittests/*", label: "Setup"
            sh script: "cd unittests; cmake ../unit", label: "cmake configure"
            sh script: "cd unittests; make -j $CPUS", label: "build tests"
            sh script: "cd unittests; ctest --output-on-failure", label: "run unit tests"
          }
        }*/

        stage('Build package') {
          steps {
            build_conan_package("$PROFILE_x86_64")
          }
        }
        /*
        stage('build example') {
          steps {
            sh script: "mkdir -p build_example", label: "Setup"
            sh script: "cd build_example; conan install ../integration/simple -pr $PROFILE_x86_64 -u", label: "conan_install"
            sh script: "cd build_example; cmake ../integration/simple",label: "cmake configure"
            sh script: "cd build_example; make -j $CPUS", label: "building example"
          }
        }*/
      }
    }
    stage('Deploy package pipeline') {
      when {
        anyOf {
          branch 'master'
        }
      }
      stages {
        stage('Build Conan package') {
          steps {
            //TODO foreach profile ?
            build_conan_package("$PROFILE_x86_64")
          }
        }
        stage('Upload to bintray') {
          steps {
            sh script: """
              VERSION=\$(conan inspect -a version . | cut -d " " -f 2)
              conan upload --all -r ${env.CONAN_REMOTE} $PACKAGE/\$VERSION@$USER/$CHAN
            """, label: "Upload to bintray"
          }
        }
      }
    }
  }
  post {
    cleanup {
      sh script: """
        VERSION=\$(conan inspect -a version . | cut -d " " -f 2)
        conan remove $PACKAGE/\$VERSION@$USER/$CHAN -f || echo 'Could not remove. This does not fail the pipeline'
      """, label: "Cleaning up and removing conan package"
    }
  }
}


def build_conan_package(String profile) {
  sh script: "conan create . $USER/$CHAN -pr ${profile}", label: "Build with profile: $profile"
}
