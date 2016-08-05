## Ultimate Deployment Appliance Templates
The problem with only a few of something...you convince yourself that automation is too much trouble. My lab has about 10 hosts. After installing ESXi half a dozen times, I was convinced automation not too much trouble. Happily, I read an article by [Mike Laverick](http://www.mikelaverick.com). The [article](http://www.mikelaverick.com/2015/08/the-ultimate-deployment-appliance-adds-vmware-esxi-6-support/) was singing the praises of Carl Thijssen's [Ultimate Deployment Appliance](http://ultimatedeployment.org). Mike has written an [admin guide](http://www.mikelaverick.com/downloads/uda20adminguide.pdf) for UDA. Once you decide to go down the path of using [kickstart scripts](http://pubs.vmware.com/vsphere-60/index.jsp?topic=%2Fcom.vmware.vsphere.install.doc%2FGUID-C5E255A0-CFF1-437A-BB8A-30FE0736B73F.html) delivered via PXE booting, you need to learn the in's and out's of configuring VMware's ESXi via the command line. I was able to learn a lot from William Lam's articles at [virtuallyGhetto](http://www.virtuallyghetto.com).

## Learnings from trial and error
The install/upgrade phase does not run in a shell. If you need to conditionally change commands, use `%pre` to write your commands to a temp file, and `%include` that temp file. I boot my vSAN hosts from a USB thumb drive plugged into the motherboard, and I do not want a VMFS datastore on that device. I needed different `install` commands, and `%pre` saved me from creating different UDA templates.

When I am installing (verses upgrading), I want all the storage cleared. I probably have not learned the "easy button", so I brute forced the issue by adding the HDD and SSD device names to my UDA subtemplate. This allows the script to have a host specific `clearpart` command. There are probably some commands I could pipe into `awk` to produce a comma separated list of devices, and this would be another good use of the `%pre` section.

The majority of the work is done during the `%firstboot` following the `install`. We have a shell (or python) available.

One of my hosts has a pair of NICs on an Intel card in addition to a NIC on the motherboard. I would have turned the motherboard NIC off in the Bios; however, this is the only NIC available at PXE boot time. I needed to deal with a different number of NICs, which turns out to be straight forward, once you make use of environment variables in your shell. A little shell logic to define your conventions for different numbers of NICs.

Networking breaks down into two sections: vmkernel adapters for special portgroups and application portgroups.

For hosts booting off a USB thumb drive, a datastore is needed for ESXi's scratch files. I decided on a convention for selecting an NFS mount point and the directory for a host's scratch.

I have become a fan of the [ESXi Embedded Host Client](https://labs.vmware.com/flings/esxi-embedded-host-client). Activity on the client is strong, so I always look to see if my script is using the latest esxui release.

## Takeaways
Kickstart scripts are more powerful than I first thought. The `%` sections provide you shell or python. A lot can be done!

Once you get UDA setup, reconfiguring your lab will not be a big deal, so you will try more configurations. As Justin Smith says in his ["The Three R’s of Enterprise Security: Rotate, Repave, and Repair"](https://medium.com/built-to-adapt/the-three-r-s-of-enterprise-security-rotate-repave-and-repair-f64f6d6ba29d#.7e4nbzyma), repaving is good.