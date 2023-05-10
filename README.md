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
