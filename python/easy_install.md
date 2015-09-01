# Easy Install

## Downloading and Installing one Package
```bash
easy_install SQLObject
easy_install -f http://pythonpaste.org/package_index.html SQLObject
easy_install http://example.com/path/to/MyPackage-1.2.3.tgz
easy_install /my_downloads/OtherPackage-3.2.1-py2.3.egg
easy_install .    # install a source distribution
easy_install --user SQLAlchemy # install it in my home directory
```

## Uninstalling Packages
```bash
easy_install -m PackageName
```

After running the command, you can safely delete the .egg files or directories, along with any scripts.

## Exclude Scripts while Installing
use -x or --exclude-scripts

## Configuration files
it will look for these files:
1. setup.cfg in the current directory
1. ~/.pydistutils.cfg
1. $HOME\\pydistutils.cfg
1. distutils.cfg in the distutils package directory

### Example:
```
[easy_install]

# set the default location to install packages
install_dir = /home/me/lib/python

# Notice that indentation can be used to continue an option
# value; this is especially useful for the "--find-links"
# option, which tells easy_install to use download links on
# these pages before consulting PyPI:
#
find_links = http://sqlobject.org/
             http://peak.telecommunity.com/dist/
```
