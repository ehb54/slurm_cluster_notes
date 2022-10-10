## slurm cluster notes
Notes about installing a basic slurm cluster
This is a work-in-progress

### background

Needed to install a simple slurm cluster, added notes here for reference.

Installation of slurm is not described

### docs

[slurm.conf reference](https://slurm.schedmd.com/slurm.conf.html)

[useful notes on setting up a slurm cluster (raspberry Pi)](https://glmdev.medium.com/building-a-raspberry-pi-cluster-784f0df9afbd)

[command summary pdf](https://slurm.schedmd.com/pdfs/summary.pdf)

### this repo

Under `slurm.confs` are example `slurm.conf` files from clusters

### services
 * control/head node
   * munge
   * slurmctld
   * slurmd (if the control node is also a worker node)
 * worker nodes
   * munge
   * slurmd
 * additional services are required for slurm accounting if needed
 
### key requirements to get a 2nd node operational

* /etc/slurm/slurm.conf must match on all nodes
* node names should match and end with a number
  * e.g.
    * node0
    * node1
* uid and gid must match for at least user `slurm` & any user that will run jobs on nodes
  * [Good notes on changing uids & gids](https://www.thegeekdiary.com/how-to-correctly-change-the-uid-and-gid-of-a-user-group-in-linux/)
* munge.key must match
  * stored in `/etc/munge/munge.key`
  * make sure ownership/permissions are correct (munge:munge r-----)
  * after change of munge.key on nodeX `nodeX $ sudo service munge restart`
  * test with `node0 $ ssh nodeX munge -n | unmunge`
* time must be syncchronized on all nodes
* ports must be open between nodes
  * if firewall can be disabled on the interconnect, all the better.
* file systems must be shared on all nodes for any directory used for slurm jobs
  * e.g. for ultrascan us3iab jobs, we must share `/home/us3/lims/work`
  * NFS is typical, [e.g. NFS on Rocky 8](https://www.howtoforge.com/how-to-set-up-an-nfs-mount-on-rocky-linux-8/)
* any dependencies (programs/libraries) should match across nodes

### slurm.conf notes
 * simple changes to slurm.conf can be set by `$ scontrol reconfigure`
   * if it doesn't work, you can restart services on all nodes including control
 * give nodes weight to allocate jobs by weight (lower weight gets jobs first)
   * e.g. a head node also runs jobs & other non-slurm services
 * [specify port ranges](https://slurm.schedmd.com/slurm.conf.html#OPT_SrunPortRange)
   * e.g. `SrunPortRange=60001-63000`
 * log files are by default in `/var/log/slurm`
   * logging level for [SlurmdSyslogDebug & SlurmctldSyslogDebug](https://slurm.schedmd.com/slurm.conf.html#OPT_SlurmctldSyslogDebug)
     * typically I keep these at `info` for normal usage and `debug5` when setting up a new cluster

### testing notes/cmds
 * to test a shell on nodeX
   * `PS1='slurm $ ' srun -n 4 -w nodeX --pty bash`
     * `env | grep SLURM_STEP` will show some useful info in the job
     
