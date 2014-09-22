This directory contains scripts for customizing the certificate trust database
so that additional app stores can be added *for testing purposes only*. Gecko
does not properly support multiple app stores signing privileged apps, so these
scripts *cannot and should not* be used to add support for privileged apps from
stores for production purposes.

# Installation

Install [NSS](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS)
for `certutil`.

* Debian/Ubuntu: `sudo apt-get install libnss3-tools`
* Fedora: `su -c "yum install nss-tools"`
* openSUSE: `sudo zypper install mozilla-nss-tools`
* Mac OS X with [homebrew](http://brew.sh/): `brew install nss`

Install the [Android ADT (SDK)](http://developer.android.com/sdk/index.html).
After you install the SDK, add `$ADT/sdk/platform-tools` to your path.

# Usage

Example usage:

    rm -Rf certdb.tmp
    ./new_certdb.sh certdb.tmp
    ./add_or_replace_root_cert.sh certdb.tmp root-ca-reviewers-marketplace
    ./change_trusted_servers.sh full_unagi \
           "https://marketplace-dev.allizom.org,"\
           "https://marketplace.allizom.org,"\
           "https://payments-alt.allizom.org,"\
           "https://marketplace.firefox.com"
    ./push_certdb.sh full_unagi certdb.tmp

The term `full_unagi` is used to look up a mounted device. You could put any
valid device name here. For example, if your `adb devices` output looked like:

    List of devices attached
    d47ce87d	device

then you would type `./change_trusted_servers.sh d47ce87d...`.

If you want to add marketplace-dev support, add one or both of these before
executing `push_certdb.sh`

    ./add_or_replace_root_cert.sh certdb.tmp marketplace-dev-public-root
    ./add_or_replace_root_cert.sh certdb.tmp marketplace-dev-reviewers-root

If you want to add marketplace-stage support, add the following before executing push_certdb.sh

    ./add_or_replace_root_cert.sh certdb.tmp marketplace-stage-public-root

If you want to add payments-alt support, add the following before executing push_certdb.sh

    ./add_or_replace_root_cert.sh certdb.tmp marketplace-paymentsalt-public-root

These steps are done in separate scripts so that `add_or_replace_root_cert.sh`
can be used for B2G desktop, which doesn't require the push/pull steps.


Windows:

    hg clone https://hg.mozilla.org/projects/nspr
    hg clone https://hg.mozilla.org/projects/nss
    cd nss
    OS_TARGET=WIN95 make nss_build_all
    cd ..
    export NSS=$PWD/dist/WIN954.0_DBG.OBJ
    # NSS/bin must be ahead of the rest of the path because NSS and Windows both
    # have tools called "certutil".
    export PATH=$NSS/bin:$NSS/lib:$PATH
