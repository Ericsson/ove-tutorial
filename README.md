https://user-images.githubusercontent.com/25057211/120758396-1528a300-c512-11eb-82c8-ec479a800102.mp4

# OVE Tutorial

This tutorial will hopefully give you an basic idea what OVE is about. If you are looking for a more in-depth description of OVE, please head over to [OVE](https://github.com/Ericsson/ove). If you on the other hand are eager to take OVE for a spin right away, just go ahead with the instructions below, it's easy!

## Part I: Set up the OVE workspace

Run the oneliner:

    curl -sSL https://raw.githubusercontent.com/Ericsson/ove/master/setup | bash -s ove-workspace https://github.com/Ericsson/ove-tutorial

'ove-workspace' will be the name of your OVE workspace. You can use multiple workspaces to keep states of different tasks for the same or different projects.


For the interested, the above oneliner runs the code in the snippet below. PLEASE NOTE: Do not manually run them, they are only there to shine light over the oneliner you have already executed. Most people do not really have to care about what the setup script is doing so let us skip below code snippet and move on!

    $ # short description of what the oneliner setup script is actually doing
    $ #mkdir -p ove-workspace
    $ #cd ove-workspace
    $ #git clone -q https://github.com/Ericsson/ove-tutorial
    $ #git clone -q https://github.com/Ericsson/ove.git .ove
    $ #ln -s ove-tutorial .owel
    $ #ln -s .ove/ove ove

The '**setup**' script will now ask you to enter the '**ove-workspace**' directory and run '**source ove**', please do so. You should observe the following:

    $ source ove
    OVE 8e1b1f1 | Ubuntu 22.04 | GNU/Linux

If you are missing some of the required dependencies on your host, OVE will let you know and suggest a solution:

    $ source ove
    error: missing command(s):

        ld

    To fix this, run the following command:

        apt install binutils

Now, run '**ove fetch**' to clone the rest of the repositories:

    $ ove fetch
    Cloning into 'ag'...
    Cloning into 'codechecker'...
    Cloning into 'dmce'...
    Cloning into 'libevent'...
    Cloning into 'ncurses'...
    Cloning into 'tmux'...
    Cloning into 'xcm'...
    ...
    .ove          master                ## master...origin/master
    ag            master                ## master...origin/master
    codechecker   v6.12.1               ## HEAD (no branch)
    dmce          master                ## master...origin/master
    libevent      release-2.1.10-stable ## HEAD (no branch)
    ncurses       v6.1                  ## HEAD (no branch)
    ove-tutorial  f038c46               ## master...origin/master
    tmux          3.2a                  ## HEAD (no branch)
    xcm           master                ## master...origin/master

Checking the content of your newly created workspace will reveal the source code repos (codechecker, dmce, tmux, xcm), the OWEL (ove-tutorial) and OVE itself:

    $ ls -A
    ag  codechecker  dmce  libevent ncurses ove  .ove  .ove.state  .ove.tmp  ove-tutorial  .owel  README  tmux  xcm

The first part of the tutorial is done! You have enhanced your bash shell with OVE functionality, set up the top git repo (OWEL) and fetched the included source git repos. This covers the basics of versioning. Let us move on and build stuff!

## Part II: Build

Your OVE workspace now consists of nine git repositories (all from github.com) and ten projects:

    $ ove list-repositories
    .ove          https://github.com/Ericsson/ove.git                https://github.com/Ericsson/ove.git                master
    ag            https://github.com/ggreer/the_silver_searcher.git  https://github.com/ggreer/the_silver_searcher.git  master
    codechecker   https://github.com/Ericsson/codechecker.git        https://github.com/Ericsson/codechecker.git        v6.12.1
    dmce          https://github.com/PatrikAAberg/dmce.git           git@github.com:PatrikAAberg/dmce.git               master
    libevent      https://github.com/libevent/libevent               https://github.com/libevent/libevent               release-2.1.10-stable
    ncurses       https://github.com/mirror/ncurses.git              https://github.com/mirror/ncurses.git              v6.1
    ove-tutorial  https://github.com/Ericsson/ove-tutorial           https://github.com/Ericsson/ove-tutorial           master
    tmux          https://github.com/tmux/tmux.git                   https://github.com/tmux/tmux.git                   3.2a
    xcm           https://github.com/Ericsson/xcm.git                https://github.com/Ericsson/xcm.git                master

    $ ove list-projects
    ag
    base
    clang_tools
    codechecker
    dmce
    foo
    libevent
    ncurses
    tmux
    xcm

For OVE, a project is something that produces output (e.g. an executable, a library or anything else machine-made). Even though projects are normally contained within a corresponding git repo, OVE treats projects and repos independently. Multiple projects can be configured using code and build systems from the same repo, and one project can use code and build systems from multiple repos.
OVE keeps track of the dependencies between projects, so let us check the build order for the tutorial OWEL:

    $ ove build-order
    foo base libevent ncurses clang_tools xcm tmux dmce codechecker ag

    $ ove digraph
                        ┌─────┐     ┌─────────────┐
                        │ ag  │     │ clang_tools │ ◀────────────────────┐
                        └─────┘     └─────────────┘                      │
                          │                                              │
                          │                                              │
                          ▼                                              │
    ┌─────────────┐     ┌───────────────────────────────────┐            │
    │ codechecker │ ──▶ │                                   │            │
    └─────────────┘     │               base                │            │
                        │                                   │            │
      ┌───────────────▶ │                                   │ ◀┐         │
      │                 └───────────────────────────────────┘  │         │
      │                   ▲           ▲                   ▲    │         │
      │                   │           │                   │    │         │
      │                   │           │                   │    │         │
    ┌─────────────┐     ┌─────┐     ┌─────────────┐       │    │         │
    │   ncurses   │     │ xcm │     │    tmux     │ ─┐    │    │         │
    └─────────────┘     └─────┘     └─────────────┘  │    │    │         │
      ▲                               │              │    │    │         │
      │                          ┌────┘              └────┼────┼────┐    │
      │                          │                        │    │    │    │
      │                          │  ┌─────────────┐       │    │    │    │
      │                          │  │    dmce     │ ──────┘    │    │    │
      │                          │  └─────────────┘            │    │    │
      │                          │    │                        │    │    │
      │                          │    └────────────────────────┼────┼────┘
      │                          │                             │    │
      │                          │  ┌─────────────┐            │    │
      │                          └▶ │  libevent   │ ───────────┘    │
      │                             └─────────────┘                 │
      │                                                             │
      └─────────────────────────────────────────────────────────────┘
                        ┌─────┐
                        │ foo │
                        └─────┘

All projects except 'foo' are dependent on '**base**' (=gcc/make/... etc.).

Let us do a 'dry-run' build first:

    $ ove dry-run
    OVE_DRY_RUN 1

    $ ove buildme
    2023-06-28 07:57:57.465737578 (+00:00:00:000000000):would prompt to install this/these package(s):
    clang
    clang-tidy
    doxygen
    gcc-multilib
    python3-flake8
    python3-virtualenv
    thrift-compiler
    2023-06-28 07:57:57.486130598 (+00:00:00:020393020):foo: bootstrap
    cd /root/ove-workspace && /root/ove-workspace/ove-tutorial/projects/foo/bootstrap
    2023-06-28 07:57:57.497634672 (+00:00:00:011504074):foo: bootstrap: done
    2023-06-28 07:57:57.523224657 (+00:00:00:025589985):libevent: bootstrap
    cd /root/ove-workspace/libevent && /root/ove-workspace/ove-tutorial/projects/libevent/bootstrap
    2023-06-28 07:57:57.537305358 (+00:00:00:014080701):libevent: bootstrap: done
    2023-06-28 07:57:57.573480150 (+00:00:00:036174792):ncurses: bootstrap
    cd /root/ove-workspace/ncurses && /root/ove-workspace/ove-tutorial/projects/ncurses/bootstrap
    2023-06-28 07:57:57.593838475 (+00:00:00:020358325):ncurses: bootstrap: done
    2023-06-28 07:57:57.624221403 (+00:00:00:030382928):clang_tools: bootstrap
    cd /root/ove-workspace && /root/ove-workspace/ove-tutorial/projects/clang_tools/bootstrap
    2023-06-28 07:57:57.648113352 (+00:00:00:023891949):clang_tools: bootstrap: done
    2023-06-28 07:57:57.690405995 (+00:00:00:042292643):xcm: bootstrap
    cd /root/ove-workspace/xcm && /root/ove-workspace/ove-tutorial/projects/xcm/bootstrap
    2023-06-28 07:57:57.715385996 (+00:00:00:024980001):xcm: bootstrap: done
    2023-06-28 07:57:57.757861023 (+00:00:00:042475027):tmux: bootstrap
    cd /root/ove-workspace/tmux && /root/ove-workspace/ove-tutorial/projects/tmux/bootstrap
    2023-06-28 07:57:57.781341146 (+00:00:00:023480123):tmux: bootstrap: done
    2023-06-28 07:57:57.830047310 (+00:00:00:048706164):ag: bootstrap
    cd /root/ove-workspace/ag && /root/ove-workspace/ove-tutorial/projects/ag/bootstrap
    2023-06-28 07:57:57.854820081 (+00:00:00:024772771):ag: bootstrap: done
    2023-06-28 07:57:57.894485731 (+00:00:00:039665650):foo: configure
    cd /root/ove-workspace && /root/ove-workspace/ove-tutorial/projects/foo/configure
    2023-06-28 07:57:57.912071286 (+00:00:00:017585555):foo: configure: done
    2023-06-28 07:57:57.935864835 (+00:00:00:023793549):foo: build
    cd /root/ove-workspace && /root/ove-workspace/ove-tutorial/projects/foo/build
    2023-06-28 07:57:57.950661167 (+00:00:00:014796332):foo: build: done
    2023-06-28 07:57:57.991995814 (+00:00:00:041334647):foo: install
    cd /root/ove-workspace && /root/ove-workspace/ove-tutorial/projects/foo/install
    2023-06-28 07:57:58.017511460 (+00:00:00:025515646):foo: install: done
    2023-06-28 07:57:58.135094336 (+00:00:00:117582876):libevent: configure
    cd /root/ove-workspace/libevent && /root/ove-workspace/ove-tutorial/projects/libevent/configure
    2023-06-28 07:57:58.156072059 (+00:00:00:020977723):libevent: configure: done
    2023-06-28 07:57:58.202909556 (+00:00:00:046837497):libevent: build
    cd /root/ove-workspace/libevent && /root/ove-workspace/ove-tutorial/projects/libevent/build
    2023-06-28 07:57:58.228188769 (+00:00:00:025279213):libevent: build: done
    2023-06-28 07:57:58.280143943 (+00:00:00:051955174):libevent: install
    cd /root/ove-workspace/libevent && /root/ove-workspace/ove-tutorial/projects/libevent/install
    2023-06-28 07:57:58.307638653 (+00:00:00:027494710):libevent: install: done
    2023-06-28 07:57:58.342913605 (+00:00:00:035274952):ncurses: configure
    cd /root/ove-workspace/ncurses && /root/ove-workspace/ove-tutorial/projects/ncurses/configure
    2023-06-28 07:57:58.367128048 (+00:00:00:024214443):ncurses: configure: done
    2023-06-28 07:57:58.406652265 (+00:00:00:039524217):ncurses: build
    cd /root/ove-workspace/ncurses && /root/ove-workspace/ove-tutorial/projects/ncurses/build
    2023-06-28 07:57:58.430949070 (+00:00:00:024296805):ncurses: build: done
    2023-06-28 07:57:58.483275764 (+00:00:00:052326694):ncurses: install
    cd /root/ove-workspace/ncurses && /root/ove-workspace/ove-tutorial/projects/ncurses/install
    2023-06-28 07:57:58.508471424 (+00:00:00:025195660):ncurses: install: done
    2023-06-28 07:57:58.633004337 (+00:00:00:124532913):xcm: configure
    cd /root/ove-workspace/xcm && /root/ove-workspace/ove-tutorial/projects/xcm/configure
    2023-06-28 07:57:58.660034355 (+00:00:00:027030018):xcm: configure: done
    2023-06-28 07:57:58.706425901 (+00:00:00:046391546):xcm: build
    cd /root/ove-workspace/xcm && /root/ove-workspace/ove-tutorial/projects/xcm/build
    2023-06-28 07:57:58.730215522 (+00:00:00:023789621):xcm: build: done
    2023-06-28 07:57:58.781834814 (+00:00:00:051619292):xcm: install
    cd /root/ove-workspace/xcm && /root/ove-workspace/ove-tutorial/projects/xcm/install
    2023-06-28 07:57:58.803236850 (+00:00:00:021402036):xcm: install: done
    2023-06-28 07:57:58.828626157 (+00:00:00:025389307):tmux: configure
    cd /root/ove-workspace/tmux && /root/ove-workspace/ove-tutorial/projects/tmux/configure
    2023-06-28 07:57:58.848573200 (+00:00:00:019947043):tmux: configure: done
    2023-06-28 07:57:58.881368780 (+00:00:00:032795580):tmux: build
    cd /root/ove-workspace/tmux && /root/ove-workspace/ove-tutorial/projects/tmux/build
    2023-06-28 07:57:58.902492454 (+00:00:00:021123674):tmux: build: done
    2023-06-28 07:57:58.951949294 (+00:00:00:049456840):tmux: install
    cd /root/ove-workspace/tmux && /root/ove-workspace/ove-tutorial/projects/tmux/install
    2023-06-28 07:57:58.978174946 (+00:00:00:026225652):tmux: install: done
    2023-06-28 07:57:59.075065176 (+00:00:00:096890230):dmce: install
    cd /root/ove-workspace/dmce && /root/ove-workspace/ove-tutorial/projects/dmce/install
    2023-06-28 07:57:59.098631546 (+00:00:00:023566370):dmce: install: done
    2023-06-28 07:57:59.175210899 (+00:00:00:076579353):codechecker: build
    cd /root/ove-workspace/codechecker && /root/ove-workspace/ove-tutorial/projects/codechecker/build
    2023-06-28 07:57:59.203807210 (+00:00:00:028596311):codechecker: build: done
    2023-06-28 07:57:59.273495655 (+00:00:00:069688445):ag: configure
    cd /root/ove-workspace/ag && /root/ove-workspace/ove-tutorial/projects/ag/configure
    2023-06-28 07:57:59.298307961 (+00:00:00:024812306):ag: configure: done
    2023-06-28 07:57:59.347068286 (+00:00:00:048760325):ag: build
    cd /root/ove-workspace/ag && /root/ove-workspace/ove-tutorial/projects/ag/build
    2023-06-28 07:57:59.370737274 (+00:00:00:023668988):ag: build: done
    2023-06-28 07:57:59.422126383 (+00:00:00:051389109):ag: install
    cd /root/ove-workspace/ag && /root/ove-workspace/ove-tutorial/projects/ag/install
    2023-06-28 07:57:59.448146998 (+00:00:00:026020615):ag: install: done

When making this tutorial we used a [LXC container](https://us.images.linuxcontainers.org), hence the "/root/" path.

As seen above the '**buildme**' command will check for any missing packages (specified by the 'needs:' in the 'projs' file), then iterate through any present bootstrap/configure/build/install steps for each project (as defined in 'ove-tutorial/projects/') in the correct build order. Please note, "missing packages" in this context refers to packages needed to build ove-tutorial and have nothing to do with packages needed for OVE itself.

If you are on Ubuntu/Debian, you could try to build everything without 'dry-run':

    $ ove dry-run 0
    $ ove buildme tmux
    # go grab a cup of tea
    $ which tmux
    /root/ove-workspace/stage/usr/bin/tmux
    $ tmux -V
    tmux 3.2a

Wait! What's up with the 'stage/usr/bin...' stuff? A short note: OVE uses a staging area for both intermediate and final build steps. You can look at it as a per-workspace mirror of how the included projects would install on your host if built without OVE. It is up to each project to decide what to do with the end results. Typically an OVE plugin (see Part IV, Plugins) would package relevant output (found in stage/...) into, well.. packages.

## Part III: OVE commands

As mentioned [here](https://github.com/Ericsson/ove), OVE commands are divided into four categories:

* High level git commands
* Build related commands
* Utility commands
* Plugins

In this part, we will give a brief introduction to a few useful commands. Try them out! Then go ahead and investigate the full list using the first command below. A majority of the commands are tab completion enabled.

    $ ove list-commands
    cmd           args                                                       help                                                                   category
    ---
    add-config    file config value                                          add config/value pair to one OVE config file                           CORE
    add-project   [name path]|[name path cmd...]|[name path cmd@content...]  add one project to 'projs' and 'projects/...'                          CORE
    add-repo      dir|url|url name|url name rev                              add a new repo to this OVE workspace                                   CORE
    add           [git...]                                                   git add for all/specified repositories                                 CORE
    ag            pattern                                                    search OVE workspace using ag [duckduckgo.com/?q=The+Silver+Searcher]  SEARCH
    ahead         [git...]                                                   list local commits not yet published for all/specified repositories    CORE
    am            file                                                       apply a bz2 archive file created with 'format-patch'                   CORE
    apply-cached  patch                                                      apply one OVE patch to index                                           CORE
    apply         patch                                                      apply one OVE patch                                                    CORE
    authors                                                                  list author summary for all git repositories                           UTIL
    ...

    # list a particular command
    $ ove list-commands news
    cmd        args      help                                                               category
    ---
    news       [git...]  list upstream changes for all/specified repositories               CORE
    show-news  [git...]  run 'ove show' on upstream changes for all/specified repositories  CORE

    # what is the current status of this project?
    $ ove status
    .ove          master                 ## master...origin/master
    ag            master                 ## master...origin/master
    codechecker   v6.12.1                ## HEAD (no branch)
    dmce          master                 ## master...origin/master
    libevent      release-2.1.10-stable  ## HEAD (no branch)
    ncurses       v6.1                   ## HEAD (no branch)
    ove-tutorial  f038c46                ## master...origin/master
    tmux          3.2a                   ## HEAD (no branch)
    xcm           master                 ## master...origin/master

    # update some files
    $ sed -i '1i Hi all' */{README,README.md,docs/README.md}

    # what files got updated?
    $ ove status
    .ove          master                 ## master...origin/master
    ag            master                 ## master...origin/master M README.md
    codechecker   v6.12.1                ## HEAD (no branch) M docs/README.md
    dmce          master                 ## master...origin/master M README.md
    libevent      release-2.1.10-stable  ## HEAD (no branch) M README.md
    ncurses       v6.1                   ## HEAD (no branch) M README
    ove-tutorial  f038c46                ## master...origin/master M README.md
    tmux          3.2a                   ## HEAD (no branch) M README
    xcm           master                 ## master...origin/master M README.md

    # open all modified files in 'vi'
    $ ove vi
    ...

    # interactively ask user (using git add -p) what chunks to add in repo 'dmce'
    $ ove add dmce
    ...

    # create a commit for the staged changes in 'dmce'
    $ ove commit
    ...

    $ ove status dmce
    dmce          master  ## master...origin/master [ahead 1]

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
    $OVE_OWEL_DIR/scripts/
    <all repositories>/.ove/scripts/

All executable files in these three directories will be added to OVE command list. Here we added an entry point for codechecker in the top repo (ove-tutorial):

    $ ls $OVE_OWEL_DIR/scripts
    codechecker  codechecker.complete  codechecker.help  dmce  dmce.complete  dmce.help
    $ ove help codechecker
    codechecker [PROJECT]    run codechecker for all/specified project(s)   PLUGIN


| File                 | Description               |
|:---------------------|:--------------------------|
| codechecker          | Entry point               |
| codechecker.complete | Bash completion script    |
| codechecker.help     | Help text for codechecker |

Now we can run 'codechecker' like this:

    $ ove buildme codechecker
    ...

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
