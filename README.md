https://user-images.githubusercontent.com/25057211/120758396-1528a300-c512-11eb-82c8-ec479a800102.mp4

# OVE Tutorial

This tutorial will hopefully give you an basic idea what OVE is about. If you are looking for a more in-depth description of OVE, please head over to [OVE](https://github.com/Ericsson/ove). If you on the other hand are eager to take OVE for a spin right away, just go ahead with the instructions below, it's easy!

## Part I: Set up the OVE workspace

Run the oneliner:

    curl -sSL https://raw.githubusercontent.com/Ericsson/ove/master/setup | bash -s my-ove-workspace https://github.com/Ericsson/ove-tutorial

'my-ove-workspace' will be the name of your OVE workspace. You can use multiple workspaces to keep states of different tasks for the same or different projects.


For the interested, the above oneliner runs the code in the snippet below. PLEASE NOTE: Do not manually run them, they are only there to shine light over the oneliner you have already executed. Most people do not really have to care about what the setup script is doing so let us skip below code snippet and move on!

    $ # short description of what the oneliner setup script is actually doing
    $ #mkdir -p ove
    $ #cd ove
    $ #git clone -q https://github.com/Ericsson/ove-tutorial
    $ #git clone -q https://github.com/Ericsson/ove.git .ove
    $ #ln -s ove-tutorial .owel
    $ #ln -s .ove/ove ove

The '**setup**' script will now ask you to enter the '**my-ove-workspace**' directory and run '**source ove**', please do so. You should observe the following:

    $ source ove
    OVE [SHA-1: 04c8b0b @ Ubuntu 18.04]

If you are missing some of the required dependencies on your host, OVE will let you know and suggest a solution:

    $ source ove
    error: missing command(s):

        ld

    To fix this, run the following command:

        apt install binutils

Now, run '**ove fetch**' to clone the rest of the repositories:

    $ ove fetch
    Cloning into 'codechecker'...
    Cloning into 'dmce'...
    Cloning into 'tmux'...
    Cloning into 'xcm'...
    ...
    .ove          master   ## master...origin/master
    codechecker   v6.12.1  ## HEAD (no branch)
    dmce          master   ## master...origin/master
    ove-tutorial  d78b9d9  ## master...origin/master
    tmux          2.9      ## HEAD (no branch)
    xcm           master   ## master...origin/master

Checking the content of your newly created workspace will reveal the source code repos (codechecker, dmce, tmux, xcm), the OWEL (ove-tutorial) and OVE itself:

    $ ls -A
    codechecker  dmce  ove  .ove  .ove.state  .ove.tmp  ove-tutorial  .owel  README  tmux  xcm

The first part of the tutorial is done! You have enhanced your bash shell with OVE functionality, set up the top git repo (OWEL) and fetched the included source git repos. This covers the basics of versioning. Let us move on and build stuff!

## Part II: Build

Your OVE workspace now consists of six git repositories (all from github.com) and six projects:

    $ ove list-repositories
    .ove          https://github.com/Ericsson/ove.git          https://github.com/Ericsson/ove.git          master
    codechecker   https://github.com/Ericsson/codechecker.git  https://github.com/Ericsson/codechecker.git  v6.12.1
    dmce          https://github.com/PatrikAAberg/dmce.git     git@github.com:PatrikAAberg/dmce.git         master
    ove-tutorial  https://github.com/Ericsson/ove-tutorial.git https://github.com/Ericsson/ove-tutorial.git master
    tmux          https://github.com/tmux/tmux.git             https://github.com/tmux/tmux.git             2.9
    xcm           https://github.com/Ericsson/xcm.git          https://github.com/tmux/xcm.git              master

    $ ove list-projects
    base
    codechecker
    dmce
    libevent
    ncurses
    tmux
    xcm


For OVE, a project is something that produces output (e.g. an executable, a library or anything else machine-made). Even though projects are normally contained within a  corresponding git repo, OVE treats projects and repos independently. Multiple projects can be configured using code and build systems from the same repo, and one project can use code and build systems from multiple repos.
OVE keeps track of the dependencies between projects, so let us check the build order for the tutorial OWEL:

    $ ove build-order
    base libevent ncurses xcm tmux dmce codechecker

    $ ove digraph
    digraph ove_tutorial {
        base;
        codechecker -> base;
        dmce -> base;
        libevent -> base;
        ncurses -> base;
        tmux -> base;
        tmux -> libevent;
        tmux -> ncurses;
        xcm -> base;
    }

All projects are dependent on '**base**' (=gcc/make/... etc.).

Let us do a 'dry-run' build first:

    $ ove dry-run
    OVE_DRY_RUN 1

    $ ove buildme
    2019-05-27 09:00:15.986867812 (+00:00:00:000000000):Would prompt to install this/these package(s):
    autoconf
    automake
    bc
    bison
    build-essential
    clang
    clang-tidy
    clang-tools
    curl
    doxygen
    gcc-multilib
    graphviz
    libtool
    make
    perl
    pkg-config
    python3
    python3-dev
    python3-setuptools
    python3-virtualenv
    rsync
    thrift-compiler

    2020-09-14 08:51:20.000446089 (+00:01:48:926002802):dmce: bootstrap
    cd /root/ove/ove-tutorial/dmce && /root/ove/ove-tutorial/ove-tutorial/projects/dmce/bootstrap
    2020-09-14 08:51:20.008529897 (+00:00:00:008083808):dmce: bootstrap: done
    2020-09-14 08:51:20.023348560 (+00:00:00:014818663):xcm: bootstrap
    cd /root/ove/ove-tutorial/xcm && /root/ove/ove-tutorial/ove-tutorial/projects/xcm/bootstrap
    2020-09-14 08:51:20.035435920 (+00:00:00:012087360):xcm: bootstrap: done
    2020-09-14 08:51:20.049928310 (+00:00:00:014492390):libevent: bootstrap
    cd /root/ove/ove-tutorial/libevent && /root/ove/ove-tutorial/ove-tutorial/projects/libevent/bootstrap
    2020-09-14 08:51:20.060426359 (+00:00:00:010498049):libevent: bootstrap: done
    2020-09-14 08:51:20.077421133 (+00:00:00:016994774):ncurses: bootstrap
    cd /root/ove/ove-tutorial/ncurses && /root/ove/ove-tutorial/ove-tutorial/projects/ncurses/bootstrap
    2020-09-14 08:51:20.091968697 (+00:00:00:014547564):ncurses: bootstrap: done
    2020-09-14 08:51:20.109279194 (+00:00:00:017310497):tmux: bootstrap
    cd /root/ove/ove-tutorial/tmux && /root/ove/ove-tutorial/ove-tutorial/projects/tmux/bootstrap
    2020-09-14 08:51:20.126547583 (+00:00:00:017268389):tmux: bootstrap: done
    2020-09-14 08:51:20.148767522 (+00:00:00:022219939):codechecker: build
    cd /root/ove/ove-tutorial/codechecker && /root/ove/ove-tutorial/ove-tutorial/projects/codechecker/build
    2020-09-14 08:51:20.162019327 (+00:00:00:013251805):codechecker: build: done
    2020-09-14 08:51:20.189839073 (+00:00:00:027819746):xcm: configure
    cd /root/ove/ove-tutorial/xcm && /root/ove/ove-tutorial/ove-tutorial/projects/xcm/configure
    2020-09-14 08:51:20.199349649 (+00:00:00:009510576):xcm: configure: done
    2020-09-14 08:51:20.214720988 (+00:00:00:015371339):xcm: build
    cd /root/ove/ove-tutorial/xcm && /root/ove/ove-tutorial/ove-tutorial/projects/xcm/build
    2020-09-14 08:51:20.231518377 (+00:00:00:016797389):xcm: build: done
    2020-09-14 08:51:20.250685616 (+00:00:00:019167239):xcm: install
    cd /root/ove/ove-tutorial/xcm && /root/ove/ove-tutorial/ove-tutorial/projects/xcm/install
    2020-09-14 08:51:20.266696846 (+00:00:00:016011230):xcm: install: done
    2020-09-14 08:51:20.284269540 (+00:00:00:017572694):libevent: configure
    cd /root/ove/ove-tutorial/libevent && /root/ove/ove-tutorial/ove-tutorial/projects/libevent/configure
    2020-09-14 08:51:20.301842634 (+00:00:00:017573094):libevent: configure: done
    2020-09-14 08:51:20.318503283 (+00:00:00:016660649):libevent: build
    cd /root/ove/ove-tutorial/libevent && /root/ove/ove-tutorial/ove-tutorial/projects/libevent/build
    2020-09-14 08:51:20.337544517 (+00:00:00:019041234):libevent: build: done
    2020-09-14 08:51:20.361461548 (+00:00:00:023917031):libevent: install
    cd /root/ove/ove-tutorial/libevent && /root/ove/ove-tutorial/ove-tutorial/projects/libevent/install
    2020-09-14 08:51:20.374159184 (+00:00:00:012697636):libevent: install: done
    2020-09-14 08:51:20.387922998 (+00:00:00:013763814):ncurses: configure
    cd /root/ove/ove-tutorial/ncurses && /root/ove/ove-tutorial/ove-tutorial/projects/ncurses/configure
    2020-09-14 08:51:20.403197721 (+00:00:00:015274723):ncurses: configure: done
    2020-09-14 08:51:20.414664176 (+00:00:00:011466455):ncurses: build
    cd /root/ove/ove-tutorial/ncurses && /root/ove/ove-tutorial/ove-tutorial/projects/ncurses/build
    2020-09-14 08:51:20.424169640 (+00:00:00:009505464):ncurses: build: done
    2020-09-14 08:51:20.437513878 (+00:00:00:013344238):ncurses: install
    cd /root/ove/ove-tutorial/ncurses && /root/ove/ove-tutorial/ove-tutorial/projects/ncurses/install
    2020-09-14 08:51:20.450696709 (+00:00:00:013182831):ncurses: install: done
    2020-09-14 08:51:20.468480452 (+00:00:00:017783743):tmux: configure
    cd /root/ove/ove-tutorial/tmux && /root/ove/ove-tutorial/ove-tutorial/projects/tmux/configure
    2020-09-14 08:51:20.484651391 (+00:00:00:016170939):tmux: configure: done
    2020-09-14 08:51:20.504082513 (+00:00:00:019431122):tmux: build
    cd /root/ove/ove-tutorial/tmux && /root/ove/ove-tutorial/ove-tutorial/projects/tmux/build
    2020-09-14 08:51:20.519958130 (+00:00:00:015875617):tmux: build: done
    2020-09-14 08:51:20.540685476 (+00:00:00:020727346):tmux: install
    cd /root/ove/ove-tutorial/tmux && /root/ove/ove-tutorial/ove-tutorial/projects/tmux/install
    2020-09-14 08:51:20.557312446 (+00:00:00:016626970):tmux: install: done

When making this tutorial we used an Ubuntu 18.04 [LXC container](https://us.images.linuxcontainers.org), hence the "/root/" path.

As seen above the '**buildme**' command will check for any missing packages (specified by the 'needs:' in the 'projs' file), then iterate through any present bootstrap/configure/build/install steps for each project (as defined in 'ove-tutorial/projects/') in the correct build order. Please note, "missing packages" in this context refers to packages needed to build ove-tutorial and have nothing to do with packages needed for OVE itself.

If you are on Ubuntu/Debian, you could try to build everything without 'dry-run':

    $ ove dry-run 0
    $ ove buildme tmux
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
    add-config             file config value          add config/value pair to one oveconfig file
    add-repo               url|url name|url name rev  add a new repo to this OVE workspace
    add                    [git...]                   git add -p for all/specified repositories
    ag                     pattern                    search OVE workspace using The Silver Searcher [duckduckgo.com/?q=The+Silver+Searcher]
    ahead                  [git...]                   list local commits not yet published for all/specified repositories
    am                     file                       apply a bz2 archive file created with 'format-patch'
    apply                  patch                      apply one OVE patch
    authors                                           list author summary for all git repositories
    blame-history          pattern                    git log -G -p pattern for all git repositories
    blame                  pattern                    git grep-blame-log combo
    ...

    # ask for help for a particular command
    $ ove help news
    news                   [git...]                   list upstream changes for all/specified repositories
    ...

    # what is the current status of this project?
    $ ove status
    .ove          master   ## master...origin/master
    codechecker   v6.12.1  ## HEAD (no branch)
    dmce          master   ## master...origin/master
    ove-tutorial  d78b9d9  ## master...origin/master
    tmux          2.9      ## HEAD (no branch)
    xcm           master   ## master...origin/master

    # update some files
    $ sed -i '1i Hi all' */{README,README.md,docs/README.md}

    # what files got updated?
    $ ove status
    .ove          master   ## master...origin/master
    codechecker   v6.12.1  ## HEAD (no branch)  M docs/README.md
    dmce          master   ## master...origin/master  M README.md
    ove-tutorial  d78b9d9  ## master...origin/master  M README.md
    tmux          2.9      ## HEAD (no branch)  M README
    xcm           master   ## master...origin/master  M README.md

    # open all modified files in 'vi'
    $ ove vi
    ...

    # interactively ask user (using git add -p) what chunks to add in repo 'dmce'
    $ ove add dmce
    ...

    # create a commit for the staged changes in 'dmce'
    $ ove commit
    ...

    $ ove status
    .ove          master   ## master...origin/master
    codechecker   v6.12.1  ## HEAD (no branch)  M docs/README.md
    dmce          master   ## master...origin/master [ahead 1]
    ove-tutorial  d78b9d9  ## master...origin/master M README.md
    tmux          2.9      ## HEAD (no branch)  M README
    xcm           master   ## master...origin/master  M README.md

    # check for new commits in a specific git repo
    $ ove fetch dmce
    Fetching origin for dmce
    ...
       4aa5c56..d3400f5  master     -> origin/master
    dmce ## master...origin/master [ahead 1, behind 2]

    # hmm, what got pushed upstream?
    $ ove news
    dmce: 2 new commit(s):
    d3400f5 84 minutes ago  email-A  subject X
    d6c7e7c  2 hours ago    email-B  subject Y

    # take a look at one of these commits
    $ ove show d3<tab>
    $ ove show d3400f5

    # rebase dmce
    $ ove pull dmce
    dmce
    First, rewinding head to replay your work on top of it...
    Applying: Hi all
    dmce  master   ## master...origin/master [ahead 1]

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
    dmce libevent ncurses tmux xcm

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


    OVE log: /tmp/$USER/ove/logs/fab64a1/20190607-155603496153257-ove-systest-ahost.log

    $ ove systest t2
    ove-systest:[001/001    ]:t2
    Hello
    ove-systest:[001/001 OK]:t2


    OVE log: /tmp/$USER/ove/logs/fab64a1/20190607-155613260889322-ove-systest-ahost.log

    $ove systest t3
    ove-systest:[001/001    ]:t3
    ove-systest:[001/001 NOK TIMEOUT]:t3


    OVE log: /tmp/$USER/ove/logs/fab64a1/20190607-155616418011546-ove-systest-ahost.log

The first two tests will pass and the third will fail. The reason for failing the third test is of course the timeout being shorter than the argument to sleep.

That's it! You are now ready to start using OVE. And remember, if you write/use a plugin that you believe should go into OVE, let's get in touch!
