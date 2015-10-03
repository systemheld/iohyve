# iohyve v0.6
"Multiple Disk Disorder Edition"

FreeBSD bhyve manager utilizing ZFS and other FreeBSD tools. 
*Over the hill to 1.0*

iohyve creates, stores, manages, and launches bhyve guests utilizing built in FreeBSD features. 
The idea is based on iocage, a jail manager utilizing some of the same principles. 
There is currently support for VIMAGE/VNET. Documentation soon to come. For more information on VNET support, run:

    iohyve readme

Or create a readme file:

    iohyve readme > awesomesauce.txt 

DO YOU EVEN MAN PAGE?

    iohyve man 

## ZOMG EVERYTHING BROKE AFTER UPDATING?!
You probably are updating from something before v0.5.6 and disk images 
are not loading. NEVER FEAR. I just changed how iohyve names disk 
images. Lame, I know. For now, clone the latest git, run `./fixme.sh` in 
the directory. This should rename all the zvols to the correct scheme. 

The script will shutdown all the guests (scram).

If you run into further issues, please feel free to open an issue. 

**Pre-Flight Checklist**

[Taken from the FreeBSD handbook https://www.freebsd.org/doc/en/books/handbook/virtualization-host-bhyve.html]


The first step to creating a virtual machine in bhyve is configuring the host system. First, load the bhyve kernel module:

    kldload vmm
    kldload nmdm

Then, create a tap interface for the network device in the virtual machine to attach to. In order for the network device to participate in the network, also create a bridge interface containing the tap 
interface ane the physical interface as members. In this example, the physical interface is igb0:

    ifconfig tap0 create
    sysctl net.link.tap.up_on_open=1
        net.link.tap.up_on_open: 0 -> 1
    ifconfig bridge0 create
    ifconfig bridge0 addm igb0 addm tap0
    ifconfig bridge0 up

Once you have created tap0, there is no need to create more tap interfaces for iohyve, it will create them and add them 
to bridge0 automatically as of v0.3.2 master branch.

The built-in readme 'iohyve readme' has more information on VNET setups. 

**Usage**

```
iohyve  version
        setup [pool]
        list
        isolist
        vmmlist
        running
        fetch [URL]
        renameiso [ISO] [newname]
        remove [ISO]
        create [name] [size]
        install [name] [ISO]
        load [name] [path/to/bootdisk]
        boot [name] [runmode] [pcidevices]
        start [name] [-s | -a]
        stop [name]
        scram
        destroy [name]
        rename [name] [newname]
        delete [name]
        set [name] [prop=value]
        get [name] [prop]
        getall [name]
        taplist
        activetaps
        conlist
        activecons
        console [name]
        conreset
        readme
        help
        man 
```

**Setup**

Setup iohyve by telling it what zpool to use

    iohyve setup tank

**General Usage**

List all guests created with:

    iohyve list

List all guests that have resources allocated using:

    iohyve vmmlist

List all runnng guests using:

    iohvye running

You can change guest properties by using set:

    iohyve set bsdguest ram=512M                 #set ram to 512 Megabytes
    iohyve set bsdguest cpu=1                    #set cpus to 1 core
    iohyve set bsdguest tap=tap0                 #set tap device for ethernet
    iohyve set bsdguest con=nmdm0                #set the console to attach to
    iohyve set bsdguest pcidev:1=passthru,2/0/0  #pass through a pci device

Get a specific guest property:

    iohyve get bsdguest ram

Get all guest properties:

    iohyve getall bsdguest


**FreeBSD Guests**

Fetch FreeBSD install ISO for later:

    iohyve fetch ftp://ftp.freebsd.org/.../10.1/FreeBSD-10.1-RELEASE-amd64-bootonly.iso

Rename the ISO if you would like:

    iohyve renameiso FreeBSD-10.1-RELEASE-amd64-bootonly.iso fbsd10.iso

Create a new FreeBSD guest named bsdguest with an 8Gigabyte virtual HDD:

    iohyve create bsdguest 8G

List ISO's:

    iohyve isolist

Install the FreeBSD guest bsdguest:

    iohyve install bsdguest FreeBSD-10.1-RELEASE-amd64-bootonly.iso

Console into the intallation:

    iohyve console bsdguest

Once installation is done, exit console (~~.) and stop guest:

    iohyve stop bsdguest

Now that the guest is installed, it can be started like usual:

    iohyve start bsdguest

Some guest os's can be gracefully stopped:

    iohyve stop bsdguest


**Other BSDs:**

Try out OpenBSD:
````
iohyve set obsdguest loader=grub-bhyve
iohyve set obsdguest os=openbsd
iohyve install obsdguest install57.iso
iohyve console obsdguest
````
Try out NetBSD:
````
iohyve set nbsdguest loader=grub-bhyve
iohyve set nbsdguest os=netbsd
iohyve install nbsdguest NetBSD-6.1.5-amd64.iso
iohyve console nbsdguest
````
**Linux flavors:**

Try out Debian or Ubuntu:
````
iohyve set debguest loader=grub-bhyve
iohyve set debguest os=debian
iohyve install debguest debian-8.2.0-amd64-i386-netinst.iso
iohyve console debguest
````
