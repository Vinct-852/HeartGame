#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <ctype.h>

#define BUFFERSIZE 300

int inputIdentifier(char c);
int siutCmp(char c1, char c2);
int rankCmp(char c1, char c2);
void arrange(char *arranged, char numberOfCards);
int tolead(char *arranged, char numberOfCards);
int childNum(int child);
int toplay(char *arranged, char numberOfCards, char siut);
int winner(char *deck, char siut);
int discard(char *arranged, char numberOfCards);
int score(char *deck, int winner);

int main(int argc, char *argv[]){
    int inputLen;
    char endStage1[] = "endStage1";
    char cards[BUFFERSIZE];
    int childsID[4];
    char *card;
    int scores[4];

/*-----------------Input-------------------*/
    // getting input redirection from a file
    if(argc == 1){
        inputLen = fread(cards, sizeof(*cards), sizeof(cards)/sizeof(*cards), stdin);
    }
    // Input is received from the keyboard
    else{
        printf("Input is received from the keyboard\n");
    }
/*----------------------------------*/
    
/*-------------- pipe --------------*/
    int p2c[4][2];	// parent to child
    int c2p[4][2];    // child to parent
    char p2cBuf[20], c2pBuf[20];
    //pipe error
    int i;
    for(i=0;i<4;i++){
        if (pipe(&p2c[i][0]) < 0) {
            printf("Pipe creation error\n");
            exit(1);
        }
        if (pipe(&c2p[i][0]) < 0) {
            printf("Pipe creation error\n");
            exit(1);
        }
    }
/*----------------------------------*/

/*-----------child process----------*/
	for(i = 0; i < 4; i++) {
        int childId = fork();
        childsID[i] = childId;

        // error
        if(childId < 0) {   
            printf("*Create child process falied*\n");
            exit(1);
        }    
        // child process
		else if(childId == 0) {

            close(p2c[i][1]);   // close p2c out
            close(c2p[i][0]);   // close c2p read

            char myCards[100];
            char arranged[100];

            // receiving cards
            int n;
            int numberOfCards=0;
            int receivingCards=1;

            while((n = read(p2c[i][0],p2cBuf,20)) > 0){
                if(strcmp(p2cBuf,"done") == 0) break;

                else if(strcmp(p2cBuf,"endStage1") == 0){
                    printf("child %d pid %d: received %s\n",i+1,getpid(),myCards);
                    receivingCards=0;
                    
                    // sorting cards
                    strcpy(arranged, myCards);
                    arrange(arranged, numberOfCards);
                    printf("child %d pid %d: arranged %s\n",i+1,getpid(),arranged);
                    sleep(1);
                }

                else if(receivingCards == 1){
                    p2cBuf[n] = 0;
                    // printf("child %d pid %d cards %s\n",i+1,getpid(),p2cBuf);
                    strcat(myCards,p2cBuf);
                    memset(p2cBuf, 0, 20);  
                    strcat(myCards," ");
                    numberOfCards++;
                }
                
                else if(strcmp(p2cBuf,"tolead") == 0){
                    // printf("child %d pid %d: i am the first player??\n",i+1,getpid());
    
                    int index = tolead(arranged, numberOfCards);
                    // printf("child %d pid %d: index %d \n",i+1,getpid(),index);
                    // printf("index %d \n",index);
                    memset(c2pBuf,0,20);
                    c2pBuf[0] = arranged[index];
                    c2pBuf[1] = arranged[index+1];
                    arranged[index] = ' ';
                    arranged[index+1] = ' ';

                    write(c2p[i][1],c2pBuf,20);
                    printf("child %d pid %d: play %s\n",i+1,getpid(), c2pBuf);
                }

                else{
                    // receive
                    char siut = p2cBuf[0];
                    int index = toplay(arranged, numberOfCards, siut);

                    // select card
                    if(index == -1){
                        index = discard(arranged, numberOfCards);
                        // printf("child %d pid %d: discards\n");
                    }
                    
                    // play cards
                    memset(c2pBuf,0,20);
                    c2pBuf[0] = arranged[index];
                    c2pBuf[1] = arranged[index+1];
                    arranged[index] = ' ';
                    arranged[index+1] = ' ';

                    write(c2p[i][1],c2pBuf,20);

                    printf("child %d pid %d: play %s\n",i+1,getpid(), c2pBuf);
                }
            }
            
			exit(0);
		}
        else{
            close(p2c[i][0]);   // close p2c in
            close(c2p[i][1]);   // close c2p out
        }
	}
/*----------------------------------*/

/*------------parent process---------------*/

    // print out the first line
    printf("Parent pid %d: child players are %d %d %d %d\n", getpid(), childsID[0], childsID[1], childsID[2], childsID[3]);
    // printf("%s\n", cards);
    // printf("\n");

    // cards distribution
    int j=0;
    int counter=0;
    int totalcards=0;
    while(j<inputLen){
        if(inputIdentifier(cards[j]) == 1){
            memset(p2cBuf, 0, 20); 
            p2cBuf[0] = cards[j];
            p2cBuf[1] = cards[j+1];            
            write(p2c[counter][1],p2cBuf,20);
            memset(p2cBuf, 0, 20);  
            // printf("Pass %c%c to %d\n",cards[j],cards[j+1], counter);
            j+=2;
            counter++;
            totalcards++;
            if(counter == 4) counter = 0;
        }
        else j++;
    }
    
    // end the sending cards stage
    strcpy( p2cBuf, endStage1);
    for(i=0; i<4; i++){
        write(p2c[i][1],p2cBuf,20);
    }
    memset(p2cBuf, 0, 20);  
    sleep(1);

    // start the game
    counter = 0;
    int counter2;
    int win=1;
    int child=win-1;
    char deck[8];
    char siut;
    
    for(i=0; i<4; i++){
        scores[i]=0;
    }
    printf("\nscore: %d %d %d %d\n", scores[0], scores[1], scores[2], scores[3]);

    while(counter < totalcards/4){ 
        child = win-1;
        printf("Parent pid %d: round %d child %d to lead\n", getpid(), counter+1, win);
        
        // send tolead to winer
        int n;
        memset(p2cBuf, 0, 20); 
        strcpy( p2cBuf, "tolead");
        write(p2c[child][1],p2cBuf,20);

        memset(c2pBuf, 0, 20);
        memset(deck, 0, 8);

        if((n = read(c2p[child][0],c2pBuf,20)) > 0){
            // printf("Parent pid %d: received %c\n", getpid(), c2pBuf[0]);
            // printf("Parent pid %d: received %c\n", getpid(), c2pBuf[1]);
            deck[child+child] = c2pBuf[0];
            deck[child+child+1] = c2pBuf[1];
            siut = deck[child+child];
            printf("Parent pid %d: child %d plays %c%c\n", getpid(), child+1, deck[child+child],deck[child+child+1]);
        }

        // send siut and deck to child
        // and receive their cards
        child = childNum(child);
        counter2 = 0;
        while(counter2 < 3){
            // send
            memset(p2cBuf,0,20);
            p2cBuf[0] = siut;
            write(p2c[child][1],p2cBuf,20);
            sleep(0.5);

            // receive
            if((n = read(c2p[child][0],c2pBuf,20)) > 0){
                deck[child+child] = c2pBuf[0];
                deck[child+child+1] = c2pBuf[1];
                memset(c2pBuf, 0, 20);
                printf("Parent pid %d: child %d plays %c%c\n", getpid(), child+1, deck[child+child],deck[child+child+1]);
            }

            child = childNum(child);
            counter2++;
        }
        
        // calculate who win
        win = winner(deck,siut);
        printf("Parent pid %d: child %d wins the trick\n", getpid(), win);
        scores[win-1] += score(deck,win);
        // printf("\nscore1: %d\n",score(deck,win));
        // printf("score: %d %d %d %d\n", scores[0], scores[1], scores[2], scores[3]);

        counter++;
    }

    // terminate all childs
    strcpy( p2cBuf, "done");
    for(i=0; i<4; i++){
        write(p2c[i][1],p2cBuf,20);
    }
    memset(p2cBuf, 0, 20);

    //wait for the child process
    for (i = 0; i < 4; i++) { 
        // close(p2c[i][1]);
        wait(NULL);
    }

    printf("Parent pid %d: game completed\n", getpid());

    // score handling
    int bigWin = 0;
    int wins;
    for(i=0; i<4; i++){
        if(scores[i] == 26){
            bigWin = 1;
            wins = i;
        }
    }
    if(bigWin == 1){
        for(i=0; i<4; i++){
            if(i == wins) scores[i] = 0;
            else scores[i] = 26;
        }
    }

    printf("Parent pid %d: score = <%d %d %d %d>\n", getpid(), scores[0], scores[1], scores[2], scores[3]);

    exit(0);
}

/*----------------------suit compare------------------------*/
int siutCmp(char c1, char c2){
    // if c1 is bigger than c2 return 1 else 0;
    if(c1 > c2){
        if(c1 != 'D'){
            return 1;
        }
        else return 0;  // D > C
    }
    else{
        if(c2 != 'D'){
            return 0;
        }
        else return 1;  // C < D
    }
}

/*---------------------rank compare-----------------------------*/
int rankCmp(char c1, char c2){
    // c1: return 1, c2: return 0
    // when c1 is number 
    if(c1 > c2 && isdigit(c1)){
        return 1;
    }
    // when c1 is not number 
    else if(c1 > c2 && isdigit(c2)){
        return 1;
    }
    // when both are not number
    if(isalpha(c1) && isalpha(c2)){
        if(c1 > c2){
            if(c2 == 'A'){
                return 0;
            }
            else if(c1 == 'T'){
                return 0;
            }
            else if(c1 == 'Q' && c2 == 'K'){
                return 0;
            }
            else return 1;
        }
        else{
            if(c1 == 'A'){
                return 1;
            }
            else if(c2 == 'T'){
                return 1;
            }
            else if(c2 == 'Q' && c1 == 'K'){
                return 1;
            }
            else return 0;
        }
    }
}

/*--------------Check the input is valid or not-----------------*/
int inputIdentifier(char c){
    if(isdigit(c) || c == 'A' || c == 'K' || c == 'Q'|| c == 'J'|| c == 'T'|| c == 'S'|| c == 'H'|| c == 'C'|| c == 'D'){
        return 1;
    }
    else return 0;
}

void arrange(char *arranged, char numberOfCards){
    int j, k=0;
    char siut, rank;

    while(k < numberOfCards*3){
        siut = arranged[k];
        rank = arranged[k+1];
        j = k - 3;

        while (j >= 0 && ((arranged[j]!=siut && siutCmp(arranged[j],siut)==0) || (arranged[j]==siut && rankCmp(arranged[j+1],rank)== 0))){
            arranged[j+3] = arranged[j];
            arranged[j+4] = arranged[j+1];
            j = j - 3;
        }
        arranged[j+3] = siut;
        arranged[j+4] = rank;
        k+=3;
    }
}

int tolead(char *arranged, char numberOfCards){
    // getting the smallest rank
    int i=1;
    char rank = 'A';
    while(i<numberOfCards*3){
        if(rankCmp(arranged[i],rank)==0 && arranged[i] != ' '){
            rank = arranged[i];
        }
        i+=3;
    }
    // getting the smallest siut array[rank-1]
    i=0;
    char siut = 'S';
    int res = -1;
    while(i<numberOfCards*3){
        if(arranged[i+1] == rank && siutCmp(arranged[i],siut) == 0){
            siut = arranged[i];
            res = i;
        }
        i+=3;
    }

    return res;
}

int toplay(char *arranged, char numberOfCards, char siut){
    int i=0;
    char rank = 'A';
    int res = -1;
    while(i<numberOfCards*3){
        if(arranged[i] == siut && ( arranged[i+1]==rank || (arranged[i+1]!=rank && rankCmp(arranged[i+1],rank) == 0))){
            rank = arranged[i+1];
            res = i;
        }
        i+=3;
    }
    return res;
}

int childNum(int child){
    if(child == 3) return 0;
    else return child+1;
}

int winner(char *deck, char siut){
    int winner = -1;
    int i = 0;
    char rank = '2';
    while(i<7){
        if(deck[i]==siut && (deck[i+1]!=rank && rankCmp(deck[i+1],rank)==1)){
            rank = deck[i+1];
            winner = i;
        }
        i+=2;
    }
    return winner/2+1;
}

int discard(char *arranged, char numberOfCards){
    int res;
    char siut = 'd';
    char rank = '2';
    int haveH = 0;
    int i=0;
    
    while(i<numberOfCards*3){
        // SQ will be chosen for discard
        if(arranged[i] == 'S' && arranged[i+1] == 'Q'){
            return i;
        }
        // highest H card will be discarded
        if(arranged[i] == 'H'){
            haveH = 1;
        }
        // No H card -> the highest card (rank) > (siut)
        if((arranged[i]!=siut && rankCmp(arranged[i+1],rank)==1) || (arranged[i]==siut && rankCmp(arranged[i+1],rank) == 1)){
            siut = arranged[i];
            rank = arranged[i+1];
            res = i;
        }
        i+=3;
    }
    
    if(haveH == 0){
        return res;
    }
    else{
        siut = 'D';
        rank = '2';
        i=0;
        while(i<numberOfCards*3){
            if(arranged[i] == 'H' && (arranged[i+1] == rank || (arranged[i+1]!=rank && rankCmp(arranged[i+1],rank)==1))){
                siut = arranged[i];
                rank = arranged[i+1];
                res = i;
            }
            i+=3;
        }
        return res;
    }

    return -1;
}

int score(char *deck, int winner){
    int i=0;
    int res=0;
    int win=winner-1;
    while(i<7){
        //winning a SQ -> 13 pt
        if(deck[i]=='S' && deck[i+1]=='Q' && i != win){
            res+=13;
        }
        //winning a card of H -> 1 pt
        if(deck[i]=='H' ){
            res+=1;
        }
        i+=2;
    }
    
    return res;
}
