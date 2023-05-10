# Editreadaos
AOS SLIPs
Slip 1
A)	 Write a C program to find whether a given file is present in current directory or not.
Solution:
#include<stdio.h>
#include<dirent.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
int main(int argc,char *argv[])
{
DIR *dirptr;
struct dirent *entry;
int found = 0;
char curDir[20];
getcwd(curDir,20);
if(argc<2)
{
printf("\n Insufficient arguments\n");
exit(1);
}
dirptr = opendir(curDir);
while((entry = readdir(dirptr))!=NULL)
{
if(strcmp(entry->d_name,argv[1])==0)
{
printf("\nFile %s present in current directory\n",argv[1]);
found=1;
break;
}
}
closedir(dirptr);
if(found==0)
printf("\nnFile %s not present in current directory\n",argv[1]);
return 0;
}

B)	Write a C program which blocks SIGOUIT signal for 5 seconds. After 5 second process checks any occurrence of quit signal during this period, if so, it unblock the signal. Now another occurrence of quit signal terminates the program. (Use sigprocmask() and sigpending() )
Solution 
#include<stdio.h>
#include<signal.h>
static void sig_quit(int signo)
{
printf("\nCaught SIG_QUIT");
if(signal(SIGQUIT,SIG_DFL)==SIG_ERR)
printf("\ncan't reset SIGQUIT");
}
int main()
{
sigset_t newmask, oldmask, pendmask;
if(signal(SIGQUIT,sig_quit)==SIG_ERR)
printf("\ncant catch sigquit");
sigemptyset(&newmask);
sigaddset(&newmask, SIGQUIT);
if(sigprocmask(SIG_BLOCK, &newmask, &oldmask)<0)
printf("\nsigblock error");
sleep(5);
printf("old signal set : %8.8ld.\n",oldmask);
if(sigpending(&pendmask)<0)
printf("\nsig-pending error");
printf("pending signal set : %8.8ld.\n",pendmask);
if(sigismember(&pendmask,SIGQUIT))
printf("\nSIGQUIT pending");
if(sigprocmask(SIG_SETMASK, &oldmask, NULL)<0)
printf("\nsig_setmask error");
printf("\nSIGQUIT unblocked");
printf("\nhello\n");
sleep(10);
printf("\nhello\n");
return 0;
}


Slip 3
A)	 Write a C program to find file properties such as inode number, number of hard link, File permissions, File size, File access and modification time and so on of a given file using stat() system call.
Solution:
#include<sys/stat.h>
#include<string.h>
#include<stdio.h>
#include<stdlib.h>
#include<time.h>
#include<sys/types.h>
#include<pwd.h>
#include<grp.h>
int main(int argc,char *argv[])
{
struct stat s;
struct tm *timeinfo;
struct passwd *pw;
struct group*gr;
char filetype,perm,*date;
int i;
memset(&s,0,sizeof(s));
if(argc<2)
{
printf("Insufficient arguments\n");
exit(1);
}
printf("\nFile size \t inode\n");
for( i=1;i<argc;i++)
{
printf("\n");
stat(argv[i],&s);
if((s.st_mode & S_IFMT)==S_IFREG) filetype = 'R';
else if((s.st_mode & S_IFMT)==S_IFSOCK) filetype = 'S';
else if((s.st_mode & S_IFMT)==S_IFLNK) filetype = 'L';
else if((s.st_mode & S_IFMT)==S_IFBLK) filetype = 'B';
else if((s.st_mode & S_IFMT)==S_IFDIR) filetype = 'D';
else if((s.st_mode & S_IFMT)==S_IFCHR) filetype = 'C';
else if((s.st_mode & S_IFMT)==S_IFIFO) filetype = 'F';
printf("%s\t%ld\t%C\t%ld\t%ld",argv[i],s.st_ino,filetype,s.st_size,s.st_nlink);
date = ctime(&s.st_atime);
timeinfo = localtime(&s.st_atime);
printf("\nmonth=%d\n",timeinfo->tm_mon);
printf("\nFile access time = %s",date);
printf("\nFile access time = %s",ctime(&s.st_mtime));
printf("\nFile access time = %s",ctime(&s.st_ctime));
pw = getpwuid(s.st_uid);
gr = getgrgid(s.st_gid);
printf("\n user = %s",pw->pw_name);
printf("\n group = %s",gr->gr_name);
printf((s.st_mode & S_IRUSR)?"r":"-");
printf((s.st_mode & S_IWUSR)?"w":"-");
printf((s.st_mode & S_IXUSR)?"x":"-");
printf((s.st_mode & S_IRGRP)?"r":"-");
printf((s.st_mode & S_IWGRP)?"w":"-");
printf((s.st_mode & S_IXGRP)?"x":"-");
printf((s.st_mode & S_IROTH)?"r":"-");
printf((s.st_mode & S_IWOTH)?"w":"-");
printf((s.st_mode & S_IXOTH)?"x":"-");
}
return 0;
}

Slip 6
A)	Write a C program to map a given file in memory and display the contain of mapped file in reverse.

Solution:
#include<stdio.h>
#include<stdlib.h>
#include<sys/mman.h>
#include<unistd.h>
#include<sys/stat.h>
#include<sys/types.h>
#include<fcntl.h>
#include<string.h>
#include<malloc.h>
char *addr;
int main()
{
int fd,len,stats;
char *rev;
struct stat st;
memset(&st,0,sizeof(st));
fd = open("dip.txt",O_RDONLY);
stats = fstat(fd,&st);
len = st.st_size;
if((addr = mmap(NULL, len, PROT_READ, MAP_PRIVATE,fd,0))==MAP_FAILED)
printf("\nError in mmap");
printf("\nmap=\n%s",addr); // display mapped file
printf("\nmapped file in reverse order\n");
rev = addr + strlen(addr);
while(rev != addr)
{
printf("%c",*rev);
rev--;
}
printf("%c",*rev);
return 0;
}

B)	Write a C program that behaves like a shell (command interpreter). It has its own promp say . Any normal shell command is executed from your shell by starting a child process to
execute the system program corresponding to the command. It should additionally interpret
the following command.
i) list f<dirname> - print name of all files in directory
ii) list n <dirname> - print number of all entries
iii) list i<dirname> - print name and inode of all files

Solution 
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<string.h>
#include<sys/wait.h>
#include<dirent.h>
void separate_tokens(char *cmd,char *tok[])
{
int i=0;
char *p;
p=strtok(cmd," ");
puts(p);
while(p!=NULL)
{
tok[i++]=p;
p=strtok(NULL," ");
}
tok[i]=NULL;
}
void list(char *dirName,char param)
{
DIR *dir;
int count=0;
struct dirent *entry;
// struct stat buff;
if((dir=opendir(dirName))==NULL)
{
printf("\n\tDirectory %s notfound\n",dirName);
return;
}
switch(param)
{
case 'f': while((entry=readdir(dir))!=NULL)
printf("\n%s",entry->d_name);
break;
case 'n': while((entry=readdir(dir))!=NULL)
count++;
printf("\nTotal number of entries = %d\n",count);
break;
case 'i': while((entry=readdir(dir))!=NULL)
printf("\n%ld:%s",entry->d_ino,entry-
>d_name);
break;
}
}
int main()
{
char cmd[80],*args[10];
int pid;
system("clear");
do
{
printf("\nNewShell$ ");
fgets(cmd,80,stdin);
cmd[strlen(cmd)-1]='\0';
separate_tokens(cmd,args);
if(strcmp(args[0],"list")==0)
list(args[2],args[1][0]);
else
{
pid = fork();
if(pid > 0)
wait(0);
else if(execvp(args[0],args)==-1)
printf("\n Command %s not found\n",args[0]);
}
}while(1);
return 0;
}


slip 8
A)	Write a C program to get and set the resource limits such as files, memory associated
with a process.*/
Solution:

#include<stdio.h>
#include<stdlib.h>
#include<sys/time.h>
#include<sys/resource.h>
int main()
{
struct rlimit limit;
if(getrlimit(RLIMIT_NPROC,&limit)<0)
printf("getrlimit error\n");
printf("No. of extant process = [%10ld][%10ld]\n",limit.rlim_max,limit.rlim_cur);
if(getrlimit(RLIMIT_CPU,&limit)<0)
printf("getrlimit error\n");
printf("limit on amount of CPU time that process can consume =
[%ld][%ld]\n",limit.rlim_max,limit.rlim_cur);
if(getrlimit(RLIMIT_DATA,&limit)<0)
printf("getrlimit error\n");
printf("max.size of process's data segment =
[%ld][%ld]\n",limit.rlim_max,limit.rlim_cur);
if(getrlimit(RLIMIT_FSIZE,&limit)<0)
printf("getrlimit error\n");
printf("max. size in bytes of files that process may create =
[%ld][%ld]\n",limit.rlim_max,limit.rlim_cur);
if(getrlimit(RLIMIT_LOCKS,&limit)<0)
printf("getrlimit error\n");
printf("limit on locks = [%ld][%ld]\n",limit.rlim_max,limit.rlim_cur);
if(getrlimit(RLIMIT_MEMLOCK,&limit)<0)
printf("getrlimit error\n");
printf("max. no. of bytes of memory that can be locked in RAM =
[%ld][%ld]\n",limit.rlim_max,limit.rlim_cur);
if(getrlimit(RLIMIT_MSGQUEUE,&limit)<0)
printf("getrlimit error\n");
printf("msg queue = [%ld][%ld]\n",limit.rlim_max,limit.rlim_cur);
return 0;
}

B)	Write a C program which receives file names as command line arguments and display
those filenames in ascending order according to their sizes.
(e.g $ a.out a.txt b.txt c.txt, ...) */

Solution 
#include<sys/stat.h>
#include<string.h>
#include<stdio.h>
#include<stdlib.h>
#include<time.h>
#include<sys/types.h>
#include<pwd.h>
#include<grp.h>
#include<dirent.h>
#include<unistd.h>
struct fileinfo
{
char fileName[20];
int size;
}files[20],temp;
int main(int argc,char *argv[])
{
struct stat s;
memset(&s,0,sizeof(s));
int i,j,n;
for(i=1;i<argc;i++)
{
printf("\n");
stat(argv[i],&s);
strcpy(files[i-1].fileName,argv[i]);
files[i-1].size = s.st_size;
}
n=i-1;
for(i=0;i<n;i++)
{
for(j=i+1;j<n;j++)
{
if(files[i].size > files[j].size)
{
temp = files[i];
files[i]=files[j];
files[j]=temp;
}
}
}
for(i=0;i<n;i++)
printf("\n%s\t%d",files[i].fileName,files[i].size);
return 0;
}

Slip 12
A)	Write a C program to send SIGALRM signal by child process to parent process and parent process make a provision to catch the signal and display alarm is fired.(Use Kill, fork, signal and sleep system call)*/
Solution:

#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<signal.h>
#include<string.h>
#include<sys/wait.h>
static void my_alarm(int signo)
{
printf("\n in signal handler");
alarm(1);
}
int main()
{
int i;
pid_t pid;
signal(SIGALRM,my_alarm);
if((pid=fork())<0)
printf("\nfork error");
if(pid==0)
{
printf("\n child");
alarm(2);
kill(getppid(),SIGALRM);
}
// alarm(2);
else
{
printf("\nparent");
for(i=1;;i++)
{
printf("\n inside main");
sleep(1);
}
}
return 0;
}
B)	 Write a C program to display all the files from current directory and its subdirectory whose size is greater than ’n’ Bytes Where n is accepted from user through command line.
Solution 
#include<sys/stat.h>
#include<string.h>
#include<stdio.h>
#include<stdlib.h>
#include<time.h>
#include<sys/types.h>
#include<pwd.h>
#include<grp.h>
#include<dirent.h>
#include<unistd.h>
int main(int argc,char *argv[])
{
DIR *dirptr,*subdirptr;
struct dirent *entry,*subentry;
struct stat s;
memset(&s,0,sizeof(s));
int n = atoi(argv[1]);
char curDir[80];
getcwd(curDir,80);
if(argc<2)
{
printf("\n Insufficient arguments\n");
exit(1);
}
dirptr = opendir(curDir);
while((entry = readdir(dirptr))!=NULL)
{
stat(entry->d_name,&s);
printf("\n%s",entry->d_name);
if(((s.st_mode & S_IFMT)==S_IFREG) && s.st_size > n)
printf("\n%s : %ld",entry->d_name,s.st_size);
if((s.st_mode & S_IFMT)==S_IFDIR)
{
subdirptr = opendir(entry->d_name);
while((subentry = readdir(subdirptr))!=NULL)
{
stat(subentry->d_name,&s);
if(((s.st_mode & S_IFMT)==S_IFREG) && s.st_size > n)
printf("\n%s : %ld",subentry->d_name,s.st_size);
}
}
}
closedir(dirptr);
return 0;
}
Slip 14
A)	Write a C program to create an unnamed pipe. The child process will write following three messages to pipe and parent process display it.

Message1 = “Hello World” 
Message2 = “Hello SPPU” 
Message3 = “Linux is Funny”

Solution:
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#define MSGSIZE 16
char *msg1 = "Hello World";
char *msg2 = "Hello SPPU";
char *msg3 = "Linux is funny";
int main()
{
int fd[2],i;
char buff[MSGSIZE];
if(pipe(fd)<0)
exit(1);
write(fd[1],msg1,MSGSIZE);
write(fd[1],msg2,MSGSIZE);
write(fd[1],msg3,MSGSIZE);
for(i=0;i<3;i++)
{
read(fd[0],buff,MSGSIZE);
printf("\n%s",buff);
}
return 0;
}

B)	 Write a C program that behaves like a shell (command interpreter). It has its own prompt say “NewShell$”.Any normal shell command is executed from your shell by starting a child process to execute the system program corresponding to the command. It should additionally interpret the following command. 
i) search f - search first occurrence of pattern in filename 
ii) search c - count no. of occurrences of pattern in filename 
iii) search a - search all occurrences of pattern in filename
solution:
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h> 
#include<string.h>
char *buff,*t1,*t2,*t3,*t4,ch;
FILE *fp;
int pid;
void search(char *t2,char *t3,char *t4)
{
    int i=1,count=0;
    char *p;
    if((fp=fopen(t4,"r"))==NULL)
        printf("File not found\n");
    else
    {
        if(strcmp(t2,"f")==0)
        {
            while(fgets(buff,80,fp))
            {
            if((strstr(buff,t3))!=NULL)
                {
            printf("%d: %s\n",i,buff);
                break;
                }
            }
            //i++;
        }
        else if(strcmp(t2,"c")==0)
        {
            while(fgets(buff,80,fp))
            {
            if((strstr(buff,t3))!=NULL)
                {
                    count++;

                }
            }
    printf("No of occurences of %s= %d\n",t3,count);
        }
        else if(strcmp(t2,"a")==0)
        {
            while(fgets(buff,80,fp))
            {
            if((strstr(buff,t3))!=NULL)
                {
            printf("%d: %s\n",i,buff);

                }
            
                //i++;
            }
        }
        else 
            printf("Command not found\n");

        fclose(fp);
    }
}
main()
{
    while(1)
    {
        printf("myshell$");
        fflush(stdin);
        t1=(char *)malloc(80);
        t2=(char *)malloc(80);
        t3=(char *)malloc(80);
        t4=(char *)malloc(80);
        buff=(char *)malloc(80);
        fgets(buff,80,stdin);   
        sscanf(buff,"%s %s %s %s",t1,t2,t3,t4);
        if(strcmp(t1,"pause")==0)
            exit(0);
        else if(strcmp(t1,"search")==0)
            search(t2,t3,t4);
        else
        {
            pid=fork();
            if(pid<0)
        printf("Child process is not created\n");
            else if(pid==0)
            {   
                execlp("/bin",NULL);
                if(strcmp(t1,"exit")==0)
                    exit(0);
                system(buff);
            }
            else
            {   
                wait(NULL);
                exit(0);
            }
        }

    }
}
/*
Output:

 [root@localhost Desktop]# cc shellsearch.c
[root@localhost Desktop]# ./a.out
myshell$ search f aaa f1.txt
myshell$ search f aaa f1.txt
1: aaa

myshell$ search f 123 f1.txt
1: 123456

myshell$ search c a f1.txt
No of occurences of a= 1
myshell$ search c a f1.txt
No of occurences of a= 2
myshell$ search a aa f1.txt
1: aaa

2: aa

myshell$ search a 12 f1.txt
3: 123456

5: 1112

myshell$ pause
[root@localhost Desktop]# 

*/



Slip 15 
A)Write a C program to Identify the type (Directory, character device, Block device, Regular
file, FIFO or pipe, symbolic link or socket) of given file using stat() system call. */
Solution:
#include<sys/stat.h>
#include<string.h>
#include<stdio.h>
#include<stdlib.h>
#include<time.h>
#include<sys/types.h>
#include<pwd.h>
#include<grp.h>
int main(int argc,char *argv[])
{
struct stat s;
char filetype;
memset(&s,0,sizeof(s));
if(argc<2)
{
printf("Insufficient arguments\n");
exit(1);
}
stat(argv[1],&s);
if((s.st_mode & S_IFMT)==S_IFREG) filetype = 'R';
else if((s.st_mode & S_IFMT)==S_IFSOCK) filetype = 'S';
else if((s.st_mode & S_IFMT)==S_IFLNK) filetype = 'L';
else if((s.st_mode & S_IFMT)==S_IFBLK) filetype = 'B';
else if((s.st_mode & S_IFMT)==S_IFDIR) filetype = 'D';
else if((s.st_mode & S_IFMT)==S_IFCHR) filetype = 'C';
else if((s.st_mode & S_IFMT)==S_IFIFO) filetype = 'F';
printf("%s\t%c\n",argv[1],filetype);
return 0;
}

C)	Write a C program which creates a child process and child process catches a signal
SIGHUP, SIGINT and SIGQUIT. The Parent process send a SIGHUP or SIGINT signal after every
3 seconds, at the end of 15 second parent send SIGQUIT signal to child and child terminates
by displaying message "My Papa has Killed me!!!”. 
Solution 

#include<stdio.h>
#include<signal.h>
#include<unistd.h>
#include<stdlib.h>
void sighup(int signo)
{
signal(SIGHUP,sighup);
printf("\nCHILD : I have received SIGHUP");
}
void sigint(int signo)
{
signal(SIGINT,sigint);
printf("\nCHILD : I have received SIGINT");
}
void sigquit(int signo)
{
// signal(SIGQUIT,sigquit);
printf("\nCHILD : My daddy has killed me");
exit(0);
}
int main()
{
int pid;
struct sigaction sigact;
sigact.sa_flags=0;
sigemptyset(&sigact.sa_mask);
sigact.sa_handler = sighup;
if(sigaction(SIGHUP,&sigact,NULL)<0)
{
printf("\nsigaction error");
exit(1);
}
sigact.sa_handler = sigint;
if(sigaction(SIGINT,&sigact,NULL)<0)
{
printf("\nsigaction error");
exit(1);
}
sigact.sa_handler = sigquit;
if(sigaction(SIGQUIT,&sigact,NULL)<0)
{
printf("\nsigaction error");
exit(1);
}
if((pid=fork()) < 0)
{
printf("\nfork error");
exit(1);
}
if(pid == 0) //child
{
for(;;) ;
}
else //parent
{
sigact.sa_handler = SIG_DFL;
sigaction(SIGHUP,&sigact,NULL);
sigaction(SIGINT,&sigact,NULL);
sigaction(SIGQUIT,&sigact,NULL);
printf("\nparent sending SIGHUP");
kill(pid,SIGHUP);
sleep(3);
printf("\nparent sending SIGINT");
kill(pid,SIGINT);
sleep(3);
printf("\nparent sending SIGHUP");
kill(pid,SIGHUP);
sleep(3);
printf("\nparent sending SIGINT");
kill(pid,SIGINT);
sleep(3);
printf("\nparent sending SIGINT");
kill(pid,SIGINT);
sleep(3);
printf("\nparent sending SIGQUIT");
kill(pid,SIGQUIT);
sleep(3);
}
return 0;
}
Slip 16
A)	Write a C program that catches the ctrl-c (SIGINT) signal for the first time and display the appropriate message and exits on pressing ctrl-c again. 
Solution:
#include<stdio.h>
#include<signal.h>
void handle_sigint(int sig)
{
printf("\ncaught signal %d\n",sig);
signal(SIGINT, SIG_DFL);
}
int main()
{
signal(SIGINT,handle_sigint);
while(1)
{
printf("hello world\n");
sleep(1);
}
return 0;
}

B)	Write a C program to implement the following unix/linux command on current directory ls –l > output.txt DO NOT simply exec ls -l > output.txt or system command from the program.
Solution 
#include<sys/stat.h>
#include<string.h>
#include<stdio.h>
#include<stdlib.h>
#include<time.h>
#include<sys/types.h>
#include<pwd.h>
#include<grp.h>
#include<dirent.h>
#include<unistd.h>
#include<fcntl.h>
int main(int argc,char *argv[])
{
DIR *dirptr;
struct dirent *entry;
char curDir[80];
getcwd(curDir,80);
printf("%s\n",curDir);
struct stat s;
struct tm *timeinfo;
struct passwd *pw;
struct group*gr;
int fd;
char filetype,perm,*date;
memset(&s,0,sizeof(s));
fd=open("a.txt",O_CREAT | O_WRONLY);
chmod("a.txt",0777);
close(1);
dup(fd);
dirptr = opendir(curDir);
while((entry = readdir(dirptr))!=NULL)
{
printf("\n");
stat(entry->d_name,&s);
if((s.st_mode & S_IFMT)==S_IFREG) filetype = '-';
else if((s.st_mode & S_IFMT)==S_IFSOCK) filetype = 'S';
else if((s.st_mode & S_IFMT)==S_IFLNK) filetype = 'L';
else if((s.st_mode & S_IFMT)==S_IFBLK) filetype = 'B';
else if((s.st_mode & S_IFMT)==S_IFDIR) filetype = 'D';
else if((s.st_mode & S_IFMT)==S_IFCHR) filetype = 'C';
else if((s.st_mode & S_IFMT)==S_IFIFO) filetype = 'F';
date = ctime(&s.st_atime);
timeinfo = localtime(&s.st_atime);
date[strlen(date)-1]='\0';
pw = getpwuid(s.st_uid);
gr = getgrgid(s.st_gid);
printf("%c",filetype);
printf((s.st_mode & S_IRUSR)?"r":"-");
printf((s.st_mode & S_IWUSR)?"w":"-");
printf((s.st_mode & S_IXUSR)?"x":"-");
printf((s.st_mode & S_IRGRP)?"r":"-");
printf((s.st_mode & S_IWGRP)?"w":"-");
printf((s.st_mode & S_IXGRP)?"x":"-");
printf((s.st_mode & S_IROTH)?"r":"-");
printf((s.st_mode & S_IWOTH)?"w":"-");
printf((s.st_mode & S_IXOTH)?"x":"-");
printf(" %ld %s %s %ld\t%s %s",s.st_nlink,pw->pw_name,pw-
>pw_name,s.st_size,date,entry->d_name);
}
close(fd);
return 0;
}
Slip 19
A)	Write a C program to move the content of file1.txt to file2.txt and remove the file1.txt
from directory.
Solution:
#include<stdio.h>
#include<fcntl.h>
#include<unistd.h>
#include<stdlib.h>
int main()
{
char ch;
int fd1 = open("dip.txt",O_RDONLY);
int fd2 = creat("dip1.txt",O_CREAT | O_WRONLY);
while((read(fd1,&ch,1)!=0))
write(fd2,&ch,1);
close(fd1);
close(fd2);
unlink("dip.txt");
return 0;
}

B)	Write a C program which creates a child process to run linux/ unix command or any user defined program. The parent process set the signal handler for death of child signal and Alarm signal. If a child process does not complete its execution in 5 second then parent process kills child process.

Solution 
#include<stdio.h>
#include<unistd.h>
#include<sys/wait.h>
#include<signal.h>
pid_t pid;
static void sig_handler(int signo)
{
// if(signo==SIG_ERR)
// printf("\n sig err");
if(signo == SIGCHLD)
printf("\nchild signal");
if(signo == SIGALRM)
{
printf("\n alarm signal");
kill(pid,SIGKILL);
}
}
int main()
{
signal(SIGCHLD,sig_handler);
signal(SIGALRM,sig_handler);
if((pid=fork())<0)
printf("\nfork error");
if(pid==0)
{
sleep(5);
execlp("ls","ls","-l",NULL);
}
alarm(5);
wait(NULL);
return 0;
}
Slip 20
A)	Write a C program that print the exit status of a terminated child process
solution
#include<stdio.h>
#include<sys/wait.h>
#include<stdlib.h>
#include<sys/types.h>
#include<unistd.h>
void pr_exit(int status)
{
if(WIFEXITED(status))
printf("\nnormal termination\nexit status = %d\n",WEXITSTATUS(status));
else if(WIFSIGNALED(status))
printf("\nabnormal termination\nsignal number =
%d%s\n",WTERMSIG(status),
#ifdef WCOREDUMP
WCOREDUMP(status)? "(Core file generated)":"");
#else
"");
#endif
else if(WIFSTOPPED(status))
printf("\nchild stopped \nsignal number = %d\n",WSTOPSIG(status));
}
int main()
{
pid_t pid;
int status;
if((pid=fork())<0)
printf("fork error");
else if(pid==0) //child
exit(7);
if(wait(&status) != pid) //wait for child
printf("wait error");
pr_exit(status); // & print its status
if((pid=fork())<0)
printf("fork error");
else if(pid==0) //child
abort(); // generates SIGABRT
if(wait(&status) != pid) //wait for child
printf("wait error");
pr_exit(status);
if((pid=fork())<0)
printf("fork error");
else if(pid==0) //child
status/=0; // divide by 0 generates SIGFPE
if(wait(&status) != pid) //wait for child
printf("wait error");
pr_exit(status);
return 0;
}

    
    
    
    
    ANDROID STUDIO
-:Programs:-
Slip (1)
Q.1 A) Write an Android Program to demonstrate Activity life Cycle?
-:Program:-
activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>

MainActivity.java
package com.example.activitylifecycle;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.util.Log;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toast toast = Toast.makeText(this,"onCreate Called",Toast.LENGTH_LONG);
        toast.show();

    }
    protected void onStart() {
        super.onStart();
        Toast toast = Toast.makeText(this, "onStart Called", Toast.LENGTH_LONG);
        toast.show();
    }
    protected void onRestart() {
        super.onRestart();
        Toast toast = Toast.makeText(this, "onRestart Called", Toast.LENGTH_LONG);
        toast.show();
    }
    protected void onPause() {
        super.onPause();
        Toast toast = Toast.makeText(this, "onPause Called", Toast.LENGTH_LONG);
        toast.show();
    }
    protected void onResume() {
        super.onResume();
        Toast toast = Toast.makeText(this, "onResume Called", Toast.LENGTH_LONG);
        toast.show();
    }

    protected void onStop() {
        super.onStop();
        Toast toast = Toast.makeText(this, "onStop Called", Toast.LENGTH_LONG);
        toast.show();
    }
    protected void onDestroy() {
        super.onDestroy();
        Toast toast = Toast.makeText(this, "onDestroy Called", Toast.LENGTH_LONG);
        toast.show();
    }
}










B) Create table Customer (id, name, address, phno). Create Android Application for performing the following operation on the table. (usingsqlite database)
 i) Insert New Customer Details.
 ii) Show All the Customer Details



Activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
 xmlns:app="http://schemas.android.com/apk/res-auto"
 xmlns:tools="http://schemas.android.com/tools"
 android:layout_width="match_parent"
 android:layout_height="match_parent"
 android:orientation="vertical"
 tools:context=".MainActivity">
 <EditText
 android:id="@+id/id"
 android:layout_width="match_parent"
 android:layout_height="wrap_content"
 android:layout_margin="10dp"
 android:hint="Enter id" />
 <EditText
 android:id="@+id/name"
 android:layout_width="match_parent"
 android:layout_height="wrap_content"
 android:layout_margin="10dp"
 android:hint="Enter Name" />
 <EditText
 android:id="@+id/address"
 android:layout_width="match_parent"
 android:layout_height="wrap_content"
 android:layout_margin="10dp"
 android:hint="Address" />
 <Button
 android:id="@+id/btnadd"
 android:layout_width="match_parent"
 android:layout_height="wrap_content"
 android:layout_margin="10dp"
 android:text="Add Customer"
 android:textAllCaps="false" />
 <Button
 android:layout_width="match_parent"
 android:layout_height="wrap_content"
 android:text="View"
 android:id="@+id/btnview"
 android:layout_margin="10dp"
 />
 <TextView
 android:layout_width="wrap_content"
 android:layout_height="wrap_content"
 android:id="@+id/txt"
 />
</LinearLayout>
MainAct.java
package com.example.customer;
import androidx.appcompat.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;
public class MainActivity extends AppCompatActivity {
 private EditText id,name,address;
 private Button btn,btnview;
 private DBHandler dbHandler;
 TextView tv;
 @Override
 protected void onCreate(Bundle savedInstanceState) {
 super.onCreate(savedInstanceState);
 setContentView(R.layout.activity_main);
 id= findViewById(R.id.id);
 name = findViewById(R.id.name);
 address = findViewById(R.id.address);
 btn = findViewById(R.id.btnadd);
 btnview=findViewById(R.id.btnview);
 tv=findViewById(R.id.txt);
 // creating a new dbhandler class
 // and passing our context to it.
 dbHandler = new DBHandler(MainActivity.this);
 // below line is to add on click listener for our add course button.
 btn.setOnClickListener(new View.OnClickListener() {
 @Override
 public void onClick(View v) {
 // below line is to get data from all edit text fields.
String id1 = id.getText().toString();
 String name1 = name.getText().toString();
String address1 = address.getText().toString();
 // validating if the text fields are empty or not.
 if (id1.isEmpty() && name1.isEmpty() && address1.isEmpty()) {
 Toast.makeText(MainActivity.this, "Please enter all the
data..", Toast.LENGTH_SHORT).show();
 return;
 }
 // on below line we are calling a method to add new
// course to sqlite data and pass all our values to it.
dbHandler.addcustomer(id1,name1,address1);
 // after adding the data we are displaying a toast message.
Toast.makeText(MainActivity.this, "Customer has been added.",
Toast.LENGTH_SHORT).show();
 id.setText("");
 name.setText("");
 address.setText("");
 tv.setText("Inserted");
 //tv.setText("");
 }
 });
 btnview.setOnClickListener(new View.OnClickListener() {
 @Override
 public void onClick(View v) {
 tv.setText(dbHandler.getRecords());
 }
 });
 }
}
DBHandler.java
package com.example.customer;
import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;
public class DBHandler extends SQLiteOpenHelper {
 private static final String DB_NAME = "custdb";
 // below int is our database version
 private static final int DB_VERSION = 1;
 // below variable is for our table name.
 private static final String TABLE_NAME = "customer";
 // below variable is for our id column.
 private static final String ID_COL = "id";
 // below variable is for our course name column
 private static final String NAME_COL = "name";
 // below variable id for our course duration column.
 private static final String Address_COL = "address";
 // creating a constructor for our database handler.
 public DBHandler(Context context) {
 super(context, DB_NAME, null, DB_VERSION);
 }
 @Override
 public void onCreate(SQLiteDatabase db) {
 String query = "CREATE TABLE " + TABLE_NAME + " ("
 + ID_COL + " INTEGER PRIMARY KEY AUTOINCREMENT, "
 + NAME_COL + " TEXT,"
 + Address_COL + " TEXT)";
 // at last we are calling a exec sql
 // method to execute above sql query
 db.execSQL(query);
 }
 // this method is use to add new course to our sqlite database.
 public void addcustomer(String id1, String name1, String address1) {
 // on below line we are creating a variable for
 // our sqlite database and calling writable method
 // as we are writing data in our database.
 SQLiteDatabase db = this.getWritableDatabase();
 // on below line we are creating a
 // variable for content values.
 ContentValues values = new ContentValues();
 // on below line we are passing all values
 // along with its key and value pair.
 values.put(NAME_COL, name1);
 values.put(Address_COL,address1);
 // after adding all values we are passing
 // content values to our table.
 db.insert(TABLE_NAME, null, values);
 // at last we are closing our
 // database after adding database.
 db.close();
 }
 @Override
 public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion)
{
 // this method is called to check if the table exists already.
 db.execSQL("DROP TABLE IF EXISTS " + TABLE_NAME);
 onCreate(db);
 }
 public String getRecords(){
 String query="SELECT * FROM "+TABLE_NAME;
 String result="";
 SQLiteDatabase db=this.getReadableDatabase();
 Cursor cursor=db. rawQuery(query,null);//rawquery returns a cursor
over result set.
 cursor.moveToFirst();//moves cursor to 1st row in query result.
 while(cursor.isAfterLast()==false) //isAfterLast returns true if read
all positions in your cursor & false otherwise.
 {
 result+=cursor.getString(0)+" "+cursor.getString(1)+"
"+cursor.getString(2)+"\n";
 cursor.moveToNext(); // moves cursor to next row.
 }
 db.close();
 return result;

}
}





                       Slip (3)
Q.1 A) Create an Android Application to accept two numbers and create two buttons (power and Average). Display the result on the next activity on Button click
-:Program:-
activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <EditText
        android:id="@+id/ed1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter first number"
        />



    <EditText
        android:id="@+id/ed2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter second number"
        tools:layout_editor_absoluteX="166dp"
        tools:layout_editor_absoluteY="183dp" />

    <Button
        android:id="@+id/power"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Power"
        />
    <Button
        android:id="@+id/average"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Average"
        />
</LinearLayout>

activity_afterclick.xml

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"

    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".afterclick">
    <TextView
        android:id="@+id/txt1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="the result is:"
        android:layout_gravity="center_horizontal"
        android:textSize="25sp"
        />


</LinearLayout>


afterclick.java
package com.example.myapplication;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.widget.TextView;

public class afterclick extends AppCompatActivity {
TextView t1;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_afterclick);
        t1=(TextView)findViewById(R.id.txt1);
        String value = getIntent().getStringExtra("value");
        t1.setText("the result is:"+""+value);

    }
}


MainActivity.java
package com.example.myapplication;

import androidx.appcompat.app.AppCompatActivity;
import androidx.appcompat.widget.ButtonBarLayout;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;

public class MainActivity extends AppCompatActivity {
 EditText ed1,ed2;

 TextView txt;

 Button power,average;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ed1=(EditText) findViewById(R.id.ed1);
        ed2=(EditText)findViewById(R.id.ed2);
        power=(Button)findViewById(R.id.power);
        average=(Button)findViewById(R.id.average);

        power.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                double no1=Double.valueOf(ed1.getText().toString());
                double no2=Double.valueOf(ed2.getText().toString());
                double total=(no1/no2)/2;
                Intent intent=new Intent(MainActivity.this,afterclick.class);
                intent.putExtra("value",String.valueOf(total));
                startActivity(intent);
            }
        });


    }
}

AndroidManifest.xml

<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/Theme.MyApplication"
        tools:targetApi="31">
        <activity
            android:name=".afterclick"
            android:exported="false" />
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

    </application>

</manifest>
 

B) Create an Android Application to perform following string operation according to user selection of radio button..
 
-:Program:-
activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<TableLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">


    <TableRow>
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Hello World!"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

        <EditText
            android:id="@+id/input"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:hint="Enter String"/>


    </TableRow>
    <TableRow>
        <RadioGroup
            android:id="@+id/rg"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content">
            <RadioButton
                android:id="@+id/r1"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Uppercase" />
            <RadioButton
                android:id="@+id/r2"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="LowerCase" />
            <RadioButton
                android:id="@+id/r3"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Right 5 char" />

            <RadioButton
                android:id="@+id/r4"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="left 5 char" />
        </RadioGroup>
    </TableRow>

    <TableRow>
        <Button
            android:id="@+id/btn"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="output"/>
        <EditText
            android:id="@+id/output"
            android:layout_width="250dp"
            android:layout_height="wrap_content"/>
    </TableRow>


</TableLayout>

MainActivity.java
package com.example.myapplication;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.RadioGroup;

import java.util.Locale;

public class MainActivity extends AppCompatActivity {
    EditText input,output;

    Button btn;

    RadioGroup rg;

    String inpstr,outstr;

    @Override
    protected void onCreate(Bundle savedInstanceState) {

        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        input=(EditText)findViewById(R.id.input);
        output=(EditText)findViewById(R.id.output);
        rg=(RadioGroup)findViewById(R.id.rg);
        btn=(Button)findViewById(R.id.btn);
        btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                int id=rg.getCheckedRadioButtonId();
                switch(id){
                    case R.id.r1:inpstr=input.getText().toString();
                    outstr=inpstr.toUpperCase();
                    output.setText(outstr);
                    break;
                    case R.id.r2:inpstr=input.getText().toString();
                    outstr=inpstr.toLowerCase();
                    output.setText(outstr);
                    break;
                    case R.id.r3:inpstr=input.getText().toString();
                        output.setText(" "+inpstr.substring(input.length()-5));
                        break;


                    case R.id.r4:inpstr=input.getText().toString();
                        output.setText(" "+inpstr.substring(0,5));
                        break;
                }
            }
        });

    }
}

 























Slip (14)
Q.1 A) Construct an Android app that toggles a light bulb on and off when the user clicks on toggle button [10] 
-:Program:-
activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <ToggleButton
        android:id="@+id/toggleButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="8dp"
        android:layout_marginTop="80dp"
        android:text="ToggleButton"
        android:textOff="Off"
        android:textOn="On"
        />

    <ToggleButton
        android:id="@+id/toggleButton2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginRight="20dp"
        android:layout_marginTop="80dp"
        android:text="ToggleButton"
        android:textOff="Off"
        android:textOn="On"
        android:layout_marginLeft="30dp"
         />
    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
       android:layout_marginTop="300dp"
        android:text="Submit" />
</LinearLayout>


MainActivity.java

package com.example.togglebutton;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.Toast;
import android.widget.ToggleButton;

public class MainActivity extends AppCompatActivity {
    private ToggleButton toggleButton1, toggleButton2;
    private Button buttonSubmit;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        toggleButton1=(ToggleButton)findViewById(R.id.toggleButton);
        toggleButton2=(ToggleButton)findViewById(R.id.toggleButton2);
        buttonSubmit=(Button)findViewById(R.id.button);
        buttonSubmit.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                StringBuilder result = new StringBuilder();
                result.append("ToggleButton1 : ").append(toggleButton1.getText());
                result.append("\nToggleButton2 : ").append(toggleButton2.getText());
                //Displaying the message in toast
                Toast.makeText(getApplicationContext(), result.toString(),Toast.LENGTH_LONG).show();
            }
        });
    }
}


B) Construct an Android application to accept a number and calculate Armstrong and Perfect number of a given number using Menu.
Slip (15)
Q.1 A) Write an Android code to merge given two Array/List
 
-:Program:-
activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="30sp"
        android:text="Array 1:"
        android:id="@+id/tv1">
    </TextView>
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="30dp"
        android:id="@+id/ed1"
        android:layout_toRightOf="@+id/tv1"
        android:layout_marginLeft="20sp">
    </EditText>
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="30sp"
        android:layout_below="@+id/tv1"
        android:textSize="30sp"
        android:text="Array 2:"
        android:id="@+id/tv2">
    </TextView>
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="30sp"
        android:id="@+id/ed2"
        android:layout_toRightOf="@+id/tv2"
        android:layout_below="@+id/ed1"
        android:layout_marginLeft="20sp">
    </EditText>
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="30sp"
        android:text="Array 3:"
        android:layout_marginTop="30sp"
        android:layout_below="@+id/tv2"
        android:id="@+id/tv3">
    </TextView>
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="30sp"
        android:id="@+id/ed3"
        android:layout_toRightOf="@+id/tv3"
        android:layout_below="@+id/ed2"
        android:layout_marginLeft="20sp">
    </EditText>
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="30sp"
        android:id="@+id/ed3"
        android:layout_toRightOf="@+id/tv3"
        android:layout_below="@+id/ed2"
        android:layout_marginLeft="20sp">
    </EditText>
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="30sp"
        android:layout_marginTop="30sp"
        android:layout_below="@+id/ed3"
        android:text="show"
        android:id="@+id/btn1">
    </Button>
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/txt1"
        android:layout_marginTop="300dp"
        android:textSize="30dp"/>
</RelativeLayout>

MainActivity.java
package com.example.myapplication;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;

import java.util.ArrayList;
import java.util.Collection;
import java.util.Collections;
import java.util.List;

public class MainActivity extends AppCompatActivity {
    EditText ed1,ed2,ed3;
    Button btn1;
    TextView tv;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ed1=findViewById(R.id.ed1);
        ed2=findViewById(R.id.ed2);
        ed3=findViewById(R.id.ed3);
        btn1=findViewById(R.id.btn1);
        tv=findViewById(R.id.txt1);
        btn1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String s1=ed1.getText().toString();
                String s2=ed2.getText().toString();
                String s3=ed3.getText().toString();
                List<String> l1=new ArrayList<String>(Collections.singleton(s1));
                List<String> l2=new ArrayList<String>(Collections.singleton(s2));
                List<String> l3=new ArrayList<String>(Collections.singleton(s3));
                List<String> l=new ArrayList<String>();
                l1.addAll(l2);
                l1.addAll(l3);
                tv.setText("Output is"+l1);

            }
        });
    }
}

B) Write an Android Application to send Email.
activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/to"
        android:hint="to"/>
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/sub"
        android:hint="sub"/>
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/msg"
        android:hint="msg"/>
    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/send"
        android:text="send"/>
</LinearLayout>

MainActivity.java
package com.example.myapplication;

import androidx.appcompat.app.AppCompatActivity;

import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;

public class MainActivity extends AppCompatActivity {
    EditText etto,etmessage,etsubject;
    Button btsend;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        etto=findViewById(R.id.to);
        etmessage=findViewById(R.id.msg);
        etsubject=findViewById(R.id.sub);
        btsend=findViewById(R.id.send);
        btsend.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent=new Intent(Intent.ACTION_VIEW,
                       Uri.parse("mailto:"+etto.getText().toString()));
                intent.putExtra(Intent.EXTRA_SUBJECT,etsubject.getText().toString());
                intent.putExtra(Intent.EXTRA_TEXT,etmessage.getText().toString());
                startActivity(intent);

            }
        });
    }
}
Slip (20)
Q.1 A) Write an Android application to accept two numbers from the user, and displays them, but reject input if both numbers are greater than 10 and asks for two new numbers. [10]
 B) Create the simple calculator shown below also perform appropriate operation
                    
activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/relative1"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
    <EditText
        android:id="@+id/edt1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
    <Button
        android:id="@+id/button1"
        style="?android:attr/buttonStyleSmall"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignEnd="@+id/button4"
        android:layout_alignRight="@+id/button4"
        android:layout_below="@+id/edt1"
        android:layout_marginTop="94dp"
        android:text="1" />
    <Button
        android:id="@+id/button2"
        style="?android:attr/buttonStyleSmall"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignTop="@+id/button1"
        android:layout_toLeftOf="@+id/button3"
        android:layout_toStartOf="@+id/button3"
        android:text="2" />
    <Button
        android:id="@+id/button3"
        style="?android:attr/buttonStyleSmall"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignTop="@+id/button2"
        android:layout_centerHorizontal="true"
        android:text="3" />
    <Button
        android:id="@+id/button4"
        style="?android:attr/buttonStyleSmall"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/button1"
        android:layout_toLeftOf="@+id/button2"
        android:text="4" />
    <Button
        android:id="@+id/button5"
        style="?android:attr/buttonStyleSmall"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignBottom="@+id/button4"
        android:layout_alignLeft="@+id/button2"
        android:layout_alignStart="@+id/button2"
        android:text="5" />
    <Button
        android:id="@+id/button6"
        style="?android:attr/buttonStyleSmall"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignLeft="@+id/button3"
        android:layout_alignStart="@+id/button3"
        android:layout_below="@+id/button3"
        android:text="6" />
    <Button
        android:id="@+id/button7"
        style="?android:attr/buttonStyleSmall"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/button4"
        android:layout_toLeftOf="@+id/button2"
        android:text="7" />
    <Button
        android:id="@+id/button8"
        style="?android:attr/buttonStyleSmall"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignLeft="@+id/button5"
        android:layout_alignStart="@+id/button5"
        android:layout_below="@+id/button5"
        android:text="8" />
    <Button
        android:id="@+id/button9"
        style="?android:attr/buttonStyleSmall"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignLeft="@+id/button6"
        android:layout_alignStart="@+id/button6"
        android:layout_below="@+id/button6"
        android:text="9" />
    <Button
        android:id="@+id/buttonadd"
        style="?android:attr/buttonStyleSmall"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignEnd="@+id/edt1"
        android:layout_alignRight="@+id/edt1"
        android:layout_alignTop="@+id/button3"
        android:layout_marginLeft="46dp"
        android:layout_marginStart="46dp"
        android:layout_toRightOf="@+id/button3"
        android:text="+" />
    <Button
        android:id="@+id/buttonsub"
        style="?android:attr/buttonStyleSmall"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignEnd="@+id/buttonadd"
        android:layout_alignLeft="@+id/buttonadd"
        android:layout_alignRight="@+id/buttonadd"
        android:layout_alignStart="@+id/buttonadd"
        android:layout_below="@+id/buttonadd"
        android:text="-" />
    <Button
        android:id="@+id/buttonmul"
        style="?android:attr/buttonStyleSmall"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignLeft="@+id/buttonsub"
        android:layout_alignParentEnd="true"
        android:layout_alignParentRight="true"
        android:layout_alignStart="@+id/buttonsub"
        android:layout_below="@+id/buttonsub"
        android:text="*" />
    <Button
        android:id="@+id/button10"
        style="?android:attr/buttonStyleSmall"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/button7"
        android:layout_toLeftOf="@+id/button2"
        android:text="." />
    <Button
        android:id="@+id/button0"
        style="?android:attr/buttonStyleSmall"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignLeft="@+id/button8"
        android:layout_alignStart="@+id/button8"
        android:layout_below="@+id/button8"
        android:text="0" />
    <Button
        android:id="@+id/buttonC"
        style="?android:attr/buttonStyleSmall"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignLeft="@+id/button9"
        android:layout_alignStart="@+id/button9"
        android:layout_below="@+id/button9"
        android:text="C" />
    <Button
        android:id="@+id/buttondiv"
        style="?android:attr/buttonStyleSmall"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignEnd="@+id/buttonmul"
        android:layout_alignLeft="@+id/buttonmul"
        android:layout_alignRight="@+id/buttonmul"
        android:layout_alignStart="@+id/buttonmul"
        android:layout_below="@+id/buttonmul"
        android:text="/" />
    <Button
        android:id="@+id/buttoneql"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignEnd="@+id/buttondiv"
        android:layout_alignLeft="@+id/button10"
        android:layout_alignRight="@+id/buttondiv"
        android:layout_alignStart="@+id/button10"
        android:layout_below="@+id/button0"
        android:layout_marginTop="37dp"
        android:text="=" />
</RelativeLayout>

MainActivity.java
package com.example.calculator;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;

public class MainActivity extends AppCompatActivity {

    Button button0, button1, button2, button3, button4, button5, button6,
            button7, button8, button9, buttonAdd, buttonSub, buttonDivision,
            buttonMul, button10, buttonC, buttonEqual;
    EditText crunchifyEditText;
    float mValueOne, mValueTwo;
    boolean crunchifyAddition, mSubtract, crunchifyMultiplication, crunchifyDivision;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        button0 = (Button) findViewById(R.id.button0);
        button1 = (Button) findViewById(R.id.button1);
        button2 = (Button) findViewById(R.id.button2);
        button3 = (Button) findViewById(R.id.button3);
        button4 = (Button) findViewById(R.id.button4);
        button5 = (Button) findViewById(R.id.button5);
        button6 = (Button) findViewById(R.id.button6);
        button7 = (Button) findViewById(R.id.button7);
        button8 = (Button) findViewById(R.id.button8);
        button9 = (Button) findViewById(R.id.button9);
        button10 = (Button) findViewById(R.id.button10);
        buttonAdd = (Button) findViewById(R.id.buttonadd);
        buttonSub = (Button) findViewById(R.id.buttonsub);
        buttonMul = (Button) findViewById(R.id.buttonmul);
        buttonDivision = (Button) findViewById(R.id.buttondiv);
        buttonC = (Button) findViewById(R.id.buttonC);
        buttonEqual = (Button) findViewById(R.id.buttoneql);
        crunchifyEditText = (EditText) findViewById(R.id.edt1);
        button1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                crunchifyEditText.setText(crunchifyEditText.getText() + "1");
            }
        });
        button2.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                crunchifyEditText.setText(crunchifyEditText.getText() + "2");
            }
        });
        button3.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                crunchifyEditText.setText(crunchifyEditText.getText() + "3");
            }
        });
        button4.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                crunchifyEditText.setText(crunchifyEditText.getText() + "4");
            }
        });
        button5.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                crunchifyEditText.setText(crunchifyEditText.getText() + "5");
            }
        });
        button6.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                crunchifyEditText.setText(crunchifyEditText.getText() + "6");
            }
        });
        button7.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                crunchifyEditText.setText(crunchifyEditText.getText() + "7");
            }
        });
        button8.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                crunchifyEditText.setText(crunchifyEditText.getText() + "8");
            }
        });
        button9.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                crunchifyEditText.setText(crunchifyEditText.getText() + "9");
            }
        });
        button0.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                crunchifyEditText.setText(crunchifyEditText.getText() + "0");
            }
        });
        buttonAdd.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (crunchifyEditText == null) {
                    crunchifyEditText.setText("");
                } else {
                    mValueOne = Float.parseFloat(crunchifyEditText.getText() + "");
                    crunchifyAddition = true;
                    crunchifyEditText.setText(null);
                }
            }
        });
        buttonSub.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mValueOne = Float.parseFloat(crunchifyEditText.getText() + "");
                mSubtract = true;
                crunchifyEditText.setText(null);
            }
        });
        buttonMul.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mValueOne = Float.parseFloat(crunchifyEditText.getText() + "");
                crunchifyMultiplication = true;
                crunchifyEditText.setText(null);
            }
        });
        buttonDivision.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mValueOne = Float.parseFloat(crunchifyEditText.getText() + "");
                crunchifyDivision = true;
                crunchifyEditText.setText(null);
            }
        });
        buttonEqual.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mValueTwo = Float.parseFloat(crunchifyEditText.getText() + "");
                if (crunchifyAddition == true) {
                    crunchifyEditText.setText(mValueOne + mValueTwo + "");
                    crunchifyAddition = false;
                }
                if (mSubtract == true) {
                    crunchifyEditText.setText(mValueOne - mValueTwo + "");
                    mSubtract = false;
                }
                if (crunchifyMultiplication == true) {
                    crunchifyEditText.setText(mValueOne * mValueTwo + "");
                    crunchifyMultiplication = false;
                }
                if (crunchifyDivision == true) {
                    crunchifyEditText.setText(mValueOne / mValueTwo + "");
                    crunchifyDivision = false;
                }
            }
        });
        buttonC.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                crunchifyEditText.setText("");
            }
        });
        button10.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                crunchifyEditText.setText(crunchifyEditText.getText() + ".");
            }
        });
    }
}













Slip (6)
 



<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
 xmlns:app="http://schemas.android.com/apk/res-auto"
 xmlns:tools="http://schemas.android.com/tools"
 android:layout_width="match_parent"
 android:layout_height="match_parent"
 android:orientation="vertical"
 tools:context=".MainActivity">
 <EditText
 android:layout_width="match_parent"
 android:layout_height="wrap_content"
 android:id="@+id/to"
 android:hint="to"/>
 <EditText
 android:layout_width="match_parent"
 android:layout_height="wrap_content"
 android:id="@+id/sub"
 android:hint="sub"/>
 <EditText
 android:layout_width="match_parent"
 android:layout_height="wrap_content"
 android:id="@+id/msg"
 android:hint="msg"/>
 <Button
 android:layout_width="match_parent"
 android:layout_height="wrap_content"
 android:id="@+id/send"
 android:text="Send"/>
</LinearLayout>
MainActivity.java
package com.example.email;
import androidx.appcompat.app.AppCompatActivity;
import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
public class MainActivity extends AppCompatActivity {
 EditText etto,etmessage,etsubject;
 Button btsend;
 @Override
 protected void onCreate(Bundle savedInstanceState) {
 super.onCreate(savedInstanceState);
 setContentView(R.layout.activity_main);
 etto=findViewById(R.id.to);
 etmessage=findViewById(R.id.msg);
 etsubject=findViewById(R.id.sub);
 btsend=findViewById(R.id.send);
 btsend.setOnClickListener(new View.OnClickListener() {
 @Override
 public void onClick(View v) {
 Intent intent =new Intent(Intent.ACTION_VIEW,
 Uri.parse("mailto:"+ etto.getText().toString()));

intent.putExtra(Intent.EXTRA_SUBJECT,etsubject.getText().toString());

intent.putExtra(Intent.EXTRA_TEXT,etmessage.getText().toString());
 startActivity(intent);
 }
 });
 }
}
Slip (8)
 

<?xml version="1.0" encoding="utf-8"?>
<TableLayout xmlns:android="http://schemas.android.com/apk/res/android"
 xmlns:app="http://schemas.android.com/apk/res-auto"
 xmlns:tools="http://schemas.android.com/tools"
 android:layout_width="match_parent"
 android:layout_height="match_parent"
 tools:context=".MainActivity">
 <TableRow
 android:layout_width="wrap_content"
 android:layout_height="wrap_content"
 android:orientation="horizontal">
 <TextView
 android:layout_width="wrap_content"
 android:layout_height="wrap_content"
 android:text="List" />
 <EditText
 android:id="@+id/l"
 android:layout_width="200dp"
 android:layout_height="50dp"
 android:hint="Enter 5 numbers" />
 </TableRow>
 <TableRow
 android:layout_height="wrap_content"
 android:layout_width="wrap_content"
 android:orientation="horizontal">
 <Button
 android:layout_width="wrap_content"
 android:layout_height="wrap_content"
 android:text="Ans"
 android:id="@+id/ans"/>
 <EditText
 android:layout_height="50dp"
 android:layout_width="200dp"
 android:id="@+id/et" />
 </TableRow>
 <TableRow
 android:layout_height="wrap_content"
 android:layout_width="wrap_content"
 android:orientation="vertical">
 <RadioGroup
 android:layout_width="wrap_content"
 android:layout_height="wrap_content"
 android:id="@+id/rg">
 <RadioButton
 android:layout_width="wrap_content"
android:layout_height="wrap_content"
android:text="Sum"
 android:id="@+id/rd1"
/>
 <RadioButton
 android:layout_width="wrap_content"
 android:layout_height="wrap_content"
 android:text="Average"
 android:id="@+id/rd2"
 />
 </RadioGroup>
 </TableRow>
</TableLayout>
MainActivity
package com.example.sumaverage;
import androidx.appcompat.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.RadioGroup;
import android.widget.Toast;
public class MainActivity extends AppCompatActivity {
 RadioGroup rg;
 Button ans;
 EditText l, et;
 @Override
 protected void onCreate(Bundle savedInstanceState) {
 super.onCreate(savedInstanceState);
 setContentView(R.layout.activity_main);
 rg=(RadioGroup)findViewById(R.id.rg);
 ans=(Button) findViewById(R.id.ans);
 et = (EditText) findViewById(R.id.et);
 ans.setOnClickListener(new View.OnClickListener() {
 @Override
 public void onClick(View view) {
 l= (EditText) findViewById(R.id.l);
 String str = l.getText().toString();
 String[] arr = str.split(",");
 int id = rg.getCheckedRadioButtonId();
 switch (id){
 case R.id.rd1:int sum=0;
 for (int i=0; i<arr.length; i++) {
 sum += Integer.parseInt(arr[i]);
et.setText("Sum=" + sum);
 }
 break;
 case R.id.rd2: float avg;
 sum=0;
 for (int i=0; i<arr.length; i++){
 sum+= Integer.parseInt(arr[i]);
 }
 avg=sum/arr.length;
et.setText("Average="+avg);
 break;
 default:
 Toast.makeText(MainActivity.this, "Please select
radio button",
 Toast.LENGTH_SHORT).show();
 }
 }
 });
 }
} 

B) Create a Notification in Android and display the notification message on second activity.
Activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="ANDROID NOTIFICATION"
        android:textSize="20dp"
        />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/button"
        android:text="Notify"
         />

</LinearLayout>


Second.xml

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <TextView
        android:id="@+id/textView2"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:text="your detail of notification..."
         />

    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="8dp"
        android:layout_marginEnd="8dp"
        android:layout_marginStart="8dp"
        android:layout_marginTop="8dp"
       />

</LinearLayout>

MainActivity.java
package com.example.notification;

import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.NotificationCompat;

import android.app.NotificationManager;
import android.app.PendingIntent;
import android.content.Context;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;

public class MainActivity extends AppCompatActivity {
    Button button;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        button = findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                addNotification();
            }
        });
    }

    private void addNotification() {
        NotificationCompat.Builder builder =
                new NotificationCompat.Builder(this)
                        .setSmallIcon(R.drawable.ic_launcher_background) //set icon for notification
                        .setContentTitle("Notifications Example") //set title of notification
                        .setContentText("This is a notification message")//this is notification message
                        .setAutoCancel(true) // makes auto cancel of notification
                        .setPriority(NotificationCompat.PRIORITY_DEFAULT); //set priority of notification
        Intent notificationIntent = new Intent(this, NotificationView.class);
        notificationIntent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
        //notification message will get at NotificationView
        notificationIntent.putExtra("message", "This is a notification message");

        PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, notificationIntent,
                PendingIntent.FLAG_UPDATE_CURRENT);
        builder.setContentIntent(pendingIntent);

        // Add as notification
        NotificationManager manager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
        manager.notify(0, builder.build());
    }
}

NotificationView.java
package com.example.notification;

import android.os.Bundle;
import android.widget.TextView;

import androidx.appcompat.app.AppCompatActivity;

public class NotificationView extends AppCompatActivity {
    TextView textView;
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.secondact);
        textView = findViewById(R.id.textView);
        //getting the notification message
        String message=getIntent().getStringExtra("message");
        textView.setText(message);
    }

}
AndroidManifest.xml

<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/Theme.Notification"
        tools:targetApi="31">
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <activity android:name=".NotificationView"
            android:parentActivityName=".MainActivity"/>
    </application>

</manifest>

Slip (16)


Q.1 A) Create a Simple Android Application Which Send ―Hello‖ message from one activity to another with help of Button (Use Intent). [10]

B) Create an Android application which will ask the user to input his name and a message, display thetwo items concatenated in a label, and change the format of the label using radio buttons and check boxes for selection, the user can make the label text bold, underlined or italic and change its color . nclude buttons to display the message in the label, clear the text boxes and label and then exit.
Activity_main.xml

<?xml version="1.0" encoding="utf-8"?>
<TableLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TableRow>
        <EditText
            android:id="@+id/name"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:hint="Name" />
    </TableRow>
    <TableRow>
        <EditText
            android:id="@+id/msg"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:hint="Massage" />
    </TableRow>
    <TableRow>
        <TextView
            android:id="@+id/con"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Concatinate display here" />

    </TableRow>
    <TableRow>
        <RadioButton
            android:id="@+id/font"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Font"/>
    </TableRow>
    <TableRow>
        <RadioButton
            android:id="@+id/style"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Style"/>
    </TableRow>
    <TableRow>
        <CheckBox
            android:id="@+id/bold"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Bold"/>
    </TableRow>
    <TableRow>
        <CheckBox
            android:id="@+id/italic"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Italic"/>
    </TableRow>
    <TableRow>
        <CheckBox
            android:id="@+id/underline"

            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Underline"/>
    </TableRow>
    <TableRow>
        <RadioButton
            android:id="@+id/color"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Change Color"/>
    </TableRow>
    <TableRow>
        <Button
            android:id="@+id/display"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Display" />
    </TableRow>
    <TableRow>
        <Button
            android:id="@+id/clear"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Clear" />
    </TableRow>
    <TableRow>
        <Button
            android:id="@+id/exit"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Exit" />
    </TableRow>


</TableLayout>

MainActivity.java
package com.example.textlabelcheckbox;

import androidx.appcompat.app.AppCompatActivity;

import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.Typeface;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.CheckBox;
import android.widget.EditText;
import android.widget.RadioButton;
import android.widget.TextView;

public class MainActivity extends AppCompatActivity {
    EditText name,msg;
    TextView con;
    Button display,clear,exit;
    RadioButton font,style,color;
    CheckBox bold,italic,underline;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        name=(EditText) findViewById(R.id.name);
        msg=(EditText) findViewById(R.id.msg);
        con=(TextView) findViewById(R.id.con);
        display = (Button) findViewById(R.id.display);
        clear = (Button) findViewById(R.id.clear);
        exit = (Button) findViewById(R.id.exit);
        font = (RadioButton) findViewById(R.id.font);
        style = (RadioButton) findViewById(R.id.style);

        color = (RadioButton) findViewById(R.id.color);
        bold = (CheckBox) findViewById(R.id.bold);
        italic = (CheckBox) findViewById(R.id.italic);
        underline = (CheckBox) findViewById(R.id.underline);
        display.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String c=name.getText().toString()+" "+msg.getText().toString();
                con.setText(c);
                con.setTypeface(null, Typeface.BOLD);
                con.setTextSize(20);

            }
        });
        clear.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                name.setText(" ");
                msg.setText(" ");

            }
        });

        italic.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                con.setTypeface(null, Typeface.ITALIC);
            }
        });
        bold.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                con.setTypeface(null, Typeface.BOLD);
            }
        });
        underline.setOnClickListener(new View.OnClickListener() {
            @Override

            public void onClick(View view) {
                con.setPaintFlags(con.getPaintFlags() | Paint.UNDERLINE_TEXT_FLAG);
            }
        });
        exit.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
            }
        });
        color.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                con.setTextColor(Color.CYAN);
            }
        });

    }
}






Slip (12)
Q.1 A) Write an Android program to perform Zoom In, Zoom Out operation and display Satellite view, on Google Map.
B) Create an Android application, where the user can enter player name and points in one view and display it in another view.

