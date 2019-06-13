HTSST UK Team Boards Status
=========================================

* author: xuwei
* date: 13 June 2019

Background
-------------

Currently some boards are deployed at 302 office and some are deployed at 101 office.

Even in the 101 office, some boards are deployed inside our lab and some are deployed
in the 101 server room.

From the network view, all the boards are deployed in the blue zone.

And the blue zone network between 302 and 101 are connected but they belong to different sub network.

The blue zone network in 302 begins 172.18.

The blue zone network in 101 begins 172.19.

Board Booting
----------------------

All the boards are configured to boot via PXE which provide dhcp service and tftp service.

Generally the board booting sequence is like below. ::

        PXE ----> grub ----> tftp ----> NFS server(optional)

The whole PXE environment can be set up according the document from the Estuary project:
`Setup_PXE_Env_on_Host <https://github.com/open-estuary/estuary/blob/master/doc/Setup_PXE_Env_on_Host.4All.md>`_


The only difference is that we use dnsmasq other than isc-dhcp-server for the dhcp service.

There are 2 PXE servers deployed. Their detail information is as below table.


.. list-table:: PXE configurations
        :widths: 5 15 5 30 30
        :header-rows: 1

        * - office
          - physical address
          - ip address
          - dnsmasq configuraion
          - tftp configuration
        * - 302
          - bluezone lab stack huawei x86 server
          - 172.18.45.23
          - `/etc/dnsmasq.conf` ::

                log-dhcp
                dhcp-range=172.18.45.100,172.18.45.200
                dhcp-boot=grubaa64.efi,,172.18.45.23
                dhcp-option=3,172.18.45.1
                dhcp-option=6,172.18.45.1

          - `/etc/default/tftpd-hpa` ::

                TFTP_USERNAME="tftp"
                TFTP_DIRECTORY="/home/hisilicon/ftp"
                TFTP_ADDRESS="[::]:69"
                TFTP_OPTIONS="-s -c -l"



        * - 101
          - bluezone lab x86 desktop A141109007
          - 172.19.45.36
          - `/etc/dnsmasq.conf` ::

                log-dhcp
                dhcp-range=172.19.45.50,172.19.45.255
                dhcp-boot=grubaa64.efi,,172.19.45.36
                dhcp-option=3,172.19.45.1
                dhcp-option=6,172.19.45.1

          - `/etc/default/tftpd-hpa` ::

                TFTP_USERNAME="tftp"
                TFTP_DIRECTORY="/home/hisilicon/ftp"
                TFTP_ADDRESS="[::]:69"
                TFTP_OPTIONS="-s -c -l"

.. WARNING:: WARNING

        To avoid duplication, 172.18.45.23 server's tftp root folder is remotly mounted by 172.19.45.36.

        Everytime after rebooted the 172.19.45.36, below command must be run manually to mount the tftp share foler on 172.19.45.36. ::

                sudo sshfs -o reconnect,ServerAliveInterval=15,nonempty,allow_other,IdentityFile=/home/joyx/.ssh/id_rsa joyx@172.18.45.23:/home/hisilicon/ftp  /var/lib/lava/dispatcher/tmp

Access Agent
----------------

Every board can still be accessed via ipmi commands like::

        ipmitool -H 172.19.45.48 -I lanplus -U Administrator -P Admin@9000 power reset
        ipmitool -H 172.19.45.48 -I lanplus -U Administrator -P Admin@9000 sol activate

To share the boards and avoid confiliciton among the team, boardlab script is imported to the
UK lab as well.

All the boards are managed via the `boardlab script <https://github.com/open-estuary/boardlab.git>`_ .

Currently 172.18.45.23 is our only board server on which the boardlab script is deployed.

boards topology
~~~~~~~~~~~~~~~~

All the board related infomation is configured via `/usr/local/openlab/openlab_conf/`.

.. list-table:: board configurations
        :widths: 5 5 10 5 5 5 5
        :header-rows: 1

        * - board number
          - board type
          - physical address
          - status
          - bmc ip address
          - bmc username
          - bmc username password
        * - board1
          - D03
          - HTSST stack @bluezone lab, 302
          - working
          - 172.18.45.40
          - root
          - Huawei12#$
        * - board2
          - none
          - HTSST stack @bluezone lab, 302
          - broken
          - 172.18.45.41
          - root
          - Huawei12#$
        * - board3
          - D05
          - HTSST stack @bluezone lab, 302
          - working
          - 172.18.45.42
          - root
          - Huawei12#$
        * - board4
          - none
          - HTSST stack @bluezone lab, 302
          - broken
          - 172.18.45.40
          - root
          - Huawei12#$
        * - board5
          - none
          - HTSST stack @bluezone lab, 302
          - broken
          - 172.18.45.43
          - root
          - Huawei12#$
        * - board6
          - D03
          - HTSST stack @bluezone lab, 101
          - working
          - 172.19.45.44
          - root
          - Huawei12#$
        * - board7
          - D05
          - HTSST stack @bluezone lab, 302
          - working
          - 172.18.45.45
          - root
          - Huawei12#$
        * - board8
          - D05
          - HTSST stack @server room, 101
          - working
          - 172.19.45.46
          - root
          - Huawei12#$
        * - board9
          - D02
          - HTSST stack @bluezone lab, 101
          - working
          - none
          - none
          - none
        * - board10
          - D06
          - HTSST stack @bluezone lab, 101
          - working
          - 172.19.45.47
          - Administrator
          - Admin@9000
        * - board11
          - D06
          - HTSST stack @bluezone lab, 101
          - working
          - 172.19.45.49
          - Administrator
          - Admin@9000
        * - board12
          - D06
          - HTSST stack @bluezone lab, 101
          - working
          - 172.19.45.48
          - Administrator
          - Admin@9000

what to do when adding new board
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. find a proper ip address for the bmc
#. configure the board bmc ip address via directly connnecting the board with the laptop
#. add bmc information in `/usr/local/openlab/openlab_conf/bmcinfo.cfg`
#. add board information in `/usr/local/openlab/openlab_conf/boardinfo.cfg` according the bmc
   and pxe mac address which can be find in the UEFI interface
#. update the user configuration to allow access the new board via `/usr/local/openlab/openlab_conf/userinfo.cfg`


what to do when new member joined
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. new user need to run the `inituser` command on the 172.19.45.23 to create the tftp and grub configuration related files.
#. the Lab adminstrator need to updat ethe user configuration to grant the board access right via `/usr/local/openlab/openlab_conf/userinfo.cfg`

Tips
------------------------

* adjust the fan noise

  #. login into the board bmc shell according the bmc ip, user and password information 
     from `/usr/local/openlab/openlab_conf/bmcinfo.cfg` on the board server(172.18.45.23)
  #. run below command to set fan to run with 20% ::

          ipmcset -d fanmode -v 1 0
          ipmcset -d fanlevel -v 20
