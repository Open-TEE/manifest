# Manifest

Manifest that is used by repo to synchronize the projects

# Integrating OpenTEE and OPTEE xtest

`xtest.xml` contains the manifest file to download the required repos
for integrating OPTEE `xtest` and OpenTEE.

`xtest` is not designed to run outside the ARM/OPTEE ecosystem.
We added support to build it for x86-64 based systems using
OpenTEE as TEE backend.

OpenTEE only passes a subset of the `xtest` suite,
this integration is an starting point to detect issues.

## Instructions

The build process is very similar to [OpenTEE](https://github.com/Open-TEE/project#quickstart-guide).
During this process, we'll use the Docker workflow of [OpenTEE](https://github.com/Open-TEE/project#docker).
The instructions below should be self-contained.

```bash
# Google repo (skip if you already have it)
mkdir -p ~/bin
curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod +x ~/bin/repo

# Clone OpenTEE and OPTEE required repos
mkdir opentee && cd opentee
~/bin/repo init -u https://github.com/acaldaya/xtest-opentee-manifest.git -b xtest_integration -m xtest.xml
~/bin/repo sync -j10

# IMPORTANT:
cd project/docker
# Update the GID and UID in Dockerfile.opentee to match yours

# Create a Docker image
# (sometimes make -j during mbedtls build fails...)
./build_docker.sh

# Start a container
./run_docker.sh

##########################
## Inside the container ##
##########################

# Build OpenTEE and install it
# Note: install location is "/opt/OpenTee"
cd ~/opentee
mkdir -p build && cd build
../autogen.sh
make -j && sudo make install

# At this point OpenTEE is installed on the system
# Run OpenTEE and example program
/opt/OpenTee/bin/opentee-engine
/opt/OpenTee/bin/example_sha1
# Expected output:
    START: example SHA1 calc app
    Initializing context: initialized
    Openning session: opened
    Registered in mem..
    Invoking command: Update sha1: done
    Registered out mem..
    Invoking command: Do final sha1: done
    Calculated sha1: df6f52ec2aa631b60a71ab23ab985a58cc67434f
    Releasing shared out memory..
    Releasing shared in memory..
    Closing session..
    Finalizing ctx..
    END: example SHA1 calc app

# Build xtest
cd ~/opentee
mkdir -p optee_test/build && cd optee_test/build
cmake ..
make
sudo make install

# Run xtest with OpenTEE as TEE backend
cd ~/opentee/optee_test/build
make reset && ./xtest -t regression 4101
# Expected output...
    [100%] Rebuilding xtest/TAs and force OpenTEE to reload the TAs
    [  3%] Built target concurrent_large
    [  8%] Built target aes_perf
    [ 13%] Built target sha_perf
    [ 28%] Built target crypt
    [ 33%] Built target sims_keepalive
    [ 37%] Built target create_fail_test
    [ 41%] Built target sims
    [ 45%] Built target concurrent
    [ 50%] Built target miss
    [ 58%] Built target os_test
    [ 62%] Built target rpc_test
    [ 66%] Built target large
    [100%] Built target xtest
    [  3%] Built target concurrent_large
    [  8%] Built target aes_perf
    [ 13%] Built target sha_perf
    [ 28%] Built target crypt
    [ 33%] Built target sims_keepalive
    [ 37%] Built target create_fail_test
    [ 41%] Built target sims
    [ 45%] Built target concurrent
    [ 50%] Built target miss
    [ 58%] Built target os_test
    [ 62%] Built target rpc_test
    [ 66%] Built target large
    [100%] Built target xtest
    Install the project...
    -- Install configuration: ""
    -- Up-to-date: /opt/OpenTee/lib/TAs/libaes_perf.so
    -- Up-to-date: /opt/OpenTee/lib/TAs/libsha_perf.so
    -- Up-to-date: /opt/OpenTee/lib/TAs/libcrypt.so
    -- Up-to-date: /opt/OpenTee/lib/TAs/libconcurrent.so
    -- Up-to-date: /opt/OpenTee/lib/TAs/libconcurrent_large.so
    -- Up-to-date: /opt/OpenTee/lib/TAs/libos_test.so
    -- Up-to-date: /opt/OpenTee/lib/TAs/libcreate_fail_test.so
    -- Up-to-date: /opt/OpenTee/lib/TAs/libsims.so
    -- Up-to-date: /opt/OpenTee/lib/TAs/libsims_keepalive.so
    -- Up-to-date: /opt/OpenTee/lib/TAs/libmiss.so
    -- Up-to-date: /opt/OpenTee/lib/TAs/librpc_test.so
    -- Up-to-date: /opt/OpenTee/lib/TAs/liblarge.so
    -- Up-to-date: /home/docker/opentee/optee_test/build/xtest
    Built target reset
    Test ID: 4101
    Run test suite with level=0

    TEE test application started over default TEE instance
    ######################################################
    #
    # regression
    #
    ######################################################

    * regression_4101 Test TEE Internal API Arithmetical API - Bigint init
    o regression_4101.1 Normal allocation and initialization
    regression_4101.1 OK
    o regression_4101.2 Zero allocation
    regression_4101.2 OK
    o regression_4101.3 Too large
    regression_4101.3 OK
    o regression_4101.4 Boundaries
    regression_4101.4 OK
    regression_4101 OK
    +-----------------------------------------------------
    Result of testsuite regression filtered by "4101":
    regression_4101 OK
    +-----------------------------------------------------
    26 subtests of which 0 failed
    1 test case of which 0 failed
    84 test cases were skipped
    TEE test application done!
```

## More details?

Take a look at `xtest.xml` repositories and their revisions for
the log of changes needed to make this work.
