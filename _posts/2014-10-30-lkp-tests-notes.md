---
layout: post
title: LKP Tests Notes
---



## Introduction

LKP-Tests is a linux kernel performance testing framework authored by Fengguang Wu @ Intel.

Codes gitted from

`git://git.kernel.org/pub/scm/linux/kernel/git/wfg/lkp-tests.git`

## setup-local

### make_wakeup

	(1) make and enter dir /tmp/event_pipe
	(2) make FIFO file argv[1]
	(3) if run by ./wait then exit
	(4) write 1024 chars out to the FIFO file
	(5) fork
		parent: write the child pid out to /tmp/pid-wakeup, then exit
		child: close all the files inherited from parent
	(6) child is raised to the process group leader of a new session
	(7) ignore four sigals: SIGCHLD/SIGTSTP/SIGTTOU/SIGTTIN
	(8) sleep 100 hours

### create_lkp_dirs

	(1) create LKP_HOME/tmp
	(2) create /lkp/paths
	(3) create /lkp/benchmarks

### create_host_config

	(1) guarantee file LKP_HOME/hosts/HOSTNAME exists
	(2) write the memory size to it
	(3) maybe some other arguments

### insall_packages

	(1) call `install_debian_packages('lkp', 'debian')`
		install the packages listed in LKP_HOME/debian/lkp
	(2) call `install_debian_packages('lkp-dev', 'debian')`
		install the packages listed in LKP_HOME/debian/lkp-dev

### Iterate over `scripts`

#### The framework

Also call `install_debian_packages(script)` or `install benchmark(script)`. But here the scripts mainly download and install the benchmarks. The details on this procedure need more survey.

#### The pack scripts

The scripts that download and install benchmarks are put in LKP_HOME/pack. LKP_HOME/pack/default defines many shell script functions in the default case. Other files in LKP_HOME/pack override the functions in each case while downloading and installing one certain benchmark.

With the scripts in LKP_HOME/pack, the LKP_HOME/sbin/pack works. It just call the `download`, `build` and `install`, and then `pack_deb`, `cleanup`.

## run-local

