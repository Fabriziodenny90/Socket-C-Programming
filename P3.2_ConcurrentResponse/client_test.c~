#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <ctype.h>
#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <rpc/xdr.h>
#include <string.h>
#include <time.h>

#define SOCKET int
#define INVALID_SOCKET -1
#define BUFLEN 128
/* FUNCTION PROTOTYPES s*/
int mygetline(char * line, size_t maxline, char *prompt);
int writen(SOCKET, char *, size_t);
int readline (SOCKET s, char *ptr, size_t maxlen);

main(int argc, char **argv)	
{
	char     	   buf[BUFLEN];		/* transmission buffer */
	char	 	   rbuf[BUFLEN];	/* reception buffer */
	uint32_t	   taddr_n;		/* server IP addr. (network byte order) */
	uint16_t	   tport_n, tport_h;	/* server port number (net/host byte order resp.) */

	SOCKET		   s;
	int		   result;
	struct sockaddr_in saddr;		/* server address structure */
	struct in_addr	   sIPaddr; 		/* server IP addr. structure */
	fd_set master;
        
        if (argc != 3) {
	    printf("Error with input parameters\n");
            exit(1);
        }

        result = inet_aton(argv[1], &sIPaddr);
        if (!result)
	    printf("Invalid address");
        
	tport_h = atoi(argv[2]);
  	tport_n = htons(tport_h);

	/* create the socket */
    	printf("Creating socket\n");
	s = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
	if (s == INVALID_SOCKET)
		printf("socket() failed");
	printf("done. Socket fd number: %d\n",s);

	/* prepare address structure */
    	saddr.sin_family = AF_INET;
	saddr.sin_port   = tport_n;
	saddr.sin_addr   = sIPaddr;

	/* connect */
	printf("Connecting to target address... ");
	result = connect(s, (struct sockaddr *) &saddr, sizeof(saddr)); //dangerous casting 
	if (result == -1){
		printf("Connect() failed!\n");
		exit(1);
	}
	printf("Done.\n");

	/* main client loop */
	printf("Enter line 'QUIT' to close connection and stop client.\n");
	int len;
	int receiving = FALSE;
	int fsize = 0, fr_block_sz = 0, written = 0;
	int exitonfinish = 0;
	FILE *fr;
	while (1)
	{
		FD_ZERO(&master);
		int st0 = 0;
		FD_SET(s, &master);
		FD_SET(st0, &master);
		select(s+1, &master, NULL, NULL, NULL);
		
		if (FD_ISSET(st0, &master))
		{
			fgets(buf, BUFLEN, stdin);
			char buf2[BUFLEN];
			strcpy(buf2,buf);
			int writey = 1;
			//printf("%s", buf);
			//strcat(buf,"\n");
			len = strlen(buf);
			
			if(len==5){
				buf[4]='\0';
				if(!strcmp(buf,"QUIT")){
					printf("Closing\n");
					close(s);
					exit(0);
				}
			}
			
			if(len==2){
				buf[1]='\0';
				if(!strcmp(buf,"A")){
					printf("Abort...\n");
					close(s);
					exit(0);
				}
				if(!strcmp(buf,"Q")){
					exitonfinish = 1;
					writey = 0;
					printf("Will exit after transer...\n");
					fflush(stdout);
				}
			}
			if(writey){
				if(writen(s, buf2, len) != len)
				{
					printf("Write error\n");
					break;
				}
			}
		}
		
		if (FD_ISSET(s, &master)){
			if(!receiving){
				printf("SENT. Waiting for response...\n");
				result = readline(s, rbuf, BUFLEN);
				if (result <= 0)
				{
					 printf("Read error/Connection closed\n");
					 close(s);
					 exit(1);
				}
				else
				{
					rbuf[result-1] = '\0';
					printf("Received response from socket %03u : %s\n", s, rbuf);		
					rbuf[3]='\0';
					if(!strcmp(rbuf,"+OK")){
						printf("[Client] Receiveing file from Server and saving it as final...\n");
						readline(s, rbuf, BUFLEN);
						sscanf (rbuf,"%d",&fsize);
						printf("Size of incoming file: %d\n", fsize);
						fr_block_sz = 0; 
						written, exitonfinish = 0;
						char fr_name[BUFLEN] = "/home/parallels/Desktop/final";
						char tmppid[BUFLEN];
						int a = (int)getppid();
						//printf("Ecco il PID %d\n", a);
						sprintf(tmppid, "%d", (int)getppid());
						strcat(fr_name,tmppid);
	
						fr = fopen(fr_name, "a");
						if(fr == NULL){
							printf("File %s Cannot be opened.\n", fr_name);
							close(s);
							exit(1);
							}
						receiving = TRUE;
					}
				}
			}
			else
			{
				bzero(rbuf, BUFLEN); 
				if((fr_block_sz = recv(s, rbuf, BUFLEN, 0)) >= 0){
					int write_sz = fwrite(rbuf, sizeof(char), fr_block_sz, fr);
					if(write_sz < fr_block_sz)
					{
			   			printf("File write failed.\n");
					}
					bzero(rbuf, BUFLEN);
					if (fr_block_sz == 0) 
					{
						printf("Connection got interrupted!\n");
						receiving = FALSE;
					}
					written = written + write_sz;
					//printf("Written so far: %d over %d\n", written, fsize);
	
					fsize = fsize - write_sz;
					if(fsize==0) 
					{
						printf("Ok received from server!\n");
						printf("Bytes received: %d\n", written);
						if(exitonfinish){
							printf("Q was entered. Exiting...");
							fclose(fr);
							close(s);
							break;
						}
						receiving=FALSE;
						fclose(fr);
					}
				}
			}
		}
	}
        printf("Closing\n");
	close(s);
	
	return (0);
}

/* Gets a line of text from standard input after having printed a prompt string 
   Substitutes end of line with '\0'
   Empties standard input buffer but stores at most maxline-1 characters in the
   passed buffer
*/
int mygetline(char *line, size_t maxline, char *prompt)
{
	char	ch;
	size_t 	i;

	printf("%s", prompt);
	for (i=0; i< maxline-1 && (ch = getchar()) != '\n' && ch != EOF; i++)
		*line++ = ch;
	*line = '\0';
	while (ch != '\n' && ch != EOF)
		ch = getchar();
	if (ch == EOF)
		return(EOF);
	else    return(1);
}

/* Reads a line from stream socket s to buffer ptr 
   The line is stored in ptr including the final '\n'
   At most maxlen chasracters are read
*/
int readline (SOCKET s, char *ptr, size_t maxlen)
{
    size_t n;
    ssize_t nread;
    char c;

    for (n=1; n<maxlen; n++)
    {
	nread=recv(s, &c, 1, 0);
	if (nread == 1)
	{
	    *ptr++ = c;
	    if (c == '\n') 
		break;
	}
	else if (nread == 0)	/* connection closed by party */
	{
	    *ptr = 0;
	    return (n-1);
	}
	else 			/* error */
	    return (-1);
    }
    *ptr = 0;
    return (n);
}


/* Writes nbytes from buffer ptr to stream socket s */
int writen(SOCKET s, char *ptr, size_t nbytes)
{
    size_t  nleft;
    ssize_t nwritten;

    for (nleft=nbytes; nleft > 0; )
    {
	nwritten = send(s, ptr, nleft, 0);
	if (nwritten <=0)
	    return (nwritten);
	else
	{
	    nleft -= nwritten;
	    ptr += nwritten;   
	}
    }
    return (nbytes - nleft);
}
