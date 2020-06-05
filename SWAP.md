# Linux Swapfile

Reference: https://linuxize.com/post/create-a-linux-swap-file/

Swap is a space on a disk that is used when the amount of physical RAM memory is full. When a Linux system runs out of RAM, inactive pages are moved from the RAM to the swap space.

## Create the swap file

- Create a file that will be used for swap:

  `$ sudo fallocate -l [size in bytes or e.g 2G] /swapfile`

- Set to only root user should be able to read and write the swap file.

  `$ sudo chmod 600 /swapfile`

- Use the `mkswap` utility to set up the file as Linux swap area:

  `$ sudo mkswap /swapfile`

- Enable the swap with the following command:

  `$ sudo swapon /swapfile`
  
  - [error `swapon: /swapfile: swapon failed: Invalid argument`](https://unix.stackexchange.com/questions/294600/i-cant-enable-swap-space-on-centos-7)
  
    Solution: use `dd` instead of `fallcoate`
    
    `sudo dd if=/dev/zero of=/swapfile count=[size in MiB e.g 2048] bs=1MiB`

- To make the change permanent open the `/etc/fstab` file and append the following line:

  `/swapfile swap swap defaults 0 0`

## Adjust the swappiness value

Swappiness is a Linux kernel property that defines how often the system will use the swap space. Swappiness can have a value between 0 and 100. A low value will make the kernel to try to avoid swapping whenever possible, while a higher value will make the kernel to use the swap space more aggressively.

Reference: https://askubuntu.com/questions/184217/why-most-people-recommend-to-reduce-swappiness-to-10-20

**What is happening when the system is bogged down and swapping heavily?**

>Swapping is a slow and costly operation, so the system avoids it unless it calculates that the trade-off in cache performance will make up for it overall, or if it's necessary to avoid killing processes.
>A lot of the time people will look at their system that is thrashing the disk heavily and using a lot of swap space and blame swapping for it. That's the wrong approach to take. If swapping ever reaches this extreme, it means that swapping is your system's attempt to deal with low memory problems, not the cause of the problem, and that without swapping your running process will just randomly die.

You can check the current swappiness value by typing the following command:

```
$ cat /proc/sys/vm/swappiness
30
```

To set the swappiness value to 30, you would run:

`$ sudo sysctl vm.swappiness=30`

To make this parameter persistent across reboots append the following line to the `/etc/sysctl.conf` file:

`vm.swappiness=30`

The optimal swappiness value depends on your system workload and how the memory is being used. You should adjust this parameter in small increments to find an optimal value.

## Test

To verify that the swap is active, use the `swapon` command as shown below:

```
$ sudo swapon --show
NAME      TYPE SIZE   USED PRIO
/swapfile file   2G 355.8M   -2
```
