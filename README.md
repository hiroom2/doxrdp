# doxrdp

The doxrdp (Docker for XRDP) provides Docker image which runs XRDP on
Desktop Environment. Because doxrdp script is POSIX shell, if you use
Windows as docker host, you need to use Dockerfile in docker directory
like docker/ubuntu/1604/Dockerfile.

## Usage

    $ doxrdp list            # Show distribution.
    $ doxrdp build           # Build all distribution.
    $ doxrdp build <dist>    # Build distribution.
    $ doxrdp remove          # Remove all distribution.
    $ doxrdp remove <dist>   # Remove distribution.
    $ doxrdp clean           # Clean cache images.
    $ doxrdp rdesktop <dist> # Run rdesktop to distribution.

## Example

    $ doxrdp build ubuntu-1604_xubuntu
    $ doxrdp rdesktop ubuntu-1604_xubuntu &
    $ doxrdp rdesktop ubuntu-1604_xubuntu

![doxrdp](img/doxrdp.png)

## Support distribution

* [Ubuntu 16.04](docker/ubuntu/1604/README.md)
