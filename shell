#include <stdio.h>
#include <ctype.h>
#include <limits.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <sys/stat.h>
#include <limits.h>
#include <errno.h>
#include <fcntl.h>
#define num 10

enum { qoute,amp,flerror,space,more,less,doublemore,redirect,pipeline};

struct listword	
{
    char *s;
    struct listword *next;    
};

struct database
{
    int len;
    int base;
    char c;
    char early;
    char *str;
    int desin;
    int desout;
};

void initial(struct database *data)
{
    data->len=0;
    data->base=num;
}

void flags(char c,char early, int *flag)
{
        if (c=='"')
			flag[qoute]=!flag[qoute];
		if ((c!='"')&&(c!='&')&&(c!='<')&&(c!='>')&&(c!='|'))
			flag[space]=1;
		if (flag[amp]==1)
			flag[flerror]=1;
		if (c=='&') 
		{
			if (flag[amp]==0)
				flag[amp]=1;
			else
				flag[flerror]=1;
		}
		if (c=='|') 
		{
			if (flag[pipeline]==0)
				flag[pipeline]=1;
			else
				flag[flerror]=1;
		}
		
		if (c=='<') 
		{
			if (flag[less]==0)
				flag[less]=1;
			else
				flag[flerror]=1;
		}
	   if (c=='>') {
		   if(early=='>'){
			   if (flag[doublemore]==0)
				flag[doublemore]=1;
			else
				flag[flerror]=1;
			}
			else{
				if (flag[more]==0)
				    flag[more]=1;
		        else
				    flag[flerror]=1;
	       }
       }
       if (flag[more]||(flag[more]&&flag[doublemore])||flag[less]) 
           flag[redirect]=1;
}

void errorchange(int *flag)
{
    if ((flag[qoute]==1)||(flag[space]==0)||(flag[redirect]==1))
        flag[flerror]=1;
}

void zero(int *flag)
{
    int i;
    for (i=0;i<(pipeline+1);i++)
        flag[i]=0;
}

int word_len(struct listword *str)
{
    int len=0;
    while (str!=NULL)
    {
        len++;
        str=str->next;
    }
    return len;
}
int cd(char **args)
{
  if (args[1] == NULL) {
    chdir(getenv("HOME"));
  } else {
    if (chdir(args[1]) != 0) {
      perror("");
    }
  }
 return 1;
}

int exitt(char **args)
{
  return 0;
}

char *builtin_str[] = {
  "cd",
  "exit"
};

int (*builtin_func[]) (char **) = {
  &cd,
  &exitt
};

int num_builtins() {
  return sizeof(builtin_str) / sizeof(char *);
};

char *new_sym(char *str,struct database *data)
{
    if (((data->c)!='"')&&((data->c)!='&'))
    {
        if (data->len<data->base)
            str[data->len]=data->c;
        else
        {
            str=realloc(str,data->base*2*sizeof(char));
            data->base*=2;
            str[data->len]=data->c;
        }
        (data->len)++;
    }
    return str;
}
void rwfile(struct database *data,int *flag)
{
	if((flag[more])&&(data->desin==0))
	{
		if(flag[doublemore])
		   data->desin=open(data->str,O_WRONLY|O_APPEND|O_CREAT,0666);
		else
		   data->desin=open(data->str,O_WRONLY|O_APPEND|O_TRUNC|O_CREAT,0666);
    }
    if((flag[less])&&(data->desout==0))
		data->desout=open(data->str,O_RDONLY);
	if ((data->desin==-1)||(data->desout==-1))
	{
		perror(data->str);
		flag[flerror]=1;
		exit(1);
	}
	flag[redirect]=0;
}
int launch(char **args1,char **args2,int *flag,struct database *data)
{   
	int pid,pid2,status;
	int fd[2];
	if (flag[pipeline]) pipe(fd);
	pid=fork();
    if (pid==0){
		if(flag[less]){
			dup2(data->desout,0);
			close(data->desout);
		}
		if(flag[more]||flag[doublemore]){
			dup2(data->desin,1);
			close(data->desin);
		}
		if (flag[pipeline]){
			    close(fd[0]);
				dup2(fd[1],1);
				close(fd[1]);
		}
		execvp(args1[0], args1);
		perror(args1[0]);
        exit(1);
    }
    if (pid==-1){
			   perror("fork");
            exit(1);
        }
   if (flag[pipeline]){
	   pid2=fork();
	   if (pid2==0){
		        close(fd[1]);
			    dup2(fd[0],0);
				close(fd[0]);
				execvp(args2[0], args2);
				perror(args2[0]);
				exit(1);
		}
		if (pid2==-1){
			perror("fork");
            exit(1);
        }	
    }
    close(fd[0]);
	close(fd[1]);
 if (flag[pipeline]){
	 waitpid(pid, NULL, 0);
	 waitpid(pid2, NULL, 0);
 }
    if (flag[amp]==0){
        waitpid(pid, &status, 0);
        while((pid=waitpid(-1, &status, WNOHANG))>0)
			if(WIFEXITED(status) )
			 printf("\n[%d] %d \n",WEXITSTATUS(status),pid);
      }  
      
  return 1;
}

int execute(char **args1,char **args2,int *flag,struct database *data)
{
  int i;

  if (args1[0] == NULL) {// Была введена пустая команда.
    return 1;
  }

  for (i = 0; i < num_builtins(); i++) {
    if (strcmp(args1[0], builtin_str[i]) == 0) {
      return (*builtin_func[i])(args1);
    }
  }
  return launch(args1,args2,flag,data);
}

char *str_finale(char *str,struct database *data)
{
    char *str_new;
    str_new=malloc(data->len);
    int i;
    for (i=0;i<data->len;i++)
        str_new[i]=str[i];
    str_new[data->len]=0;
    free(str);
    data->len=0;
    data->base=num;   
    return str_new;
}
void listmk(struct database *data,struct listword**p,int *flag)
{   
	struct listword *q,*t;
	if((data->c=='>')||(data->c=='<')){
    data->str=str_finale(data->str,data);
    q=*p;
    if (q!=NULL)
        while ((q->next)!=NULL)
            q=q->next;
    t=malloc(sizeof(**p));
    t->s=data->str;
    t->next=NULL;
    if ((*p)==NULL)
        *p=t;
    else
        q->next=t;
	} else{
	   data->str=str_finale(data->str,data);
	   if(flag[redirect])
		   rwfile(data,flag);
	   else {
		q=*p;
		if (q!=NULL)
			while ((q->next)!=NULL)
             q=q->next;
		t=malloc(sizeof(**p));
		t->s=data->str;
		t->next=NULL;
		if ((*p)==NULL)
			*p=t;
		else
		q->next=t;
       }
   }
}

char **cmdcreat(struct listword *p,int *len)
{
    struct listword *tmp;
    char **cmd;
    tmp=p;
    *len=word_len(p);
    cmd=malloc((*len+1)*sizeof(char*));
    int i=0;
    for (i=0;i<(*len);i++){
        cmd[i]=tmp->s;
        tmp=tmp->next;
    }
    cmd[i]=0;
    return cmd;
}

void killist(struct listword **st,char **args)
{
    struct listword *str;
    while ((*st)!=NULL)
    {
        free((*st)->s);
        str=(*st)->next;
        free(*st);
        *st=str;
    }
    free(args);
    *st=NULL;
}

void invite(){// приглашение на ввод
	char *buf=malloc(PATH_MAX);
	int leng=PATH_MAX;
	while(!getcwd(buf,leng)){
		free(buf);
		leng*=2;
	}
    printf ("%s> ",buf);
    buf=NULL;
}

void shell()
{
  struct database *data=malloc(sizeof(*data));
  struct listword *one=NULL;
  char **args1;
  char **args2=NULL;
  int stat=1;
  int status,pid=0;
  int flag[9]={0,0,0,0,0,0,0,0,0};
  data->str= malloc(sizeof(char) *num);
  initial(data);
  data->early=' ';
  data->desin=0;
  data->desout=0;
  invite();
  while (((data->c)=getchar())!=EOF) { 
	while( (pid=wait4(-1,&status,WNOHANG,NULL))>0){
		if(WIFEXITED(status) ) 
			printf("[%d] %d \n",WEXITSTATUS(status),pid);
	}
	flags(data->c,data->early,flag);
    if (((data->c)!='\n')&&((data->c)!='|')){
       if ((((data->c)!=' ')&&((data->c)!='<')&&((data->c)!='>'))||(flag[qoute])){
                data->str=new_sym(data->str,data);
                data->early=data->c;  
			}
            else {   
                if (data->len!=0)
					listmk(data,&one,flag);
                else
                    free(data->str);
                data->str= malloc(sizeof(char) *num);
                initial(data);
			}     
        } else {
            if (data->len!=0)
                listmk(data,&one,flag);
            else free(data->str);
            if ((flag[qoute]==1)||(flag[space]==0))
                flag[flerror]=1;
            if (((data->c)=='\n')&&(flag[pipeline]==1)) args2=cmdcreat(one,&(data->len));
            if (data->c=='|') {
					args1=cmdcreat(one,&(data->len));
					one=NULL;
			}
            if(!flag[pipeline]) args1=cmdcreat(one,&(data->len));
            if ((flag[flerror]==0)&&((data->c)=='\n')) {stat=execute(args1,args2,flag,data);}
			// else if(data->len!=0) printf("Input errors\n");
	       if((data->c)=='\n') {
			   killist(&one,args1);
	           killist(&one,args2);
	           data->early=' ';
               data->desin=0;
               data->desout=0;
               zero(flag);
               if (!stat) break;  
               invite();
		   }
            data->str= malloc(sizeof(char) *num);
            initial(data);
            
   } 
 }
 free(data->str);
}

int main()
{
  shell();
  printf ("\n");
  return 0;
}
