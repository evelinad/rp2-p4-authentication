# rp2-p4-authentication
----
P4 authentication proof of concept using GRE and a 'keyed' checksum to simulate message authentication.

## Installation overview
The files of this repository should be placed in the root of the behavioral model directory. Then the autoconf.sh and configure scripts need to be run. In case the behavioral model has not been compiled yet, make needs to be run too. The simple\_router\_auth\_poc has been made using the 1.0.0 version of the software switch. The installation commands will install the environment (the behavioral model and the p4auth PoC target) from scratch on a Ubuntu 16.04 x86-64 installation.

Alternatively, the repo can be cloned somewhere else and the files copied to the behavioral model directory. Afterwards autogen.sh and configure need to be (re-)run. 
The p4-auth repository comes with the following modified files:

* configure.ac: adds targets/simple_router_auth_poc/Makefile to AC_CONFIG_FILES
* targets/Makefile.am: adds simple_router_auth_poc to SUBDIRS
* mininet/p4_mininet.py: adds additional args to improve logging (requires debugging to be enabled)

It is also possible to just place the simple\_router\_auth\_poc directory in the targets directory, copy the Makefile of the simple\_target to it (after it has been generated by autoconf and automake) and run make from the target's directory. To improve mininet's logging the following can be added manually to *behavioral-model/mininet/p4\_mininet.py*:

>@100  
>args.append('--log-file /tmp/p4s.%s.verbose.log --log-flush --debugger' % self.name)

## Installation commands

    git clone -b 1.0.0 https://github.com/p4lang/behavioral-model.git
    cd behavioral-model

    # prevent pip error from failing install_deps.sh
    #sudo apt install python-pip # not necessary any more for on an up to date installation
    #sudo -H pip install --upgrade pip
    ./install_deps.sh
    sudo pip install thrift # otherwise mininet fails to launch (running install_deps.sh as root does not resolve it)

Add P4 auth code:

    git remote add p4auth https://github.com/JcKlomp/rp2-p4-authentication.git
    git fetch p4auth
    git checkout p4auth/master -- . # copy the p4auth files into the bmv2 dir

Build the behavioral model:

    ./autogen.sh
    ./configure 'CXXFLAGS=-O0 -g' --enable-debugger
    make

At this point the behavioral model should have been built including all the necessary files of the simple\_router\_auth\_poc target.

Optionally, install p4c-bm:

    git clone -b 1.0.0 https://github.com/p4lang/p4c-bm.git
    cd p4c-bm
    #sudo pip install --upgrade pip # in case following command fails
    sudo pip install -r requirements.txt
    sudo python setup.py install

Install dependencies for auth demonstration:

    sudo apt install tmux mininet scapy wireshark tshark
    # for tshark enable dumpcap's non-superusers option and ensure the user is part of wireshark group `sudo usermod -a -G wireshark $USER && su $USER`

Now the target can be tested by running *mininet.sh* from the target's directory.
After launching it the command `h1 ping -c1 h2` should result in a successful response (table entries are added automatically after 2 seconds; sometimes this timeout is too low and needs to be increased or it can be tried to launch mininet again after files have been cached).
Then, to test the authentication PoC *tmux* should be started to run `source demonstration.sh` (preferably switch to a root user before starting tmux, otherwise the table entries need to be added manually) or `sudo ./packet_sender.py` can be used manually while the target is runnning in *mininet*.

## Files and their purpose

* simple_router.p4: P4 code
* packet_sender.py: script for sending authenticated packets in the *Mininet* environment using *Scapy*
* demonstration.sh: automatically create demonstration environment in *tmux* using *packet_sender.py*, *Mininet* and *tshark*
* add_entries.sh: simple script to add table entries via *commands.txt* and *runtime_CLI*
* commands.txt: table entry commands used with *runtime_CLI*
* debugger.sh: simple script to start the debugger via *p4dbg.py*
* launch.sh: wrapper around *run.sh* for launching the software switch without *Mininet*
* logger.sh: simple script for starting the *nanomsg* logger
* mininet.sh: script for launching the software switch in the *Mininet* environment; table entries are automatically added and a sniffer (e.g., *Wireshark*) can be launched
* mininet-host-down.sh: script for bringing a *Mininet* link down and up (no real purpose other than testing)
* p4-compile2-json.sh: simple script for compiling the P4 program to json representation via *p4c-bmv2*; can be used for replacing problematic values in  the json afterwards via multi-line *perl* search & replace
* primitives.cpp: local software switch source code containing the primitive actions; code for additional primitive actions copied from the BMv2 *simple_switch* target
* tshark.sh: script used by *demonstration.sh* around *tshark* for printing relevant packet data
* wireshark.sh: simple script for manually starting *Wireshark*; obsolete since using the *mininet.sh* launcher will also do this
