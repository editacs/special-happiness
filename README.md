[Uploading Minesweep// Edit Yesayan
// Comp222
// Project


#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>
#include <time.h>

#define MAX_ROWS 50
#define MAX_COLS 50

typedef struct {
    int hasMine;
    int adjcount;
    int uncovered;
    int flagged;
} cell;

cell **board;
int rows, cols, mines;

// Function Prototypes
void print_board(void);
void command_new(int r, int c, int m);
void command_show(void);
void command_uncover(int r, int c);
void command_flag(int r, int c);
void command_unflag(int r, int c);
int check_win(void);
void uncover_recursive(int r, int c);
int alldigits(const char *s);
int is_valid_cell(int r, int c);
void init_adj_counts(void);
void free_memory(void);

// Updated display_cell function
void display_cell(const cell *c)
{
    if (c->flagged)
    {
        printf("%2s ", "P");
    }
    else if (!c->uncovered)
    {
        printf("%2s ", "/");
    }
    else if (c->hasMine)
    {
        printf("%2s ", "*");
    }
    else if (c->adjcount > 0)
    {
        printf("%2d ", c->adjcount);
    }
    else
    {
        printf("%2s ", ".");
    }
}

// Updated init_adj_counts function
void init_adj_counts(void)
{
    const int rowneighbors[] = {-1, -1, +0, +1, +1, +1, +0, -1};
    const int colneighbors[] = {+0, +1, +1, +1, +0, -1, -1, -1};

    for (int i = 0; i < rows; i++)
    {
        for (int j = 0; j < cols; j++)
        {
            int mineCount = 0;
            for (int k = 0; k < 8; k++)
            {
                const int rn = i + rowneighbors[k];
                const int cn = j + colneighbors[k];
                if (is_valid_cell(rn, cn) && board[rn][cn].hasMine)
                {
                    mineCount++;
                }
            }
            board[i][j].adjcount = mineCount;
        }
    }
}

// Updated command_flag function
void command_flag(int r, int c)
{
    r--;
    c--;

    if (is_valid_cell(r, c))
    {
        if (!board[r][c].flagged)
        {
            board[r][c].flagged = 1;
        }
        else
        {
            printf("Cell already flagged\n");
        }
    }
}

// Updated command_unflag function

void command_unflag(int r, int c)
{
    // Adjust coordinates to 0-based indexing
    r--;
    c--;

    if (is_valid_cell(r, c))
    {
        if (board[r][c].flagged)
        {
            board[r][c].flagged = 0;
        }
        else
        {
            printf("Cell is not flagged\n");
        }
    }
}


// Updated command_uncover function
void command_uncover(int r, int c)
{
    // Adjust coordinates to 0-based indexing
    r--;
    c--;

    if (is_valid_cell(r, c))
    {
        if (board[r][c].flagged)
        {
            printf("Cannot uncover a flagged cell. Unflag it first.\n");
        }
        else
        {
            if (board[r][c].hasMine)
            {
                printf("Game over! You uncovered a mine at (%d, %d).\n", r + 1, c + 1);

                // Reveal all mine positions
                for (int i = 0; i < rows; i++)
                {
                    for (int j = 0; j < cols; j++)
                    {
                        if (board[i][j].hasMine)
                        {
                            board[i][j].uncovered = 1;
                        }
                    }
                }

                print_board(); // Display the board with mine positions
                exit(EXIT_SUCCESS);
            }
            else
            {
                printf("Uncovering: %d %d\n", r + 1, c + 1);

                uncover_recursive(r, c);

                if (!board[r][c].uncovered)
                {
                    print_board(); // Display the updated board after uncovering
                }
            }
        }
    }
}


// uncover_recursive function

void uncover_recursive(int r, int c)
{
    const int rowneighbors[] = {-1, -1, +0, +1, +1, +1, +0, -1};
    const int colneighbors[] = {+0, +1, +1, +1, +0, -1, -1, -1};

    // Check if the current cell is valid and needs to be uncovered
    if (!is_valid_cell(r, c) || board[r][c].uncovered || board[r][c].hasMine)
    {
        return;
    }

    // Uncover the current cell
    board[r][c].uncovered = 1;

    // If the adjacency count is greater than 0, stop recursion
    if (board[r][c].adjcount > 0)
    {
        return;
    }

    // Recursively uncover each valid neighbor
    for (int n = 0; n < 8; n++)
    {
        const int rn = r + rowneighbors[n];
        const int cn = c + colneighbors[n];

        if (is_valid_cell(rn, cn) && !board[rn][cn].uncovered)
        {
            uncover_recursive(rn, cn);
        }
    }
}

// Updated check_win function

int check_win()
{
    for (int r = 0; r < rows; r++)
    {
        for (int c = 0; c < cols; c++)
        {
            // Check if there is any mine that is not flagged or any non-mine that is still uncovered
            if ((!board[r][c].hasMine && !board[r][c].uncovered))
            {
                return 0; // Not a win condition
            }
        }
    }
    return 1; // Win condition satisfied
}


// Additional functions
void print_board(void)
{
    printf("\n   ");
    for (int j = 0; j < cols; j++)
    {
        printf("%2d ", j + 1);
    }
    printf("\n");

    for (int i = 0; i < rows; i++)
    {
        printf("%2d ", i + 1);
        for (int j = 0; j < cols; j++)
        {
            display_cell(&board[i][j]);
        }
        printf("\n");
    }
    printf("\n");
}

// Updated command_new function with pointers
void command_new(int r, int c, int m)
{
    rows = r;
    cols = c;
    mines = m;

    // Allocate memory for the board
    board = (cell **)malloc(rows * sizeof(cell *));
    if (board == NULL)
    {
        fprintf(stderr, "Memory allocation error for rows\n");
        exit(EXIT_FAILURE);
    }

    for (int i = 0; i < rows; i++)
    {
        board[i] = (cell *)malloc(cols * sizeof(cell));
        if (board[i] == NULL)
        {
            fprintf(stderr, "Memory allocation error for columns\n");
            exit(EXIT_FAILURE);
        }
    }

    // Initialize the board
    for (int i = 0; i < rows; i++)
    {
        for (int j = 0; j < cols; j++)
        {
            board[i][j].hasMine = 0;
            board[i][j].adjcount = 0;
            board[i][j].uncovered = 0;
            board[i][j].flagged = 0;
        }
    }

    srand((unsigned int)time(NULL));
    // Initialize random seed
    for (int i = 0; i < mines; i++)
    {
        int randRow, randCol;
        do
        {
            randRow = rand() % rows;
            randCol = rand() % cols;
        } while (board[randRow][randCol].hasMine);
        board[randRow][randCol].hasMine = 1;
    }

    init_adj_counts();
}

// Function to show the board
void command_show(void)
{
    print_board();
}

// Function to free allocated memory
void free_memory(void)
{
    // Free memory for the board
    for (int i = 0; i < rows; i++)
    {
        free(board[i]);
    }
    free(board);
}

int is_valid_cell(int r, int c)
{
    return (r >= 0 && r < rows && c >= 0 && c < cols);
}

int alldigits(const char *s)
{
    size_t len = strlen(s);
    for (size_t i = 0; i < len; i++)
    {
        if (!isdigit((unsigned char)s[i]))
            return 0;
    }
    return 1;
}

int main(void)
{
    char input[100];
    char *tokens[10];
    int tokencount;

    while (1)
    {
        printf("Enter command: ");
        if (fgets(input, sizeof(input), stdin) == NULL)
        {
            printf("Error reading input\n");
            exit(EXIT_FAILURE);
        }

        // Tokenize the input
        tokencount = 0;
        tokens[tokencount] = strtok(input, " \n");
        while (tokens[tokencount] != NULL && tokencount < 10)
        {
            tokencount++;
            tokens[tokencount] = strtok(NULL, " \n");
        }

        // Check for legal commands
        int cmdlegal = 0;
        char *legalcommands[] = {"new", "show", "quit", "uncover", "flag", "unflag"};
        for (int i = 0; i < 6; i++)
        {
            if (strcmp(tokens[0], legalcommands[i]) == 0)
            {
                cmdlegal = i + 1;
                break;
            }
        }

        // Process commands
        switch (cmdlegal)
        {
        case 1: // new
            if (tokencount == 4 && alldigits(tokens[1]) && alldigits(tokens[2]) && alldigits(tokens[3]))
            {
                command_new(atoi(tokens[1]), atoi(tokens[2]), atoi(tokens[3]));
                print_board();
            }
            else
            {
                printf("Invalid arguments for 'new' command\n");
            }
            break;
        case 2: // show
            command_show();
            break;
        case 3: // quit
            free_memory();
            exit(EXIT_SUCCESS);
        case 4: // uncover
            if (tokencount == 3 && alldigits(tokens[1]) && alldigits(tokens[2]))
            {
                command_uncover(atoi(tokens[1]), atoi(tokens[2]));
                print_board();
                if (check_win())
                {
                    printf("Congratulations! You win!\n");
                    free_memory();
                    exit(EXIT_SUCCESS);
                }
            }
            else
            {
                printf("Invalid arguments for 'uncover' command\n");
            }
            break;
        case 5: // flag
            if (tokencount == 3 && alldigits(tokens[1]) && alldigits(tokens[2]))
            {
                command_flag(atoi(tokens[1]), atoi(tokens[2]));
                print_board();
            }
            else
            {
                printf("Invalid arguments for 'flag' command\n");
            }
            break;
        case 6: // unflag
            if (tokencount == 3 && alldigits(tokens[1]) && alldigits(tokens[2]))
            {
                command_unflag(atoi(tokens[1]), atoi(tokens[2]));
                print_board();
            }
            else
            {
                printf("Invalid arguments for 'unflag' command\n");
            }
            break;
        default:
            printf("Illegal command. Please enter a valid command.\n");
            break;
        }
    }

    return 0;
}
er .c…]()
