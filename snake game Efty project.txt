#include <conio.h>
#include <stdio.h>
#include <stdlib.h>
#include <windows.h>
#include <time.h>
#define WIDTH 50
#define HEIGHT 25
int i, j, gameOver;
int x, y, fruitX, fruitY, score;
int tailX[100], tailY[100];
int nTail;
enum eDirecton { STOP = 0, LEFT, RIGHT, UP, DOWN };
enum eDirecton dir, prevDir;
HANDLE hConsole;
COORD cursorPosition;

void Setup();
void Draw();
void Input();
void Logic();
void GameOverScreen();
void SetColor(int color);
void GotoXY(int x, int y);
void DrawBorder();
void ClearGameArea();

void Setup() {
    gameOver = 0;
    dir = STOP;
    prevDir = STOP;
    x = WIDTH / 2;
    y = HEIGHT / 2;
    fruitX = rand() % WIDTH;
    fruitY = rand() % HEIGHT;
    score = 0;
    nTail = 0;

    // Display game instructions
    printf("\t\t\t\t\t\t Welcome to Snake Safari\n");
    printf("\n\t\t\t\t\t\t Use 'WASD' keys to control the snake\n");
    printf("\n\t\t\t\t\t\t Press Enter to play\n");
    _getch(); 
    system("cls");
    DrawBorder(); 
}
// Draw the border
void DrawBorder() {
    SetColor(15); // White(15) for border

    GotoXY(0, 0);
    for (i = 0; i < WIDTH + 2; i++) printf("-");
    printf("\n");

    for (i = 0; i < HEIGHT; i++) {
        printf("|");
        for (j = 0; j < WIDTH; j++) {
            if (i == y && j == x)
                printf("*");
            else if (i == fruitY && j == fruitX)
                printf("%c", 254);
            else {
                int k, print = 0;
                for (k = 0; k < nTail; k++) {
                    if (tailX[k] == j && tailY[k] == i) {
                        printf("*");
                        print = 1;
                    }
                }
                if (!print)
                    printf(" ");
            }
        }
        printf("|\n");
    }

    GotoXY(0, HEIGHT + 1);
    for (i = 0; i < WIDTH + 2; i++) printf("-");
    printf("\n");

    SetColor(15); 
}
// Clear the game area
void ClearGameArea() {
    for (i = 1; i < HEIGHT + 1; i++) {
        GotoXY(1, i);
        for (j = 1; j < WIDTH + 1; j++) {
            printf(" ");
        }
    }
}
// Draw the game screen
void Draw() {
    ClearGameArea();
    GotoXY(x + 1, y + 1);  // Offset by 1 for the border
    SetColor(10); // Green(10) for snake head
    printf("*");
    SetColor(15);
    GotoXY(fruitX + 1, fruitY + 1);  // Offset by 1 for the border
    SetColor(12); // Red(12) for fruit
    printf("%c", 254);
    SetColor(15); 

    for (i = 0; i < nTail; i++) {
        if (tailX[i] != -1 && tailY[i] != -1) {
            GotoXY(tailX[i] + 1, tailY[i] + 1);  // Offset by 1 for the border
            SetColor(10); // Green(10) for snake tail
            printf("*");
            SetColor(15); // Reset to default color
        }
    }
    GotoXY(WIDTH + 4, 2); // Position outside the game area
    printf("Score: %d", score);
}
// Get user input
void Input() {
    if (_kbhit()) {
        switch (_getch()) {
        case 'a':
            if (prevDir != RIGHT) dir = LEFT;
            break;
        case 'd':
            if (prevDir != LEFT) dir = RIGHT;
            break;
        case 'w':
            if (prevDir != DOWN) dir = UP;
            break;
        case 's':
            if (prevDir != UP) dir = DOWN;
            break;
        case 'x':
            gameOver = 1;
            break;
        }
    }
}
// Game logic
void Logic() {
    int prevX = tailX[0];
    int prevY = tailY[0];
    int prev2X, prev2Y;
    tailX[0] = x;
    tailY[0] = y;
    for (i = 1; i < nTail; i++) {
        prev2X = tailX[i];
        prev2Y = tailY[i];
        tailX[i] = prevX;
        tailY[i] = prevY;
        prevX = prev2X;
        prevY = prev2Y;
    }
    prevDir = dir;
    switch (dir) {
    case LEFT:
        x--;
        break;
    case RIGHT:
        x++;
        break;
    case UP:
        y--;
        break;
    case DOWN:
        y++;
        break;
    default:
        break;
    }
    if (x >= WIDTH || x < 0 || y >= HEIGHT || y < 0)
        gameOver = 1;

    for (i = 0; i < nTail; i++)
        if (tailX[i] == x && tailY[i] == y)
            gameOver = 1;

    if (x == fruitX && y == fruitY) {
        score += 10;
        fruitX = rand() % WIDTH;
        fruitY = rand() % HEIGHT;
        nTail++;
    }
}

// Display Game Over screen
void GameOverScreen() {
    system("cls");
    printf("\n\n\n\t\tGAME OVER\n");
    printf("\t\tScore: %d\n\n", score);
    printf("\t\tPress any key to exit\n");
    _getch();
}

// Set console text color
void SetColor(int color) {
    SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), color);
}

// Move the console cursor to (x, y)
void GotoXY(int x, int y) {
    cursorPosition.X = x;
    cursorPosition.Y = y;
    SetConsoleCursorPosition(hConsole, cursorPosition);
}
// Main function
int main() {
    hConsole = GetStdHandle(STD_OUTPUT_HANDLE);
    srand(time(NULL)); // Initialize random seed

    Setup();
    while (!gameOver) {
        Draw();
        Input();
        Logic();
        Sleep(100 - (nTail > 20 ? 20 : nTail)); // Increase speed as snake grows
    }
    GameOverScreen();
    return 0;
}
