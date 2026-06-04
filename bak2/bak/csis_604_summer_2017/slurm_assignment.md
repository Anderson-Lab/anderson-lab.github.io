---
permalink: /csis_604_summer_2017/slurm_assignment/
title: "Distributed Computing"
excerpt: "Anderson Data Science Research Lab."
---

{% include base_path %}

In this assignment, we will install and configure Slurm Manager. The master is obviously the master node, and the other two VMs are the compute nodes. There is a good tutorial available <a href="https://www.invik.xyz/work/Slurm-on-Ubuntu-Trusty/">https://www.invik.xyz/work/Slurm-on-Ubuntu-Trusty/</a>. I won't repeat everything it says. I'll make general comments below.

## Notes
My install did not have the configurator they mention in the tutorial, so I used the online one: <a href="https://slurm.schedmd.com/configurator.html">https://slurm.schedmd.com/configurator.html</a>. I kept most of the defaults and only tweaked for master, server1, etc. Honestly, it's probably a good idea to start with just everything on master, and then incorporate the other compute nodes.

I had to install this as well. sudo apt-get install mailutils

sinfo and other commands are good for showing success. You can also try a script that shows it is running on different hostnames, such as:
<pre>
#!/bin/bash
#SBATCH -p debug
#SBATCH -n 1
#SBATCH -t 12:00:00
#SBATCH -J some_job_name

hostname > /shared/`hostname`.out
</pre>

There are also a lot of other resources online such as <a href="https://thehatteronline.com/2014/11/18/turn-your-workstation-into-a-mini-grid-with-slurm/">https://thehatteronline.com/2014/11/18/turn-your-workstation-into-a-mini-grid-with-slurm/</a>
