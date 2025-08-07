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
    codechecker   v6.24.4               ## HEAD (no branch)
    dmce          master                ## master...origin/master
    libevent      release-2.1.10-stable ## HEAD (no branch)
    ncurses       v6.1                  ## HEAD (no branch)
    ove-tutorial  d5727dc               ## master...origin/master
    tmux          3.5a                  ## HEAD (no branch)
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
    codechecker   https://github.com/Ericsson/codechecker.git        https://github.com/Ericsson/codechecker.git        v6.24.4
    dmce          https://github.com/PatrikAAberg/dmce.git           git@github.com:PatrikAAberg/dmce.git               master
    libevent      https://github.com/libevent/libevent               https://github.com/libevent/libevent               release-2.1.10-stable
    ncurses       https://github.com/mirror/ncurses.git              https://github.com/mirror/ncurses.git              v6.1
    ove-tutorial  https://github.com/Ericsson/ove-tutorial           https://github.com/Ericsson/ove-tutorial           master
    tmux          https://github.com/tmux/tmux.git                   https://github.com/tmux/tmux.git                   3.5a
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
    2024-10-31 07:09:19.482022 (+00:00:00:000000):would prompt to install this/these package(s):
    bison
    clang-tools
    libtool
    python3-colorama
    python3-flake8
    python3-numpy
    pushd /root/ove-workspace
    2024-10-31 07:09:19.508085 (+00:00:00:026063):foo: bootstrap
    eval foo_version= OVE_ACTIVE_PROJECT_NAME=foo OVE_ACTIVE_PROJECT_COMMAND=bootstrap OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/foo/bootstrap
    2024-10-31 07:09:19.512341 (+00:00:00:004256):foo: bootstrap: done
    popd
    pushd /root/ove-workspace/libevent
    2024-10-31 07:09:19.537189 (+00:00:00:024848):libevent: bootstrap
    eval libevent_version= OVE_ACTIVE_PROJECT_NAME=libevent OVE_ACTIVE_PROJECT_COMMAND=bootstrap OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/libevent/bootstrap
    2024-10-31 07:09:19.540876 (+00:00:00:003687):libevent: bootstrap: done
    popd
    pushd /root/ove-workspace/ncurses
    2024-10-31 07:09:19.562455 (+00:00:00:021579):ncurses: bootstrap
    eval ncurses_version= OVE_ACTIVE_PROJECT_NAME=ncurses OVE_ACTIVE_PROJECT_COMMAND=bootstrap OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/ncurses/bootstrap
    2024-10-31 07:09:19.566095 (+00:00:00:003640):ncurses: bootstrap: done
    popd
    pushd /root/ove-workspace
    2024-10-31 07:09:19.590822 (+00:00:00:024727):clang_tools: bootstrap
    eval clang_tools_version= OVE_ACTIVE_PROJECT_NAME=clang_tools OVE_ACTIVE_PROJECT_COMMAND=bootstrap OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/clang_tools/bootstrap
    2024-10-31 07:09:19.593911 (+00:00:00:003089):clang_tools: bootstrap: done
    popd
    pushd /root/ove-workspace/xcm
    2024-10-31 07:09:19.618531 (+00:00:00:024620):xcm: bootstrap
    eval xcm_version= OVE_ACTIVE_PROJECT_NAME=xcm OVE_ACTIVE_PROJECT_COMMAND=bootstrap OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/xcm/bootstrap
    2024-10-31 07:09:19.622520 (+00:00:00:003989):xcm: bootstrap: done
    popd
    pushd /root/ove-workspace/tmux
    2024-10-31 07:09:19.650178 (+00:00:00:027658):tmux: bootstrap
    eval tmux_version= OVE_ACTIVE_PROJECT_NAME=tmux OVE_ACTIVE_PROJECT_COMMAND=bootstrap OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/tmux/bootstrap
    2024-10-31 07:09:19.655555 (+00:00:00:005377):tmux: bootstrap: done
    popd
    pushd /root/ove-workspace/ag
    2024-10-31 07:09:19.680689 (+00:00:00:025134):ag: bootstrap
    eval ag_version= OVE_ACTIVE_PROJECT_NAME=ag OVE_ACTIVE_PROJECT_COMMAND=bootstrap OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/ag/bootstrap
    2024-10-31 07:09:19.684100 (+00:00:00:003411):ag: bootstrap: done
    popd
    pushd /root/ove-workspace
    2024-10-31 07:09:19.705437 (+00:00:00:021337):foo: configure
    eval foo_version= OVE_ACTIVE_PROJECT_NAME=foo OVE_ACTIVE_PROJECT_COMMAND=configure OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/foo/configure
    2024-10-31 07:09:19.708978 (+00:00:00:003541):foo: configure: done
    popd
    pushd /root/ove-workspace
    2024-10-31 07:09:19.734519 (+00:00:00:025541):foo: build
    eval foo_version= OVE_ACTIVE_PROJECT_NAME=foo OVE_ACTIVE_PROJECT_COMMAND=build OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/foo/build
    2024-10-31 07:09:19.739595 (+00:00:00:005076):foo: build: done
    popd
    pushd /root/ove-workspace
    2024-10-31 07:09:19.764393 (+00:00:00:024798):foo: install
    eval foo_version= OVE_ACTIVE_PROJECT_NAME=foo OVE_ACTIVE_PROJECT_COMMAND=install OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/foo/install
    2024-10-31 07:09:19.767326 (+00:00:00:002933):foo: install: done
    popd
    pushd /root/ove-workspace/libevent
    2024-10-31 07:09:19.868098 (+00:00:00:100772):libevent: configure
    eval libevent_version= OVE_ACTIVE_PROJECT_NAME=libevent OVE_ACTIVE_PROJECT_COMMAND=configure OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/libevent/configure
    2024-10-31 07:09:19.873167 (+00:00:00:005069):libevent: configure: done
    popd
    pushd /root/ove-workspace/libevent
    2024-10-31 07:09:19.896467 (+00:00:00:023300):libevent: build
    eval libevent_version= OVE_ACTIVE_PROJECT_NAME=libevent OVE_ACTIVE_PROJECT_COMMAND=build OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/libevent/build
    2024-10-31 07:09:19.901816 (+00:00:00:005349):libevent: build: done
    popd
    pushd /root/ove-workspace/libevent
    2024-10-31 07:09:19.923944 (+00:00:00:022128):libevent: install
    eval libevent_version= OVE_ACTIVE_PROJECT_NAME=libevent OVE_ACTIVE_PROJECT_COMMAND=install OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/libevent/install
    2024-10-31 07:09:19.930476 (+00:00:00:006532):libevent: install: done
    popd
    pushd /root/ove-workspace/ncurses
    2024-10-31 07:09:19.953568 (+00:00:00:023092):ncurses: configure
    eval ncurses_version= OVE_ACTIVE_PROJECT_NAME=ncurses OVE_ACTIVE_PROJECT_COMMAND=configure OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/ncurses/configure
    2024-10-31 07:09:19.956717 (+00:00:00:003149):ncurses: configure: done
    popd
    pushd /root/ove-workspace/ncurses
    2024-10-31 07:09:19.981461 (+00:00:00:024744):ncurses: build
    eval ncurses_version= OVE_ACTIVE_PROJECT_NAME=ncurses OVE_ACTIVE_PROJECT_COMMAND=build OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/ncurses/build
    2024-10-31 07:09:19.984478 (+00:00:00:003017):ncurses: build: done
    popd
    pushd /root/ove-workspace/ncurses
    2024-10-31 07:09:20.010211 (+00:00:00:025733):ncurses: install
    eval ncurses_version= OVE_ACTIVE_PROJECT_NAME=ncurses OVE_ACTIVE_PROJECT_COMMAND=install OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/ncurses/install
    2024-10-31 07:09:20.013514 (+00:00:00:003303):ncurses: install: done
    popd
    pushd /root/ove-workspace/xcm
    2024-10-31 07:09:20.139466 (+00:00:00:125952):xcm: configure
    eval xcm_version= OVE_ACTIVE_PROJECT_NAME=xcm OVE_ACTIVE_PROJECT_COMMAND=configure OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/xcm/configure
    2024-10-31 07:09:20.142550 (+00:00:00:003084):xcm: configure: done
    popd
    pushd /root/ove-workspace/xcm
    2024-10-31 07:09:20.163529 (+00:00:00:020979):xcm: build
    eval xcm_version= OVE_ACTIVE_PROJECT_NAME=xcm OVE_ACTIVE_PROJECT_COMMAND=build OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/xcm/build
    2024-10-31 07:09:20.168438 (+00:00:00:004909):xcm: build: done
    popd
    pushd /root/ove-workspace/xcm
    2024-10-31 07:09:20.193589 (+00:00:00:025151):xcm: install
    eval xcm_version= OVE_ACTIVE_PROJECT_NAME=xcm OVE_ACTIVE_PROJECT_COMMAND=install OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/xcm/install
    2024-10-31 07:09:20.196736 (+00:00:00:003147):xcm: install: done
    popd
    pushd /root/ove-workspace/tmux
    2024-10-31 07:09:20.221370 (+00:00:00:024634):tmux: configure
    eval tmux_version= OVE_ACTIVE_PROJECT_NAME=tmux OVE_ACTIVE_PROJECT_COMMAND=configure OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/tmux/configure
    2024-10-31 07:09:20.224484 (+00:00:00:003114):tmux: configure: done
    popd
    pushd /root/ove-workspace/tmux
    2024-10-31 07:09:20.248755 (+00:00:00:024271):tmux: build
    eval tmux_version= OVE_ACTIVE_PROJECT_NAME=tmux OVE_ACTIVE_PROJECT_COMMAND=build OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/tmux/build
    2024-10-31 07:09:20.252889 (+00:00:00:004134):tmux: build: done
    popd
    pushd /root/ove-workspace/tmux
    2024-10-31 07:09:20.276105 (+00:00:00:023216):tmux: install
    eval tmux_version= OVE_ACTIVE_PROJECT_NAME=tmux OVE_ACTIVE_PROJECT_COMMAND=install OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/tmux/install
    2024-10-31 07:09:20.280563 (+00:00:00:004458):tmux: install: done
    popd
    pushd /root/ove-workspace/dmce
    2024-10-31 07:09:20.340891 (+00:00:00:060328):dmce: install
    eval dmce_version= OVE_ACTIVE_PROJECT_NAME=dmce OVE_ACTIVE_PROJECT_COMMAND=install OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/dmce/install
    2024-10-31 07:09:20.343812 (+00:00:00:002921):dmce: install: done
    popd
    pushd /root/ove-workspace/codechecker
    2024-10-31 07:09:20.386850 (+00:00:00:043038):codechecker: build
    eval codechecker_version= OVE_ACTIVE_PROJECT_NAME=codechecker OVE_ACTIVE_PROJECT_COMMAND=build OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/codechecker/build
    2024-10-31 07:09:20.390212 (+00:00:00:003362):codechecker: build: done
    popd
    pushd /root/ove-workspace/codechecker
    2024-10-31 07:09:20.415122 (+00:00:00:024910):codechecker: install
    eval codechecker_version= OVE_ACTIVE_PROJECT_NAME=codechecker OVE_ACTIVE_PROJECT_COMMAND=install OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/codechecker/install
    2024-10-31 07:09:20.419225 (+00:00:00:004103):codechecker: install: done
    popd
    pushd /root/ove-workspace/ag
    2024-10-31 07:09:20.440854 (+00:00:00:021629):ag: configure
    eval ag_version= OVE_ACTIVE_PROJECT_NAME=ag OVE_ACTIVE_PROJECT_COMMAND=configure OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/ag/configure
    2024-10-31 07:09:20.444762 (+00:00:00:003908):ag: configure: done
    popd
    pushd /root/ove-workspace/ag
    2024-10-31 07:09:20.467627 (+00:00:00:022865):ag: build
    eval ag_version= OVE_ACTIVE_PROJECT_NAME=ag OVE_ACTIVE_PROJECT_COMMAND=build OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/ag/build
    2024-10-31 07:09:20.471522 (+00:00:00:003895):ag: build: done
    popd
    pushd /root/ove-workspace/ag
    2024-10-31 07:09:20.495957 (+00:00:00:024435):ag: install
    eval ag_version= OVE_ACTIVE_PROJECT_NAME=ag OVE_ACTIVE_PROJECT_COMMAND=install OVE_ACTIVE_PROJECT_VERSION= /root/ove-workspace/ove-tutorial/projects/ag/install
    2024-10-31 07:09:20.499249 (+00:00:00:003292):ag: install: done
    popd

When making this tutorial we used a [Incus container](https://linuxcontainers.org/incus), hence the "/root/" path.

As seen above the '**buildme**' command will check for any missing packages (specified by the 'needs:' in the 'projs' file), then iterate through any present bootstrap/configure/build/install steps for each project (as defined in 'ove-tutorial/projects/') in the correct build order. Please note, "missing packages" in this context refers to packages needed to build ove-tutorial and have nothing to do with packages needed for OVE itself.

If you are on Ubuntu/Debian, you could try to build everything without 'dry-run':

    $ ove dry-run 0
    $ ove buildme tmux
    # go grab a cup of tea
    $ which tmux
    /root/ove-workspace/stage/usr/bin/tmux
    $ tmux -V
    tmux 3.5a

Wait! What's up with the 'stage/usr/bin...' stuff? A short note: OVE uses a staging area for both intermediate and final build steps. You can look at it as a per-workspace mirror of how the included projects would install on your host if built without OVE. It is up to each project to decide what to do with the end results. Typically an OVE plugin (see Part IV, Plugins) would package relevant output (found in stage/...) into, well.. packages.

If you have Incus installed, you could try to build tmux within an incus container:

    $ ove create-instance tmux ubuntu/24.04/amd64
    # go grab a cup of tea
    ...
    tmux 3.5a

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
    codechecker   v6.24.4                ## HEAD (no branch)
    dmce          master                 ## master...origin/master
    libevent      release-2.1.10-stable  ## HEAD (no branch)
    ncurses       v6.1                   ## HEAD (no branch)
    ove-tutorial  f038c46                ## master...origin/master
    tmux          3.5a                   ## HEAD (no branch)
    xcm           master                 ## master...origin/master

    # update some files
    $ sed -i '1i Hi all' */{README,README.md,docs/README.md}

    # what files got updated?
    $ ove status
    .ove          master                 ## master...origin/master
    ag            master                 ## master...origin/master M README.md
    codechecker   v6.24.4                ## HEAD (no branch) M docs/README.md
    dmce          master                 ## master...origin/master M README.md
    libevent      release-2.1.10-stable  ## HEAD (no branch) M README.md
    ncurses       v6.1                   ## HEAD (no branch) M README
    ove-tutorial  d5727dc                ## master...origin/master M README.md
    tmux          3.5a                   ## HEAD (no branch) M README
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
