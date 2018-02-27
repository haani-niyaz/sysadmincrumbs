

Docker
-------------


## Must Read

**Difference between user space and kernel space**

- https://rhelblog.redhat.com/2015/07/29/architecting-containers-part-1-user-space-vs-kernel-space/
- https://rhelblog.redhat.com/2015/09/17/architecting-containers-part-2-why-the-user-space-matters-2/
- https://rhelblog.redhat.com/2015/11/10/architecting-containers-part-3-how-the-user-space-affects-your-application/
- https://rhelblog.redhat.com/2015/07/07/whats-next-for-containers-user-namespaces/

> When a container is first instantiated, the user space of the container host makes system calls into the kernel to create special data structures in the kernel (cgroups, svirt, namespaces). 


> when a container is created, the clone() system call is used to create a new process. Clone is similar to fork, but allows the new process to be set up within a kernel namespace. Kernel namespaces allow the new process to have its own hostname, IP address, filesystem mount points, process id, and more. A new name space can be created for each new container – allowing them to each look and feel similar to a virtual machine.

See image [here](https://rhelblog.files.wordpress.com/2015/09/q9ul7asyd20ccgzv9sfdbfb7daaifmmzj73umlq-kzw-foekes2lmwubyrqxihp-_gkvdtvglyqqk_nljtw5c2pv48a-ta2li0x2i5chnzayecxhvtrfylckq1vlnft1sjxg0py.png)







