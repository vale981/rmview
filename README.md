# rMview: a fast live viewer for reMarkable 1 and 2

[![Demo](https://raw.githubusercontent.com/bordaigorl/rmview/vnc/screenshot.png)][demo]


## Features

* Demo [:rocket: here][demo]
* Fast streaming of the screen of your reMarkable to a window in your computer
* Support for reMarkable 1 and 2
* UI for zooming, panning, rotating
* Pen tracking: a pointer follows the position of the pen when hovering on the reMarkable
* Clone a frame into separate window for reference
* Save screenshots as PNG

## Known issues and caveats:

### :warning: For reMarkable 2 users :warning:
rMview should work out of the box with the stock firmware version 2.5 or below.

For 2.6 and above, for the moment you need to copy an extra library to the tablet. The process has not been automated yet. Please see [the wiki](https://github.com/bordaigorl/rmview/wiki/Installing-missing-libcrypto-on-RM2-v2.6-or-above) for instructions.

If you use [`rm2fb`](https://github.com/ddvk/remarkable2-framebuffer) there are known compatibilities issues that are [being addressed](https://github.com/pl-semiotics/rM-vnc-server/issues/5).


## Installation

The most efficient installation method is the semi-automatic one below, which requires a Python3 installation.
If you are looking for a standalone executable, check the [releases page](https://github.com/bordaigorl/rmview/releases) for executable bundles.
If there is no bundle for your operating system then follow the installation instructions below.

As a basic prerequisite you will need [Python3][py3] on your computer.

> :warning: Please make sure `pip` is pointing to the Python3 version if your system has Python2 as well.
If not, use `pip3` instead of `pip` in what follows.

> :warning: **WARNING** :warning::
> If you use [Anaconda][anaconda], please install the dependencies via `conda` (and not `pip`) then run `pip install .`.

### Semi-automatic installation

The easiest installation method is by using `pip`, from the root folder of this repository:

       pip install .

(please note the command ends with a dot)
which will install all required dependencies and install a new `rmview` command.

Then, from anywhere, you can execute `rmview` from the command line.
The tool will ask for the connection parameters and then ask permission to install the VNC server on the tablet.
Press <kbd>Auto install</kbd> to proceed.

If you plan to modify the source code, use `pip install -e .` so that when executing `rmview` you will be running your custom version.

### Manual installation

Install the dependencies ([PyQt5][pyqt5], [Paramiko][paramiko], [Twisted][twisted]) with `pip` or `conda` manually:

    # install dependencies
    pip install pyqt5 paramiko twisted
    # build resources file
    pyrcc5 -o src/rmview/resources.py resources.qrc

On the reMarkable itself you need to install [rM-vnc-server][vnc] by copying the relevant binary from the `bin` folder:

    # For reMarkable 1
    scp bin/rM1-vnc-server-standalone root@REMARKABLE_ADDRESS:rM-vnc-server-standalone

    # For reMarkable 2
    scp bin/rM2-vnc-server-standalone root@REMARKABLE_ADDRESS:rM-vnc-server-standalone

Then you can run the program with `python -m rmview`.

## Usage and configuration

**Suggested first use:**
after installing run `rmview`, insert when prompted the IP of your tablet and the password,
as found in the <kbd>Menu / Settings / Help / Copyright and Licences</kbd> menu of the tablet.
Then, optionally, select "Settings..." from the context menu (or the error dialog) to open
the default configuration file which you can edit according to the documentation below.

More generally, you can invoke the program with

    rmview [config]

the optional `config` parameter is the filename of a json configuration file.
If the parameter is not found, the program will look for a `rmview.json` file in the current directory, or, if not found, for the path stored in the environment variable `RMVIEW_CONF`.
If none are found, or if the configuration is underspecified, the tool is going to prompt for address/password.

### Configuration files

The supported configuration settings are below.
Look in file `example.json` for an example configuration.
All the settings are optional.

| Setting key              | Values                                                  | Default       |
| ------------------------ | ------------------------------------------------------- | ------------- |
| `ssh`                    | Connection parameters (see below)                       | `{}`          |
| `orientation`            | `"landscape"`, `"portrait"`, `"auto"`                   | `"landscape"` |
| `pen_size`               | diameter of pointer in px                               | `15`          |
| `pen_color`              | color of pointer and trail                              | `"red"`       |
| `pen_trail`              | persistence of trail in ms                              | `200`         |
| `background_color`       | color of window                                         | `"white"`     |
| `hide_pen_on_press`      | if true, the pointer is hidden while writing            | `true`        |
| `show_pen_on_lift`       | if true, the pointer is shown when lifting the pen      | `true`        |


Connection parameters are provided as a dictionary with the following keys (all optional):

| Parameter         | Values                                                  | Comments                              |
| ----------------- | ------------------------------------------------------- | ------------------------------------- |
| `address`         | IP of remarkable                                        | tool prompts for it if missing        |
| `auth_method`     | Either `"password"` or `"key"`                          | defaults to password if key not given |
| `username`        | Username for ssh access on reMarkable                   | default: `"root"`                     |
| `password`        | Password provided by reMarkable                         | not needed if key provided            |
| `key`             | Local path to key for ssh                               | not needed if password provided       |
| `timeout`         | Connection timeout in seconds                           | default: 1                            |
| `host_key_policy` | `"ask"`, `"ignore_new"`, `"ignore_all"`, `"auto_add"`   | default: `"ask"` (description below)  |

The `address` parameter can be either:
- a single string, in which case the address is used for connection
- a list of strings, which will be presented at launch for selection

To establish a connection with the tablet, you can use any of the following:
- Leave `auth_method`, `password` and `key` unspecified: this will ask for a password
- Specify `"auth_method": "key"` to use a SSH key. In case an SSH key hasn't already been associated with the tablet, you can provide its path with the `key` setting.
- Provide a `password` in settings

If `auth_method` is `password` but no password is specified, then the tool will ask for the password on connection.

As a security measure, the keys used by known hosts are checked at each connection to prevent man-in-the-middle attacks.
The first time you connect to the tablet, it will not be among the known hosts.
In this situation rMview will present the option to add it to the known hosts, which should be done in a trusted network.
Updates to the tablet's firmware modify the key used by it, so the next connection would see the mismatch between the old key and the new.
Again rMview would prompt the user in this case with the option to update the key. This should be done in a trusted network.
The `host_key_policy` parameter controls this behaviour:
- `"ask"` is the default behaviour and prompts the user with a choice when the host key is unknown or not matching.
- `"ignore_new"` ignores unknown keys but reports mismatches.
- `"ignore_all"` ignores both unknown and not matching keys. Use at your own risk.
- `"auto_add"`  adds unknown keys without prompting but reports mismatches.

The old `"insecure_auto_add_host": true` parameter is deprecated and equivalent to `"ignore_all"`.

In case your `~/.ssh/known_hosts` file contains the relevant key associations, rMview should pick them up.
If you use the "Add/Update" feature when prompted by rMview (for example after a tablet update) then `~/.ssh/known_hosts` will be ignored from then on.


:warning: **Key format error:**
If you get an error when connect using a key, but the key seems ok when connecting manually with ssh, you probably need to convert the key to the PEM format (or re-generate it using the `-m PEM` option of `ssh-keygen`). See [here](https://github.com/paramiko/paramiko/issues/340#issuecomment-492448662) for details.


## To Do

 - [ ] Settings dialog
 - [ ] About dialog
 - [ ] Pause stream of screen/pen
 - [ ] Binary bundles for Window, Linux and MacOs (PyInstaller?)
 - [ ] Add interaction for Lamy button? (1 331 1 down, 1 331 0 up)
 - [ ] Remove dependency to Twisted in `vnc` branch


## Legacy reStreamer-like version

There are two versions of rMview, presenting the same interface but using different back-ends (thus requiring different setups on the reMarkable):

* The "VNC-based" version, in the [`vnc` branch][vnc-branch] (default)
* The "reStreamer-like" version, in the [`ssh` branch][ssh-branch]

In my tests, the VNC version is a clear winner.
The `ssh` branch of this repo hosts the reStreamer-like version for those who prefer it, but it should be considered unmaintained.


## Credits

The VNC server running on the tablet is developed by @pl-semiotics:

- [rM-vnc-server][vnc]

I took inspiration from the following projects:

- [QtImageViewer](https://github.com/marcel-goldschen-ohm/PyQtImageViewer/)
- [remarkable_mouse](https://github.com/Evidlo/remarkable_mouse/)
- [reStream](https://github.com/rien/reStream)
- [VNC client](https://github.com/sibson/vncdotool) originally written by Chris Liechti

Icons adapted from designs by Freepik, xnimrodx from www.flaticon.com

Thanks to @adem amd @ChrisPattison for their [PRs](https://github.com/bordaigorl/rmview/issues?q=is%3Apr+is%3Aclosed).

## Disclaimer

This project is not affiliated to, nor endorsed by, [reMarkable AS](https://remarkable.com/).
**I assume no responsibility for any damage done to your device due to the use of this software.**

## Licence

GPLv3

[vnc]: https://github.com/pl-semiotics/rM-vnc-server
[demo]: https://www.reddit.com/r/RemarkableTablet/comments/gtjrqt/rmview_now_with_support_for_vnc/
[ssh-branch]: https://github.com/bordaigorl/rmview/tree/ssh
[vnc-branch]: https://github.com/bordaigorl/rmview/tree/vnc

[py3]: https://www.python.org/downloads/
[anaconda]: https://docs.anaconda.com/anaconda
[pyqt5]: https://www.riverbankcomputing.com/software/pyqt/
[paramiko]: http://www.paramiko.org/
[twisted]: https://twistedmatrix.com/trac/
