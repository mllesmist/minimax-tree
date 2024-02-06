#include <stdio.h>
#include<stdlib.h>
#include<string.h>
#include <limits.h>


char vide = '.';
int size = 9 ; 


/*structure du noeud*/

typedef struct node 
{
   char qui_joue; 
   char * state; 
   int move_to_state;
   int nb_fils; 
   struct node  *children [9]; 
}node; 

/*implemùentation de la fonction min et max qui seront utiliser pour la comparaison des valeurs*/
int min (int a , int b) {return (a < b)? a :b;}
int max (int a , int b ){ return (a > b)? a :b; }

/*cette fonction check si le déplacement vers une case i est possible ou pas*/
int isValidMove(char * board , int pos) 
{
     if (pos >= 0 && pos < size && board[pos] == vide) return 1 ; 
        return 0; 
}

/* renvoie un noeud contenant les données du prochain mouvement possible*/
node* next_move_child (node * parent, int pos)
{
   
    node * child = malloc(sizeof(node));
    child->qui_joue = (parent-> qui_joue == 'X') ? 'O' : 'X';
    child->move_to_state = pos;
    child ->state = malloc(sizeof (char)*9);
    strcpy(child ->state , parent->state );
    child->state[pos] = child->qui_joue;
    child->nb_fils = 0;
    return child;
}

/*  calcul tout les mouvements possible à partir de la racine (de la matrice actuelle) */
void generateChildren( node * parent) {
   
    for (int i = 0; i < 9; i++)
    {
        
            if (isValidMove(parent->state, i))
            {   
                node * child = next_move_child(parent, i);
                parent->children[i] = child;
                parent->nb_fils+= 1;
                
            }
        
    }

}

/*creation de la racine*/
node * createNode(char * board, char player)
{    
    node * n= malloc(sizeof(node ));
    n->qui_joue = player;
    n->state = malloc(sizeof (char)*9);
    strcpy(n->state, board);
    n-> nb_fils = 0;
    return n;
}

/*affichage du jeu*/
void show_board (char * b)
{
    printf("____________\n");
    for (int i =0 ; i< 9 ; ++i )
    {
        if (b[i] == 'X') {
            printf("| \033[1;34m%c\033[0m ",b[i]);
        } else if (b[i] == 'O') {
            printf("| \033[0;31m%c\033[0m ",b[i]);
        } else {
            printf("| %c ",b[i]);
        }
        if (i == 2 || i ==5 || i ==8)  { printf ("|\n"); printf("____________\n"); }
    }
}
/*evaluation de possiblité win = 10  lose =-10  ou equal = 0*/
int eval_score (char *tab)
{
    if   (tab[0]==tab[1] && tab[0]==tab[2] && tab[0] != vide) return tab [0] == 'X' ? 10 : -10;
 
    if   (tab[3]==tab[4] && tab[3]==tab[5] && tab[3] != vide)  return tab [3] == 'X' ? 10 : -10;
 
    if   (tab[6]==tab[7] && tab[6]==tab[8] && tab[6] != vide)  return tab [6] == 'X' ? 10 : -10; 
        
    if   (tab[0]==tab[3] && tab[0]==tab[6] && tab[0] != vide)  return tab [0] == 'X' ? 10 : -10;

    if   (tab[1]==tab[4] && tab[7]==tab[4] && tab[1] != vide)  return tab [1] == 'X' ? 10 : -10;
        
    if   (tab[2]==tab[5] && tab[8]==tab[2] && tab[2] != vide)  return tab [2] == 'X' ? 10 : -10;
        
    if   (tab[0]==tab[4] && tab[4]==tab[8] && tab[0] != vide)  return tab [0] == 'X' ? 10 : -10;
        
    if  (tab[2]==tab[4] && tab[4]==tab[6] && tab[2] != vide)  return tab [2] ==  'X' ? 10 : -10;

    return 0;
}

/* procedure qy=ui intialise les case du jeu à vide*/
void init_board (char *b)
{
        for (int i = 0; i< 9 ; ++i){b[i] = vide;}
}

/*check la fin du jeu*/
int is_game_over(char *board)
{
    // Check rows
    for (int i = 0; i < 9; i += 3)
    {
        if (board[i] != '.' && board[i] == board[i + 1] && board[i] == board[i + 2]) return 1;
    }
    // Check columns
    for (int i = 0; i < 3; i++)
    {
        if (board[i] != '.' && board[i] == board[i + 3] && board[i] == board[i + 6]) 
            return 1;
    }
    // Check diagonals
    if (board[0] != '.' && board[0] == board[4] && board[0] == board[8]) return 1;
    if (board[2] != '.' && board[2] == board[4] && board[2] == board[6]) return 1;
    
    // Check si une case de jeu est vide 
    for (int i = 0; i < 9; i++)
    {
        if (board[i] == '.') return 0;
    }

    return 2;
}

/* algorithme minmax recursif */
int minmax(node *root, int depth, int maximizingPlayer)
{
   
    generateChildren(root);
    
    if (!root-> nb_fils || depth == 0)
        return eval_score(root->state);

    
    if (maximizingPlayer)
    {
        int best = INT_MIN;
        for (int i = 0; i <  9; i++)
        {
            if (root->children[i])
            {
                int val = minmax(root->children[i], depth- 1, 0);
                best = max(best, val);
            }
        }
        return best;
    }
    else
    
    {
        int best = INT_MAX;
       
        for (int i = 0; i < 9 ; i++)
        {
            if (root->children[i])
            {
                int val = minmax(root->children[i], depth-1, 1);
                best = min(best, val);
            }
        }
        return best;
    }
}

/*fonction main principale qui orchestre le jeu ! */
int main(int argc , char ** argv )
{
    char * board = malloc (sizeof (char)*9);
    init_board(board);
    show_board(board);

    
    char currentPlayer = 'X';

    while (!is_game_over(board))
    {
        if (currentPlayer == 'X')
        {
            printf("\033[1;34mVeuillez Choisir  un deplacement ([pos : 0-9] du haut vers le bas)\033[0m");
            int pos;
            scanf("%d", &pos);
            if (!isValidMove(board, pos))
            {
                                printf("le deplacement demande nest pas valide !\n");
                                continue;
            }
           
            board[pos] = currentPlayer;
            show_board(board);
            currentPlayer = (currentPlayer == 'X') ? 'O' : 'X';
        } else
        {
            
            printf("deplacement de la machine: ");
            node * root = createNode(board, currentPlayer);
            generateChildren(root);
            int bestScore = INT_MIN;
            node * bestChild = NULL;
            printf ("nombre de fils : %d\n" ,root->nb_fils );
            for (int i = 0; i < 9; i++) 
            {
                if (root->children[i])
                {
                    int score = minmax(root->children[i], 0, 0);             
                    if (score > bestScore)
                    {
                        bestScore = score;
                        bestChild = root->children[i];
                    }
                }
            }
            board[bestChild->move_to_state] = currentPlayer;
            show_board(board);
            printf ("\033[0;31m le meilleur deplacement que la machine  a trouve est : %d\033[0m\n" , bestChild->move_to_state);
            currentPlayer = (currentPlayer == 'X') ? 'O' : 'X';
        }
  
        
        
    }
    printf("\033[0;36m***La partie du jeu est termine!***\033[0m\n");
    if (eval_score(board) == 10)
    {
        printf("\033[1;34mbravo vous avez gagner la partie!\033[0m\n");
    } else 
    
    if (eval_score(board) == -10)
    {
        printf("033[0;31ml'ordinateur a gagne ! wins!\033[0m\n");
    }
    else {
        printf("Egalite entre la machine et vous!\n");
    }
    return EXIT_SUCCESS;
}
