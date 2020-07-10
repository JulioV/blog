---
date: "2020-05-28"
tags: ["data-science", "programming"]
title: "Setting up Travis CI to test R and Python scripts in MacOS and Ubuntu"
---

A colleague and I configured Travis CI to run the tests of a project that relies on R and Python scripts. This project supports both macOS and Linux, so it was essential to test it in both environments. After some trial and error, we got this working with the `travis.yaml` file below. Our deployment manages the following project requirements: MySQL, Python 3.7, miniconda with a virtual environment, R and a cached virtual environment with renv, and slack notifications.

For Linux (Ubuntu 16.04) we do:

- Install brew, linuxbrew-wrapper and linuxbrew/xorg
- Install R using brew
- Install miniconda using their provided script installer
- Restore our conda virtual env
- Cache [renv](https://rstudio.github.io/renv/index.html)'s library. We use renv to keep a reproducible R environment with 161 packages; however, renv had to build them from source in Ubuntu 16.04 and our travis build was timing out. The solution we implemented was building the renv library (and therefore the travis' cache) in three steps, first committing a small renv.lock with 40 packages, then 100, then all 161.

For MacOS (10.14.4)

- Set up the OS image with `osx_image: xcode11.3` and `language: generic`
- Install MySQL, R and miniconda using brew (brew and Python 3.7 are already installed)
- Restore our conda virtual env
- Cache renv's library. We faced and solved the same problem we had in Linux with renv's building times timing out our travis build (see above). In addition, in MacOS, renv's library path contains a space `~/Library/Application Support/renv` which was causing issues with travis' cache mechanism, as a quick fix we disabled renv's global cache `R -e 'renv::settings$use.cache(FALSE)'` for both Linux and MacOS (for consistency) and cached our project's renv library folder `$TRAVIS_BUILD_DIR/renv/library` as it now contains the actual packages instead of symbolic links.

*UPDATE* Recently ``r`` tap dependency on gcc changed from 9 to 10 and that broke our ``renv`` library cache. To force travis to use gcc 9 and r 4.0.0 use this updated file:

```yaml
services:
  - mysql

language: python            # this works for Linux but is an error on macOS or Windows

jobs:
  include:
    - name: "Python 3.7 on Xenial Linux"
      os: linux
      language: python
      python: 3.7
      before_install:
        - /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
        - export PATH=/home/linuxbrew/.linuxbrew/bin:$PATH
        - source ~/.bashrc
        - sudo apt-get install linuxbrew-wrapper
        - brew tap --shallow linuxbrew/xorg
        - brew install r
        - R --version
        - wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh -O miniconda.sh;
        - bash miniconda.sh -b -p $HOME/miniconda
        - source "$HOME/miniconda/etc/profile.d/conda.sh"
        - hash -r
        - conda config --set always_yes yes --set changeps1 no
      cache:
        directories:
          - /home/travis/.linuxbrew
          - $HOME/.local/share/renv
          - $TRAVIS_BUILD_DIR/renv/library
            
    - name: "Python 3.7 on macOS"
      os: osx
      osx_image: xcode11.3  # Python 3.7 running on macOS 10.14.4
      language: generic       # 'language: python' is an error on Travis CI macOS
      before_install:
        - HOMEBREW_NO_AUTO_UPDATE=1 brew install gcc@9 # HOMEBREW_NO_AUTO_UPDATE=1 is not set as an env variable because that invalidates renv's cache in travis
        - HOMEBREW_NO_AUTO_UPDATE=1 brew install https://github.com/Homebrew/homebrew-core/raw/218998d/Formula/r.rb # R 4.0.0
        - R --version
        - HOMEBREW_NO_AUTO_UPDATE=1 brew install mysql
        - HOMEBREW_NO_AUTO_UPDATE=1 brew services start mysql
        - HOMEBREW_NO_AUTO_UPDATE=1 brew cask install miniconda
        - eval "$(/opt/miniconda3/condabin/conda shell.bash hook)"
        - eval "$(conda shell.bash hook)" # properly initialise non-interactive shell with gcc 9 and no auto update brew

      env:
        - RENV_PATHS_ROOT="$HOME/renv/cache"
      cache:
        directories:
          - /usr/local/lib/R
          - $RENV_PATHS_ROOT
          - $TRAVIS_BUILD_DIR/renv/library

install:
  - conda init bash
  - conda update -q --all --yes conda
  - conda env create -q -n test-environment python=$TRAVIS_PYTHON_VERSION --file environment.yml
  - conda activate test-environment
  - snakemake renv_install
  - R -e 'renv::settings$use.cache(FALSE)'
  - snakemake renv_restore 
    
script:
  - bash tests/scripts/run_tests.sh
  
notifications:
  email: false
  slack:
    # if: branch = travis_test
    secure: cJIpmIjb3zA5AMDBo9axF1v6fYNIgMm6s6UdMNOlHiT511xHGsaLUFej3lACwQLig4Gr94ySI61YdrP+RX1lFcYxusH+kUU/c8LX0PmSKNeKnycM3w/pCM+yTp/6oQG6ZrJD7pNm6zhB0xPL61uSmYhcr+JJ1sh4iLiON+J8/C+IfnAHm1ORkxJ0IxASkiP/LvaiAQDw8lNyYIZNWjSDNZbx68o1VNakyk6Vik3x8omiE3w33rzI2/JAx//QTxOq2J0dtV1AqYYSOWS4iXblV09NLBqgGrhAhrQ6+TbPHSPIyL/4EdhvS+YXO+SBWS7ODD7j/MuL6XiA4SujW72od2rgXNmOjFnlQvIrULO5bzv39BKKDkldvz9+XCyXLcjoLIwA/rmUnwMndNoC7NoD/CkQEevUxswXXB9811BmIFx/7GOHouVxwB2gaMAzkCroZJVwgbrc6ESSOVE5SMcb3wPMbpd8cXOgVZXJcmk5wK206zxXPigCvFfknqOnwDqRgyIWSFoTd/2wHppA7ND3R5U42nQTbEQ7MiONsOo61GlJTTxJELz32sLKl388AuAgOY7+0sqPibxMaHJkF1V4nYVTH0/H5bO/edK4VHMloJ6s0kuyko7LT5EMQf3pBJij5TnYmD2E60t+bSBAxHuH7WA5dvL+igjGEwROnxDc9pc=
    on_success: always
    template:
      - "Repo `%{repository_slug}` *%{result}* build (<%{build_url}|#%{build_number}>) for commit (<%{compare_url}|%{commit}>) on branch `%{branch}`."
      - "Execution time: *%{duration}*"
      - "Message: %{message}"
```

```yaml
# This is the old file for reference only
services:
  - mysql
language: python

jobs:
  include:
    - name: "Python 3.7 on Xenial Linux"
      os: linux
      language: python
      python: 3.7
      before_install:
        - /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
        - export PATH=/home/linuxbrew/.linuxbrew/bin:$PATH
        - source ~/.bashrc
        - sudo apt-get install linuxbrew-wrapper
        - brew tap --shallow linuxbrew/xorg
        - brew install r
        - R --version
        - wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh -O miniconda.sh;
        - bash miniconda.sh -b -p $HOME/miniconda
        - source "$HOME/miniconda/etc/profile.d/conda.sh"
        - hash -r
        - conda config --set always_yes yes --set changeps1 no
      cache:
        directories:
          - /home/travis/.linuxbrew
          - $HOME/.local/share/renv # global renv cache in linux (not used)
          - $TRAVIS_BUILD_DIR/renv/library # local renv cache
            
    - name: "Python 3.7 on macOS"
      os: osx
      osx_image: xcode11.3  # Python 3.7 running on macOS 10.14.4
      language: generic
      before_install:
        - brew install mysql
        - brew services start mysql
        - brew install r
        - R --version
        - brew cask install miniconda
        - eval "$(/usr/local/bin/conda shell.bash hook)"
      env:
        - RENV_PATHS_ROOT="$HOME/renv/cache"
      cache:
        directories:
          - /usr/local/lib/R
          - $RENV_PATHS_ROOT # global renv cache in MacOS (not used)
          - $TRAVIS_BUILD_DIR/renv/library # local renv cache

install:
  - conda init
  - conda update -q --all --yes conda
  - conda env create -q -n test-environment python=$TRAVIS_PYTHON_VERSION --file environment.yml
  - conda activate test-environment
  - snakemake renv_install
  - R -e 'renv::settings$use.cache(FALSE)'
  - snakemake renv_restore 
    
script:
  - python -m unittest discover tests/scripts/ -v

notifications:
  email: false
  slack:
    secure: SLACK_SECURE_KEY
    on_success: always
    template:
      - "Repo `%{repository_slug}` *%{result}* build (<%{build_url}|#%{build_number}>) for commit (<%{compare_url}|%{commit}>) on branch `%{branch}`."
      - "Execution time: *%{duration}*"
      - "Message: %{message}"
```