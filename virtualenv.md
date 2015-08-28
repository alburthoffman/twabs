# Virtualenv Notes
## Installation
```bash
$ [sudo] pip install Virtualenv

$ curl -O https://pypi.python.org/packages/source/v/virtualenv/virtualenv-X.X.tar.gz
$ tar xvfz virtualenv-X.X.tar.gz
$ cd virtualenv-X.X
$ python virtualenv.py myVE
```

## usage
basic command:
```bash
$ virtualenv projectpath
```

the command will create these directories:
- projectpath/bin
- projectpath/lib
- projectpath/lib/pythonX.x/site-packages
- projectpath/include

also these will be done:
- pip and setuptools will be installed

make sure you can run script with:
```bash
#! /path/to/projectpath/bin/python
```

then you can activate the local development environment:
```bash
$ source bin/activate
```

if you want to use system environment, then disable it:
```bash
$ deactivate
```

### option --system-site-packages
If you build with virtualenv --system-site-packages ENV, your virtual environment will inherit packages from /usr/lib/python2.7/site-packages

### option --extra-search-dir
This option allows you to provide your own versions of setuptools and/or pip to use instead of the embedded versions that come with virtualenv.

run virtualenv like this:
```bash
$ virtualenv --extra-search-dir=/path/to/distributions ENV
```

## command options
### specify the python version
```
-p PYTHON_EXE, --python=PYTHON_EXE
```

### do not install setuptools and pip
```
--no-setuptools
--no-pip
```

## configuration
### environment variables
Each command line option is automatically used to look for environment variables with the name format VIRTUALENV_<UPPER_NAME>. That means the name of the command line options are capitalized and have dashes ('-') replaced with underscores ('_').

### configuration files
virtualenv also looks for a standard ini config file. On Unix and Mac OS X that’s $HOME/.virtualenv/virtualenv.ini and on Windows, it’s %APPDATA%\virtualenv\virtualenv.ini.

the content is like:
```
[virtualenv]
extra-search-dir =
    /path/to/dists
    /path/to/other/dists
```

## extend
```python
FASS_GIT_LOCATION = "git://github.com/socialplanning/fassembler.git"
import shutil
def extend_parser(parser):
    parser.add_option(
        '--git',
        dest='fassembler_git',
        default=FASS_GIT_LOCATION,
        help='Location of a git repository to use for the installation of fassembler')
def adjust_options(options, args):
    if not args:
        return # caller will raise error

    # We're actually going to build the venv in a subdirectory
    base_dir = args[0]
    args[0] = join(base_dir, 'fassembler')
def after_install(options, home_dir):
    base_dir = os.path.dirname(home_dir)
    src_dir = join(home_dir, 'src')
    fassembler_git = options.fassembler_git
    fassembler_dir = join(src_dir, 'fassembler')
    call_subprocess(['rm', '-rf', fassembler_dir])
    logger.notify('Installing fassembler from %s to %s' % (fassembler_git, fassembler_dir))
    fs_ensure_dir(src_dir)
    call_subprocess(['git', 'clone', fassembler_git, fassembler_dir],
                    show_stdout=True)
    logger.indent += 2
    try:
        call_subprocess([os.path.abspath(join(home_dir, 'bin', 'easy_install')), 'mysql-python'],
                        cwd=os.path.abspath(fassembler_dir),
                        filter_stdout=filter_python_develop,
                        show_stdout=False)
        call_subprocess([os.path.abspath(join(home_dir, 'bin', 'python')), 'setup.py', 'develop'],
                        cwd=os.path.abspath(fassembler_dir),
                        filter_stdout=filter_python_develop,
                        show_stdout=False)
    finally:
        logger.indent -= 2
    script_dir = join(base_dir, 'bin')
    script_dest = join(script_dir, 'fassembler')
    logger.notify('Copying fassembler to %s' % script_dest)
    fs_ensure_dir(script_dir)
    if os.path.exists(script_dest):
        os.unlink(script_dest)
    os.symlink('../fassembler/bin/fassembler', script_dest)
    etc_dir = join(base_dir, 'etc')
    build_ini = join(etc_dir, 'build.ini')
    if not os.path.exists(build_ini):
        fs_ensure_dir(etc_dir)
        logger.notify('Touching %s' % build_ini)
        f = open(build_ini, 'w')
        f.close()
    logger.notify('Run "%s fassembler:topp" (etc) to build out the environment'
                  % script_dest)
    logger.notify('Run "%s Package" to install new packages that provide builds'
                  % join(home_dir, 'bin', 'easy_install'))
def fs_ensure_dir(dir):
    if not os.path.exists(dir):
        logger.info('Creating directory %s' % dir)
        os.makedirs(dir)
def filter_python_develop(line):
    if not line.strip():
        return Logger.DEBUG
    for prefix in ['Searching for', 'Reading ', 'Best match: ', 'Processing ',
                   'Moving ', 'Adding ', 'running ', 'writing ', 'Creating ',
                   'creating ', 'Copying ']:
        if line.startswith(prefix):
            return Logger.DEBUG
    return Logger.NOTIFY
```
