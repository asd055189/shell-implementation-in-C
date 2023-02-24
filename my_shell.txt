#ifndef _POSIX_C_SOURCE
#define  _POSIX_C_SOURCE 200809L
#endif

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <fcntl.h>
#include <sys/stat.h>
#define READ_END 0
#define WRITE_END 1
#define true 1
#define false 0
void split(char **arr, char *str, const char *del) {
  char *s = strtok(str, del);
 
  while(s != NULL) {
    *arr++ = s;
    s = strtok(NULL, del);
  }
}
void lsh_loop(){
    char history[16][50][512]={'\0'};
    int his_size=0;
    int his_head=0;
    int his_tail=0;

    while(1){
        char *cmd_input[50]={NULL};
        printf(">>> $ ");
        char input_text[512];
        fgets(input_text,512,stdin);
        if (input_text[strlen(input_text) - 1] == '\n')
            input_text[strlen(input_text) - 1] = '\0';
        if(input_text[0]=='\0' || input_text[0]==' '|| input_text[0]=='\t' )
            continue;
        split(cmd_input,input_text," ");

        if (his_size>=16){
            his_head++;
            if(his_head==16)
                his_head=0;
        }
        int index=0;
        while(true){
            if(cmd_input[index]){
                char empty[512]="\0";
                strcpy(history[his_tail][index],empty);
                strcpy(history[his_tail][index],cmd_input[index]);
            }
            else
                break;
            index++;
        }
        his_tail++;
        his_size++;
        if(his_tail==17){
            his_tail=0;
        }
        char *read_file=NULL;
        char *write_file=NULL;
        for(int i=0;i<50;i++){
            if(cmd_input[i]){
                if(strcmp(cmd_input[i],">")==0 ||strcmp(cmd_input[i],"<")==0){
                    if(strcmp(cmd_input[i],">")==0){
                        write_file=malloc(50);
                        strcpy(write_file,cmd_input[i+1]);
                    } 
                    else if(strcmp(cmd_input[i],"<")==0){
                        read_file=malloc(50);
                        strcpy(read_file,cmd_input[i+1]);
                    }
                    cmd_input[i]=NULL;
                    cmd_input[i+1]=NULL;

                }
            }
        }
        int *write_fd=NULL;
        if(write_file){
            write_fd=(int*)malloc(2*sizeof(int));
            pipe(write_fd);
        }
        int *read_fd=NULL;
        if(read_file){
            //printf("%s\n",read_file);
            read_fd=(int*)malloc(2*sizeof(int));
            pipe(read_fd);
            int pid=fork();
            if(pid==0){
                dup2(read_fd[1],1);
                close(read_fd[0]);
                close(read_fd[1]);
                if(-1==execlp("cat","cat",read_file,NULL)){
                    perror("execlp() failed");
                }
            }
        }
        if(strcmp(cmd_input[0],"help")==0){
			printf("-------------------------------------------------\n");
            printf("The following are built in:\n");
            printf("1: help:\tshow all build-in function info\n");
            printf("2: cd:      \tchange directory\n");
            printf("3: echo:\techo the strings to standard output\n");
            printf("4: record:\tshow last-16 cmds you tuped in\n");
            printf("5: replay:\tre-execute the cmd showed in record\n");
            printf("6: mypid:\tfind and print process-ids\n");
            printf("7: exit:\texit shell\n");
            printf("\nUse the \"man\" command for info. on other programs.\n");
			printf("-------------------------------------------------\n");
        }
        else if(strcmp(cmd_input[0],"cd")==0){
			if(chdir(cmd_input[1])!=0){
                char *buf=malloc(512);
                stpcpy(buf, "./my_shell: cd: ");
                strcpy(buf, cmd_input[1]);
                perror(buf);
            }
        }
        else if(strcmp(cmd_input[0],"echo")==0){
            int count=1;
            int println=1;
            int space=0;
            FILE *fptr;

            if(write_file)
                fptr = fopen(write_file,"w");
			while(true){
                if(!cmd_input[count]){
                    if(println==1){
                        if(write_file)
                            fprintf(fptr,"\n");
                        else
                            printf("\n");
                    }
                    break;
                }
                if(println==1&&strcmp(cmd_input[count],"-n")==0){
                    println=0;
                    count++;
                    continue;
                }
                else{
                    if(space==1){
                        if(write_file)
                            fprintf(fptr,"\n");
                        else
                            printf(" ");
                    }
                    if(write_file)
                        fprintf(fptr,"%s",cmd_input[count]);
                    else
                        printf("%s",cmd_input[count]);
                    space=1;
                }
                count++;
            }
            if(write_file){
                fclose(fptr);
                write_file=NULL;
            }
        }
        else if(strcmp(cmd_input[0],"record")==0){
            int head=his_head;
            int tail=his_tail;
            int count=1;
            FILE *fptr;
            
            if(write_file)
                fptr = fopen(write_file,"w");
			while(true){
                int index=0;
                if(write_file)
                    fprintf(fptr,"%2d: ",count);
                else
                    printf("%2d: ",count);
                while(true){
                    if(history[head][index][0]!='\0')
                        if(write_file)
                            fprintf(fptr,"%s ",history[head][index]);
                        else
                            printf("%s ",history[head][index]);
                    else
                        break;
                    index++;
                }
                if(write_file)
                    fprintf(fptr,"\n");
                else
                    printf("\n");

                head++;
                count++;
                if(head==17)
                    head=0;
                if(head==tail)
                    break;
            }
            if(write_file){
                fclose(fptr);
                write_file=NULL;
            }
        }
        /*else if(strcmp(cmd_input[0],"replay")==0){
            int aa=0;
        }
        else if(strcmp(cmd_input[0],"mypid")==0){
            if(strcmp(cmd_input[1],"-i")==0){
               
               fork();
                if (-1 ==execvp(cmd_input[0],cmd_input)){
                    perror("execlp() failed");
                    return;
                }
                
            }
           
        }*/
        else if(strcmp(cmd_input[0],"exit")==0){
            printf("my_shell: See you next time.\n");
            return;
        }
        else{
            char *split_cmd[50][50];
            for (int i=0;i<50;i++)
                for(int j=0;j<50;j++)
                    split_cmd[i][j]=malloc(512);

            int pipe_num=0;
            for(int i=0,j=0,k=0;i<50;i++){
                if(cmd_input[i]){
                    if(strcmp(cmd_input[i],">")==0 ){
                        i++;
                        write_file=malloc(50);
                        strcpy(write_file,cmd_input[i]);
                    }
                    else if(strcmp(cmd_input[i],"<")==0){
                        i++;
                        read_file=malloc(50);
                        strcpy(read_file,cmd_input[i]);
                    }
                    else if (strcmp(cmd_input[i],"|")!=0 ){
                        strcpy(split_cmd[j][k++],cmd_input[i]);
                    }
                    else{
                        k=0;
                        j++;
                        pipe_num++;
                    }
                }      
            }
            char *argv[50][50]={NULL};
            for(int i=0;i<50;i++){
                    for(int j=0;j<50;j++){
                        if(split_cmd[i][j][0]!='\0'){
                            argv[i][j]=malloc(512);
                            strcpy(argv[i][j],split_cmd[i][j]);
                        }
                    }
            }
            int pid=-1;
            int **P=(int**)malloc(pipe_num*sizeof(int*));
            for(int i=0;i<pipe_num;i++){
                P[i]=(int*)malloc(2*sizeof(int));
            }
            for(int i=0;i<=pipe_num;i++){
                if(pipe_num>0 && i!=pipe_num){
                    pipe(P[i]);
                }
                switch(pid=fork()){
                    case 0:
                        if(i==0 && pipe_num>0){
                            dup2(P[i][1],1);
                            close(P[i][0]);
                            close(P[i][1]);
                            if(read_file){
                                dup2(read_fd[0],0);
                                close(read_fd[0]);
                                close(read_fd[1]);
                            }
                        }
                        else if(i==pipe_num && pipe_num>0){
                            dup2(P[i-1][0],0);
                            close(P[i-1][0]);
                            close(P[i-1][1]);
                            if(pipe_num>1){
                                close(P[i-2][0]);
                                close(P[i-2][1]);
                            }
                            if(write_file){
                                dup2(write_fd[1],1);
                                close(write_fd[0]);
                                close(write_fd[1]);
                            }
                        }
                        else if(pipe_num>0){
                            dup2(P[i-1][0],0);
                            dup2(P[i][1],1);
                            close(P[i-1][0]);
                            close(P[i-1][1]);
                            close(P[i][0]);
                            close(P[i][1]);
                        }
                        if(pipe_num==0 && i==0 && write_file){ // if the first cmd need to write file
                            dup2(write_fd[1],1);
                            close(write_fd[0]);
                            close(write_fd[1]);
                        }
                        if(pipe_num==0 && i==0 && read_file){ // if the first cmd need to read file
                            dup2(read_fd[0],0);
                            close(read_fd[0]);
                            close(read_fd[1]);
                        }
                        if(-1==execvp(argv[i][0],argv[i])){
                            perror("execlp() failed");
                        }
                        return ;

                    case -1:
                        printf("error to create child\n");
                        break;
                    default:
                        waitpid(pid,NULL,WNOHANG);
                        break;
                }
            }
            for (int i=0;i<pipe_num;i++){//close the pipes in parent process
                close(P[i][0]);
                close(P[i][1]);
            }
            usleep(1000*50);//set a little bit delay to keep the output consistency

            if(write_file){
                
                char buffer[512];
                read(write_fd[0], buffer, 512);
                close(write_fd[0]);
                close(write_fd[1]);
                FILE *fptr;
                fptr = fopen(write_file,"w");
                fprintf(fptr,"%s",buffer);
                fclose(fptr);
            }
        }
    }
}
int main(){
	
	// Load config files, if any.

  	// Run command loop.
  	lsh_loop();

  
  	// Perform any shutdown/cleanup.

    return EXIT_SUCCESS;
}