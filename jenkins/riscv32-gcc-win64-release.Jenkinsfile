// This directory is where to expect a MSYS64 installation on the build node
String MSYSHOME = 'C:\\msys64j'

// Bug URL and Package Version parameters
properties([parameters([
    string(defaultValue: '', description: 'Package Version', name: 'PackageVersion'),
    string(defaultValue: '', description: 'Bug Reporting URL', name: 'BugURL'),
    string(defaultvalue: '', description: 'Binutils Tag', name: 'BinutilsTag'),
    string(defaultvalue: '', description: 'GDB Tag', name: 'GdbTag'),
    string(defaultvalue: '', description: 'GCC Tag', name: 'GccTag'),
    string(defaultvalue: '', description: 'Newlib Tag', name: 'NewlibTag'),
    booleanParam(defaultValue: false, description: 'Test with a reduced set of multilibs', name: 'ReducedMultilibTesting'),
])])

PKGVERS = params.PackageVersion
BUGURL = params.BugURL
if (PKGVERS != '')
  currentBuild.displayName = PKGVERS

node('winbuilder') {
  stage('Cleanup') {
    deleteDir()
  }

  stage('Checkout') {
    checkout scm
    // Store workspace dir in a file we can source later
    bat script: """${MSYSHOME}\\usr\\bin\\cygpath %WORKSPACE% > workspacedir"""

    dir('binutils') {
      checkout([$class: 'GitSCM',
          branches: [[name: "tags/${BinutilsTag}"]],
          extensions: [[$class: 'CloneOption', shallow: true]],
          userRemoteConfigs: [[url: 'https://sourceware.org/git/binutils-gdb.git']]])
    }
    dir('gdb') {
      checkout([$class: 'GitSCM',
          branches: [[name: "tags/${GdbTag}"]],
          extensions: [[$class: 'CloneOption', shallow: true]],
          userRemoteConfigs: [[url: 'https://sourceware.org/git/binutils-gdb.git']]])
    }
    dir('gcc') {
      checkout([$class: 'GitSCM',
          branches: [[name: "tags/${GccTag}"]],
          extensions: [[$class: 'CloneOption', shallow: true]],
          userRemoteConfigs: [[url: 'https://github.com/gcc-mirror/gcc.git']]])
    }
    dir('newlib') {
      checkout([$class: 'GitSCM',
          branches: [[name: "tags/${NewlibTag}"]],
          extensions: [[$class: 'CloneOption', shallow: true]],
          userRemoteConfigs: [[url: 'git://sourceware.org/git/newlib-cygwin.git']]])
    }
    dir('binutils-gdb-sim') {
      checkout([$class: 'GitSCM',
          branches: [[name: '*/spc-cgen-sim-rve']],
          extensions: [[$class: 'CloneOption', shallow: true]],
          userRemoteConfigs: [[url: 'https://github.com/embecosm/riscv-binutils-gdb.git']]])
    }
    bat script: """set MSYSTEM=MINGW64
                   set /P UNIXWORKSPACE=<workspacedir
                   ${MSYSHOME}\\usr\\bin\\bash --login -c ^
                       "cd %UNIXWORKSPACE% && ./describe-build.sh" """
    archiveArtifacts artifacts: 'build-sources.txt', fingerprint: true
  }

  stage('Build') {
    // Enable greater set of multilibs
    bat script: """set MSYSTEM=MINGW64
                   set /P UNIXWORKSPACE=<workspacedir
                   ${MSYSHOME}\\usr\\bin\\bash --login -c ^
                       "cd %UNIXWORKSPACE%/gcc/gcc/config/riscv && python3 ./multilib-generator rv32e-ilp32e-- rv32em-ilp32e-- rv32ea-ilp32e-- rv32ec-ilp32e-- rv32ema-ilp32e-- rv32emc-ilp32e-- rv32eac-ilp32e-- rv32emac-ilp32e-- rv32i-ilp32-- rv32im-ilp32-- rv32ia-ilp32-- rv32if-ilp32-- rv32if-ilp32f-- rv32ic-ilp32-- rv32ima-ilp32-- rv32imf-ilp32-- rv32imf-ilp32f-- rv32imc-ilp32-- rv32iaf-ilp32-- rv32iaf-ilp32f-- rv32iac-ilp32-- rv32ifd-ilp32-- rv32ifd-ilp32f-- rv32ifd-ilp32d-- rv32ifc-ilp32-- rv32ifc-ilp32f-- rv32imaf-ilp32-- rv32imaf-ilp32f-- rv32imac-ilp32-- rv32imfd-ilp32-- rv32imfd-ilp32f-- rv32imfd-ilp32d-- rv32imfc-ilp32-- rv32imfc-ilp32f-- rv32iafd-ilp32-- rv32iafd-ilp32f-- rv32iafd-ilp32d-- rv32iafc-ilp32-- rv32iafc-ilp32f-- rv32ifdc-ilp32-- rv32ifdc-ilp32f-- rv32ifdc-ilp32d-- rv32imafd-ilp32-- rv32imafd-ilp32f-- rv32imafd-ilp32d-- rv32imafc-ilp32-- rv32imafc-ilp32f-- rv32imfdc-ilp32-- rv32imfdc-ilp32f-- rv32imfdc-ilp32d-- rv32iafdc-ilp32-- rv32iafdc-ilp32f-- rv32iafdc-ilp32d-- rv32imafdc-ilp32-- rv32imafdc-ilp32f-- rv32imafdc-ilp32d-- rv64i-lp64-- rv64im-lp64-- rv64ia-lp64-- rv64if-lp64-- rv64if-lp64f-- rv64ic-lp64-- rv64ima-lp64-- rv64imf-lp64-- rv64imf-lp64f-- rv64imc-lp64-- rv64iaf-lp64-- rv64iaf-lp64f-- rv64iac-lp64-- rv64ifd-lp64-- rv64ifd-lp64f-- rv64ifd-lp64d-- rv64ifc-lp64-- rv64ifc-lp64f-- rv64imaf-lp64-- rv64imaf-lp64f-- rv64imac-lp64-- rv64imfd-lp64-- rv64imfd-lp64f-- rv64imfd-lp64d-- rv64imfc-lp64-- rv64imfc-lp64f-- rv64iafd-lp64-- rv64iafd-lp64f-- rv64iafd-lp64d-- rv64iafc-lp64-- rv64iafc-lp64f-- rv64ifdc-lp64-- rv64ifdc-lp64f-- rv64ifdc-lp64d-- rv64imafd-lp64-- rv64imafd-lp64f-- rv64imafd-lp64d-- rv64imafc-lp64-- rv64imafc-lp64f-- rv64imfdc-lp64-- rv64imfdc-lp64f-- rv64imfdc-lp64d-- rv64iafdc-lp64-- rv64iafdc-lp64f-- rv64iafdc-lp64d-- rv64imafdc-lp64-- rv64imafdc-lp64f-- rv64imafdc-lp64d-- rv32g-ilp32-- rv32g-ilp32f-- rv32g-ilp32d-- rv32gc-ilp32-- rv32gc-ilp32f-- rv32gc-ilp32d-- rv64g-lp64-- rv64g-lp64f-- rv64g-lp64d-- rv64gc-lp64-- rv64gc-lp64f-- rv64gc-lp64d-- > t-elf-multilib" """
    bat script: """set MSYSTEM=MINGW64
                   set BUGURL=${BUGURL}
                   set PKGVERS=${PKGVERS}
                   set EXTRA_BINUTILS_OPTS=--with-python=no --with-system-readline
                   set /P UNIXWORKSPACE=<workspacedir
                   ${MSYSHOME}\\usr\\bin\\bash --login -c ^
                       "cd %UNIXWORKSPACE% && ./stages/build-riscv32-gcc.sh" """
    bat script: """set MSYSTEM=MINGW64
                   set /P UNIXWORKSPACE=<workspacedir
                   ${MSYSHOME}\\usr\\bin\\bash --login -c ^
                       "cd %UNIXWORKSPACE% && ./utils/extract-mingw-dlls.sh" """
  }

  stage('Package') {
    bat script: """set MSYSTEM=MINGW64
                   set /P UNIXWORKSPACE=<workspacedir
                   ${MSYSHOME}\\usr\\bin\\bash --login -c ^
                       "cd %UNIXWORKSPACE% && utils/prepare-zip-package.sh ${PKGVERS}" """
    archiveArtifacts artifacts: "${PKGVERS}.zip", fingerprint: true
  }

  stage('Test') {
    bat script: """set MSYSTEM=MINGW64
                   set /P UNIXWORKSPACE=<workspacedir
                   ${MSYSHOME}\\usr\\bin\\bash --login -c ^
                       "cd %UNIXWORKSPACE%/build/binutils && make check-gas" """, returnStatus: true
    bat script: """set MSYSTEM=MINGW64
                   set /P UNIXWORKSPACE=<workspacedir
                   ${MSYSHOME}\\usr\\bin\\bash --login -c ^
                       "cd %UNIXWORKSPACE%/build/binutils && make check-ld" """, returnStatus: true
    bat script: """set MSYSTEM=MINGW64
                   set /P UNIXWORKSPACE=<workspacedir
                   ${MSYSHOME}\\usr\\bin\\bash --login -c ^
                       "cd %UNIXWORKSPACE%/build/binutils && make check-binutils" """, returnStatus: true
    dir('build/binutils') {
      archiveArtifacts artifacts: '''gas/testsuite/gas.log,
                                      gas/testsuite/gas.sum,
                                      ld/ld.log,
                                      ld/ld.sum,
                                      binutils/binutils.log,
                                      binutils/binutils.sum''',
                       fingerprint: true
    }
    // Build the CGEN simulator and use it for testing
    if (params.ReducedMultilibTesting)
      bat script: """set MSYSTEM=MINGW64
                    set REDUCED_MULTILIB_TEST=1
                    set /P UNIXWORKSPACE=<workspacedir
                    ${MSYSHOME}\\usr\\bin\\bash --login -c ^
                        "cd %UNIXWORKSPACE% && ./stages/test-riscv32-gcc.sh" """
    else
      bat script: """set MSYSTEM=MINGW64
                    set /P UNIXWORKSPACE=<workspacedir
                    ${MSYSHOME}\\usr\\bin\\bash --login -c ^
                        "cd %UNIXWORKSPACE% && ./stages/test-riscv32-gcc.sh" """
    dir('build/gcc-stage2') {
      archiveArtifacts artifacts: '''gcc/testsuite/gcc/gcc.log,
                                     gcc/testsuite/gcc/gcc.sum,
                                     gcc/testsuite/g++/g++.log,
                                     gcc/testsuite/g++/g++.sum''',
                       fingerprint: true
    }
  }
}
