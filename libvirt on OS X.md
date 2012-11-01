#libvirt on OS X

by can.  
2012/10/31

## install

1. install homebrew

2. `brew doctor` : [solve warnings](http://superuser.com/questions/435032/errors-in-homebrew-on-os-x-lion)

3. `brew install libvirt`

4. link(`ln -s`) `/usr/local/Cellar/libvirt/0.9.11.6/lib/python2.7/site-packages/` to `/Library/Python/2.7/site-packages/`

5. then `import libvirt` in python should be OK


## use

the URI:

- local: `[driver]:///system`

example: `qemu:///system`

- remote: `[driver+connection]://[address]/system`

example: `qemu+ssh://root@210.25.137.233/system?socket=/var/run/libvirt/libvirt-sock`, [ref](http://jedi.be/blog/2011/09/13/libvirt-fog-provider/)

## more to read

http://publib.boulder.ibm.com/infocenter/lnxinfo/v3r0m0/index.jsp?topic=%2Fliaat%2Fliaatkvmsecsrmsasl.htm

http://www.ibm.com/developerworks/linux/library/os-python-kvm-scripting1/index.html