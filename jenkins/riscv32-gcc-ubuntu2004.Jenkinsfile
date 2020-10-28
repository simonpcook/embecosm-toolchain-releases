// Default package version and bug URL
import java.time.*
import java.time.format.DateTimeFormatter
String CURRENTTIME = LocalDateTime.ofInstant(Instant.now(), ZoneOffset.UTC) \
                         .format(DateTimeFormatter.ofPattern("yyyyMMdd"))
String PKGVERS = "riscv32-embecosm-gcc-ubuntu2004-${CURRENTTIME}"
String BUGURL = 'https://www.embecosm.com'

// Bug URL and Package Version override parameters
properties([parameters([
    string(defaultValue: '', description: 'Package Version', name: 'PackageVersion'),
    string(defaultValue: '', description: 'Bug Reporting URL', name: 'BugURL'),
    booleanParam(defaultValue: false, description: 'Test with a reduced set of multilibs', name: 'ReducedMultilibTesting'),
])])

if (params.PackageVersion != '')
  PKGVERS = params.PackageVersion
if (params.BugURL != '')
  BUGURL = params.BugURL
currentBuild.displayName = PKGVERS

node('builder') {
  stage('Cleanup') {
    deleteDir()
  }

  stage('Checkout') {
    checkout scm
    dir('binutils-gdb') {
      checkout([$class: 'GitSCM',
          branches: [[name: '*/master']],
          extensions: [[$class: 'CloneOption', shallow: true]],
          userRemoteConfigs: [[url: 'https://mirrors.git.embecosm.com/mirrors/binutils-gdb.git']]])
    }
    dir('gcc') {
      checkout([$class: 'GitSCM',
          branches: [[name: '*/master']],
          extensions: [[$class: 'CloneOption', shallow: true]],
          userRemoteConfigs: [[url: 'https://mirrors.git.embecosm.com/mirrors/gcc.git']]])
    }
    dir('newlib') {
      checkout([$class: 'GitSCM',
          branches: [[name: '*/master']],
          extensions: [[$class: 'CloneOption', shallow: true]],
          userRemoteConfigs: [[url: 'https://mirrors.git.embecosm.com/mirrors/newlib-cygwin.git']]])
    }
    dir('binutils-gdb-sim') {
      checkout([$class: 'GitSCM',
          branches: [[name: '*/spc-cgen-sim-rve']],
          extensions: [[$class: 'CloneOption', shallow: true]],
          userRemoteConfigs: [[url: 'https://github.com/embecosm/riscv-binutils-gdb.git']]])
    }
    sh script: './describe-build.sh'
    archiveArtifacts artifacts: 'build-sources.txt', fingerprint: true
  }

  stage('Prepare Docker') {
    image = docker.build('build-env-ubuntu2004',
                         '--no-cache -f docker/linux-ubuntu2004.dockerfile docker')
  }

  stage('Build') {
    image.inside {
      // Enable greater set of multilibs
      sh script: 'cd gcc/gcc/config/riscv && python3 ./multilib-generator rv32e-ilp32e-- rv32em-ilp32e-- rv32ea-ilp32e-- rv32ec-ilp32e-- rv32ema-ilp32e-- rv32emc-ilp32e-- rv32eac-ilp32e-- rv32emac-ilp32e-- rv32i-ilp32-- rv32im-ilp32-- rv32ia-ilp32-- rv32if-ilp32-- rv32if-ilp32f-- rv32ic-ilp32-- rv32ima-ilp32-- rv32imf-ilp32-- rv32imf-ilp32f-- rv32imc-ilp32-- rv32iaf-ilp32-- rv32iaf-ilp32f-- rv32iac-ilp32-- rv32ifd-ilp32-- rv32ifd-ilp32f-- rv32ifd-ilp32d-- rv32ifc-ilp32-- rv32ifc-ilp32f-- rv32imaf-ilp32-- rv32imaf-ilp32f-- rv32imac-ilp32-- rv32imfd-ilp32-- rv32imfd-ilp32f-- rv32imfd-ilp32d-- rv32imfc-ilp32-- rv32imfc-ilp32f-- rv32iafd-ilp32-- rv32iafd-ilp32f-- rv32iafd-ilp32d-- rv32iafc-ilp32-- rv32iafc-ilp32f-- rv32ifdc-ilp32-- rv32ifdc-ilp32f-- rv32ifdc-ilp32d-- rv32imafd-ilp32-- rv32imafd-ilp32f-- rv32imafd-ilp32d-- rv32imafc-ilp32-- rv32imafc-ilp32f-- rv32imfdc-ilp32-- rv32imfdc-ilp32f-- rv32imfdc-ilp32d-- rv32iafdc-ilp32-- rv32iafdc-ilp32f-- rv32iafdc-ilp32d-- rv32imafdc-ilp32-- rv32imafdc-ilp32f-- rv32imafdc-ilp32d-- rv64i-lp64-- rv64im-lp64-- rv64ia-lp64-- rv64if-lp64-- rv64if-lp64f-- rv64ic-lp64-- rv64ima-lp64-- rv64imf-lp64-- rv64imf-lp64f-- rv64imc-lp64-- rv64iaf-lp64-- rv64iaf-lp64f-- rv64iac-lp64-- rv64ifd-lp64-- rv64ifd-lp64f-- rv64ifd-lp64d-- rv64ifc-lp64-- rv64ifc-lp64f-- rv64imaf-lp64-- rv64imaf-lp64f-- rv64imac-lp64-- rv64imfd-lp64-- rv64imfd-lp64f-- rv64imfd-lp64d-- rv64imfc-lp64-- rv64imfc-lp64f-- rv64iafd-lp64-- rv64iafd-lp64f-- rv64iafd-lp64d-- rv64iafc-lp64-- rv64iafc-lp64f-- rv64ifdc-lp64-- rv64ifdc-lp64f-- rv64ifdc-lp64d-- rv64imafd-lp64-- rv64imafd-lp64f-- rv64imafd-lp64d-- rv64imafc-lp64-- rv64imafc-lp64f-- rv64imfdc-lp64-- rv64imfdc-lp64f-- rv64imfdc-lp64d-- rv64iafdc-lp64-- rv64iafdc-lp64f-- rv64iafdc-lp64d-- rv64imafdc-lp64-- rv64imafdc-lp64f-- rv64imafdc-lp64d-- rv32g-ilp32-- rv32g-ilp32f-- rv32g-ilp32d-- rv32gc-ilp32-- rv32gc-ilp32f-- rv32gc-ilp32d-- rv64g-lp64-- rv64g-lp64f-- rv64g-lp64d-- rv64gc-lp64-- rv64gc-lp64f-- rv64gc-lp64d-- > t-elf-multilib'
      sh script: "BUGURL='${BUGURL}' PKGVERS='${PKGVERS}' ./stages/build-riscv32-gcc.sh"
    }
  }

  stage('Package') {
    image.inside {
      sh script: "tar -czf ${PKGVERS}.tar.gz --transform s/^install/${PKGVERS}/ install"
      archiveArtifacts artifacts: "${PKGVERS}.tar.gz", fingerprint: true
    }
  }

  stage('Test') {
    image.inside {
      dir('build/binutils-gdb') {
        sh script: 'make check-gas', returnStatus: true
        sh script: 'make check-ld', returnStatus: true
        sh script: 'make check-binutils', returnStatus: true
        archiveArtifacts artifacts: '''gas/testsuite/gas.log,
                                       gas/testsuite/gas.sum,
                                       ld/ld.log,
                                       ld/ld.sum,
                                       binutils/binutils.log,
                                       binutils/binutils.sum''',
                         fingerprint: true
      }
      if (params.ReducedMultilibTesting)
        sh script: '''REDUCED_MULTILIB_TEST=1 ./stages/test-riscv32-gcc.sh'''
      else
        sh script: '''./stages/test-riscv32-gcc.sh'''
      dir('build/gcc-stage2') {
        archiveArtifacts artifacts: '''gcc/testsuite/gcc/gcc.log,
                                       gcc/testsuite/gcc/gcc.sum,
                                       gcc/testsuite/g++/g++.log,
                                       gcc/testsuite/g++/g++.sum''',
                         fingerprint: true
      }
    }
  }
}
