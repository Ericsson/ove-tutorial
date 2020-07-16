# OVE Tutorial

This tutorial will hopefully give you an basic idea what OVE is about. If you are looking for a more in-depth description of OVE, please head over to [OVE](https://github.com/Ericsson/ove). If you on the other hand are eager to take OVE for a spin right away, just go ahead with the instructions below, it's easy!

## Part I: Set up the OVE workspace

Run the oneliner:

    curl -sSL https://raw.githubusercontent.com/Ericsson/ove/master/setup | bash -s my-ove-workspace https://github.com/Ericsson/ove-tutorial

'my-ove-workspace' will be the name of your OVE workspace. You can use multiple workspaces to keep states of different tasks for the same or different projects.


For the interested, the above oneliner runs the code in the snippet below. PLEASE NOTE: Do not manually run them, they are only there to shine light over the oneliner you have already executed. Most people do not really have to care about what the setup script is doing so let us skip below code snippet and move on!

    $ #Short description of what the oneliner setup script is actually doing
    $ #mkdir -vp ove
    $ #cd ove
    $ #git clone https://github.com/Ericsson/ove.git .ove &
    $ #git clone https://github.com/Ericsson/ove-tutorial &
    $ #wait
    $ #ln -s ove-tutorial .owel
    $ #ln -s .ove/ove ove

The '**setup**' script will now ask you to enter the '**my-ove-workspace**' directory and run '**source ove**', please do so. You should observe the following:

    $ source ove
    OVE [SHA-1: 0959726 @ Ubuntu 18.04]

    This script will do a few things:

    * add 120 bash functions:
    ove-!   ove-config    ove-lastlog       ove-refresh
    ove-add ove-configure ove-list-commands ove-remote
    ...
    Now what? Run 'ove fetch' to sync with the outside world or 'ove help' for more information

If you are missing some of the required dependencies on your host, OVE will let you know and suggest a solution:

    $ source ove
    error: missing command(s):

        tree

    To fix this, run the following command:

        apt install tree

Now, run '**ove fetch**' to clone the rest of the repositories:

    $ ove fetch
    Cloning into 'codechecker'...
    Cloning into 'dmce'...
    Cloning into 'tmux'...
    ...
    .ove          master   ## master...origin/master
    codechecker   master   ## master...origin/master
    dmce          master   ## master...origin/master
    ove-tutorial  d78b9d9  ## master...origin/master
    tmux          2.9      ## HEAD (no branch)

Checking the content of your newly created workspace will reveal the source code repos (codechecker, dmce, tmux), the OWEL (ove-tutorial) and OVE itself:

    $ ls -A
    codechecker  dmce  ove  .ove  .ove.state  .ove.tmp  ove-tutorial  .owel  README  tmux

The first part of the tutorial is done! You have enhanced your bash shell with OVE functionality, set up the top git repo (OWEL) and fetched the included source git repos. This covers the basics of versioning. Let us move on and build stuff!

## Part II: Build

Your OVE workspace now consists of five git repositories (all from github.com) and five projects:

    $ ove list-repositories
    .ove          https://github.com/Ericsson/ove.git          https://github.com/Ericsson/ove.git          master
    codechecker   https://github.com/Ericsson/codechecker.git  https://github.com/Ericsson/codechecker.git  master
    dmce          https://github.com/PatrikAAberg/dmce.git     https://github.com/PatrikAAberg/dmce.git     master
    ove-tutorial  https://github.com/Ericsson/ove-tutorial.git https://github.com/Ericsson/ove-tutorial.git master
    tmux          https://github.com/tmux/tmux.git             https://github.com/tmux/tmux.git             2.9

    $ ove list-projects
    codechecker
    dmce
    libevent
    ncurses
    tmux


For OVE, a project is something that produces output (e.g. an executable, a library or anything else machine-made). Even though projects are normally contained within a  corresponding git repo, OVE treats projects and repos independently. Multiple projects can be configured using code and build systems from the same repo, and one project can use code and build systems from multiple repos.
OVE keeps track of the dependencies between projects, so let us check the build order for the tutorial OWEL:

    $ ove build-order
    codechecker dmce libevent ncurses tmux

    $ ove digraph
    digraph ove_tutorial {
        codechecker;
        dmce;
        libevent;
        ncurses;
        tmux -> libevent;
        tmux -> ncurses;
    }

'**codechecker**' and '**dmce**' does not have any project dependencies and will be built first. '**tmux**' on the other hand needs both '**libevent**' and '**ncurses**' and will be built last.

Let us do a 'dry-run' build first:

    $ ove dry-run
    OVE_DRY_RUN 1

    $ ove buildme
    2019-05-27 09:00:15.986867812 (+00:00:00:000000000):Would prompt to install this/these package(s):
    autoconf
    automake
    bison
    build-essential
    clang
    clang-tidy
    clang-tools
    doxygen
    gcc-multilib
    graphviz
    libtool
    pkg-config
    python-dev
    python-setuptools
    python-virtualenv
    thrift-compiler
    2019-05-27 09:00:15.991281005 (+00:00:00:004413193):dmce: bootstrap
    cd /root/ove/dmce && /root/ove/ove-tutorial/projects/dmce/bootstrap
    2019-05-27 09:00:15.994350085 (+00:00:00:003069080):dmce: bootstrap: done
    2019-05-27 09:00:15.998903938 (+00:00:00:004553853):libevent: bootstrap
    cd /root/ove/libevent && /root/ove/ove-tutorial/projects/libevent/bootstrap
    2019-05-27 09:00:16.002080645 (+00:00:00:003176707):libevent: bootstrap: done
    2019-05-27 09:00:16.007579340 (+00:00:00:005498695):ncurses: bootstrap
    cd /root/ove/ncurses && /root/ove/ove-tutorial/projects/ncurses/bootstrap
    2019-05-27 09:00:16.012477340 (+00:00:00:004898000):ncurses: bootstrap: done
    2019-05-27 09:00:16.017780778 (+00:00:00:005303438):tmux: bootstrap
    cd /root/ove/tmux && /root/ove/ove-tutorial/projects/tmux/bootstrap
    2019-05-27 09:00:16.022573403 (+00:00:00:004792625):tmux: bootstrap: done
    2019-05-27 09:00:16.028434412 (+00:00:00:005861009):codechecker: build
    cd /root/ove/codechecker && /root/ove/ove-tutorial/projects/codechecker/build
    2019-05-27 09:00:16.033221461 (+00:00:00:004787049):codechecker: build: done
    2019-05-27 09:00:16.039264356 (+00:00:00:006042895):libevent: configure
    cd /root/ove/libevent && /root/ove/ove-tutorial/projects/libevent/configure
    2019-05-27 09:00:16.044322103 (+00:00:00:005057747):libevent: configure: done
    2019-05-27 09:00:16.049945744 (+00:00:00:005623641):libevent: build
    cd /root/ove/libevent && /root/ove/ove-tutorial/projects/libevent/build
    2019-05-27 09:00:16.054730696 (+00:00:00:004784952):libevent: build: done
    2019-05-27 09:00:16.060096456 (+00:00:00:005365760):libevent: install
    cd /root/ove/libevent && /root/ove/ove-tutorial/projects/libevent/install
    2019-05-27 09:00:16.065103280 (+00:00:00:005006824):libevent: install: done
    2019-05-27 09:00:16.070573017 (+00:00:00:005469737):ncurses: configure
    cd /root/ove/ncurses && /root/ove/ove-tutorial/projects/ncurses/configure
    2019-05-27 09:00:16.075670678 (+00:00:00:005097661):ncurses: configure: done
    2019-05-27 09:00:16.081257923 (+00:00:00:005587245):ncurses: build
    cd /root/ove/ncurses && /root/ove/ove-tutorial/projects/ncurses/build
    2019-05-27 09:00:16.086333692 (+00:00:00:005075769):ncurses: build: done
    2019-05-27 09:00:16.091945760 (+00:00:00:005612068):ncurses: install
    cd /root/ove/ncurses && /root/ove/ove-tutorial/projects/ncurses/install
    2019-05-27 09:00:16.096863762 (+00:00:00:004918002):ncurses: install: done
    2019-05-27 09:00:16.102191679 (+00:00:00:005327917):tmux: configure
    cd /root/ove/tmux && /root/ove/ove-tutorial/projects/tmux/configure
    2019-05-27 09:00:16.107158907 (+00:00:00:004967228):tmux: configure: done
    2019-05-27 09:00:16.113643096 (+00:00:00:006484189):tmux: build
    cd /root/ove/tmux && /root/ove/ove-tutorial/projects/tmux/build
    2019-05-27 09:00:16.118796787 (+00:00:00:005153691):tmux: build: done
    2019-05-27 09:00:16.124239108 (+00:00:00:005442321):tmux: install
    cd /root/ove/tmux && /root/ove/ove-tutorial/projects/tmux/install
    2019-05-27 09:00:16.129281178 (+00:00:00:005042070):tmux: install: done

When making this tutorial we used an Ubuntu 18.04 [LXC container](https://us.images.linuxcontainers.org), hence the "/root/" path.

As seen above the '**buildme**' command will check for any missing packages (specified by the 'needs:' in the 'projs' file), then iterate through any present bootstrap/configure/build/install steps for each project (as defined in 'ove-tutorial/projects/') in the correct build order. Please note, "missing packages" in this context refers to packages needed to build ove-tutorial and have nothing to do with packages needed for OVE itself.

If you are on Ubuntu/Debian, you could try to build everything without 'dry-run':

    $ ove dry-run 0
    $ ove buildme
    # go grab a cup of tea
    $ stage/usr/bin/tmux -V
    tmux 2.9

Wait! What's up with the 'stage/usr/bin...' stuff? A short note: OVE uses a staging area for both intermediate and final build steps. You can look at it as a per-workspace mirror of how the included projects would install on your host if built without OVE. It is up to each project to decide what to do with the end results. Typically an OVE plugin (see Part IV, Plugins) would package relevant output (found in stage/...) into, well.. packages.

## Part III: OVE commands

As mentioned [here](https://github.com/Ericsson/ove), OVE commands are divided into four categories:

* High level git commands
* Build related commands
* Utility commands
* Plugins

In this part, we will give a brief introduction to a few useful commands. Try them out! Then go ahead and investigate the full list using the first command below. A majority of the commands are tab completion enabled.

    $ ove list-commands
    add                    [GIT...]                   git add -p for all/specified repositories
    ag                     PATTERN                    search using The Silver Searcher
    ahead                  [GIT...]                   list local commits not yet published for all/specified repositories
    am                     FILE                       apply a bz2 archive file created with 'format-patch'
    apply                  PATCH                      apply one OVE patch
    authors                                           summarize authors from tracked repositories
    blame-history          PATTERN                    git log -S for all git repositories
    blame                  PATTERN                    git grep-blame-log combo
    bootstrap-parallel     [PROJECT...]               run the 'bootstrap' step for all or individual projects (in parallel)
    bootstrap              [PROJECT...]               run the 'bootstrap' step for all or individual projects
    ...

    # ask for help for a particular command
    $ ove help add
    add                    [GIT...]                   git add -p for all/specified repositories

    # what is the current status of this project?
    $ ove status
    .ove          master   ## master...origin/master
    codechecker   master   ## master...origin/master
    dmce          master   ## master...origin/master
    ove-tutorial  d78b9d9  ## master...origin/master
    tmux          2.9      ## HEAD (no branch)

    # update some files
    $ sed -i '1i Hi all' */{README,README.md,docs/README.md}

    # what files got updated?
    $ ove status
    .ove          master   ## master...origin/master
    codechecker   master   ## master...origin/master  M docs/README.md
    dmce          master   ## master...origin/master  M README.md
    ove-tutorial  d78b9d9  ## master...origin/master  M README.md
    tmux          2.9      ## HEAD (no branch)  M README

    # open all modified files in 'vi'
    $ ove vi
    ...

    # interactively ask user (using git add -p) what chunks to add in 'codechecker' and 'dmce'
    $ ove add codechecker dmce
    ...

    # create commits for all staged changes
    $ ove commit
    ...

    $ ove status
    .ove          master   ## master...origin/master
    codechecker   master   ## master...origin/master [ahead 1]
    dmce          master   ## master...origin/master [ahead 1]
    ove-tutorial  d78b9d9  ## master...origin/master M README.md
    tmux          2.9      ## HEAD (no branch)  M README

    # check for new commits in a specific git repo
    $ ove fetch codechecker
    Fetching origin for codechecker
    ...
       f6ea954a..5c2b56bb  master     -> origin/master
    codechecker  ## master...origin/master [ahead 1, behind 2]

    # hmm, what got pushed upstream?
    $ ove news
    codechecker: 2 new commit(s):
    5c2b56bb 84 minutes ago  email-A  subject X
    2d7eeebd  2 hours ago    email-B  subject Y

    # take a look at one of these commits
    $ ove show 2d7<tab>
    $ ove show 2d7eeebd

    # rebase codechecker
    $ ove pull codechecker
    codechecker
    First, rewinding head to replay your work on top of it...
    Applying: Hi all
    codechecker   master   ## master...origin/master [ahead 1]

OVE does not have any "push" command(s). When you want to share these patches upstream, use the normal git workflow!

By now you should have a brief idea what kind of functionality OVE offers out-of-the box. Of course this is most often not enough! Let's move on to see how simple it is to add customized commands. We call them plugins!

## Part IV: Plugins

It is pretty easy to extend OVE with extra commands. In this section we will show how [CodeChecker](https://github.com/Ericsson/codechecker) can be added to this project.

When you do '**source ove**', OVE will search for executable files at the following three directories:

    $OVE_BASE_DIR/scripts/
    $OVE_PROJECT_DIR/scripts/
    <all repositories>/.ove/scripts/

All executable files in these three directories will be added to OVE command list. Here we added an entry point for codechecker in the top repo (ove-tutorial):

    $ ls $OVE_PROJECT_DIR/scripts
    codechecker  codechecker.complete  codechecker.help  dmce  dmce.complete  dmce.help
    $ ove help codechecker
    codechecker            [PROJECT]                  run codechecker for all/specified project(s)


| File                 | Description               |
|:---------------------|:--------------------------|
| codechecker          | Entry point               |
| codechecker.complete | Bash completion script    |
| codechecker.help     | Help text for codechecker |

Now we can run 'codechecker' like this:

    $ ove codechecker <tab><tab>
    dmce libevent ncurses tmux

    $ ove codechecker libevent
    CodeChecker for libevent...
    [INFO 2019-05-27 09:01] - Starting build ...
    2019-05-27 09:01:55.916308958 (+00:00:00:000000000):libevent: clean
    2019-05-27 09:01:55.975411897 (+00:00:00:059102939):libevent: clean: done
    2019-05-27 09:01:55.981158479 (+00:00:00:005746582):libevent: build
    2019-05-27 09:01:58.821146844 (+00:00:02:839988365):libevent: build: done
    ...
    [INFO 2019-05-27 09:01] - Starting static analysis ...
    [INFO 2019-05-27 09:01] - [1/168] clangsa analyzed evthread_pthread.c successfully.
    ...
    -----------------------
    Severity | Report count
    -----------------------
    STYLE    |         2501
    LOW      |          690
    MEDIUM   |           52
    HIGH     |           25
    -----------------------
    ----=================----
    Total number of reports: 3268
    ----=================----
    CodeChecker: libevent OK

## Part V: System tests

Finally, let us have a brief look at how system tests work. We prepared some simple test entry points. Please list them:

    $ ove list-systests
    all
    sanity
    t1
    t2
    t3

'all' and 'sanity' are system test groups, and 't1', 't2' and 't3' are actual system test entrypoints. They are defined in 'systests' and 'systests-groups' found in your OWEL (ove-tutorial):

    $ cat ove-tutorial/systests

    # name       timeout (s)   type   path   command
    # ----------------------------------------------
    t1              5          0      dmce  "sleep 4"
    t2              1          0      dmce  "echo Hello"
    t3              3          0      dmce  "sleep 10"

    $ cat ove-tutorial/systests-groups
    all:
      - t1
      - t2
      - t3
    sanity:
      - t1


You can try them out:

    $ ove systest t1
    ove-systest:[001/001    ]:t1
    ove-systest:[001/001 OK]:t1


    OVE log: /tmp/$USER/ove/logs/fab64a1/20190607-155603496153257-ove-systest-elxa3wb5lz1.log

    $ ove systest t2
    ove-systest:[001/001    ]:t2
    Hello
    ove-systest:[001/001 OK]:t2


    OVE log: /tmp/$USER/ove/logs/fab64a1/20190607-155613260889322-ove-systest-elxa3wb5lz1.log

    $ove systest t3
    ove-systest:[001/001    ]:t3
    ove-systest:[001/001 NOK TIMEOUT]:t3


    OVE log: /tmp/$USER/ove/logs/fab64a1/20190607-155616418011546-ove-systest-elxa3wb5lz1.log

The first two tests will pass and the third will fail. The reason for failing the third test is of course the timeout being shorter than the argument to sleep.

That's it! You are now ready to start using OVE. And remember, if you write/use a plugin that you believe should go into OVE, let's get in touch!
