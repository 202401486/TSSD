# TSSD

#include <iostream>
#include <deque>
#include <chrono>
#include <cstdlib>
#include <conio.h>
#include <windows.h>
#include <limits>

using namespace std;

class SnakeGame {
private:
    int width, height;
    deque<pair<int, int>> snake;
    pair<int, int> direction;
    pair<int, int> fruit;
    bool running;
    int score;
    static int high_score;  // Static high score
    int frameDelay;
    HANDLE hConsole;
    CHAR_INFO *screenBuffer;
    COORD bufferSize;
    SMALL_RECT screenRect;

    void initScreenBuffer() {
        hConsole = GetStdHandle(STD_OUTPUT_HANDLE);
        if (hConsole == INVALID_HANDLE_VALUE) {
            cout << "Error: Could not get console handle!" << endl;
            exit(1);
        }
        bufferSize = { (short)width, (short)height };
        screenRect = { 0, 0, (short)(width - 1), (short)(height - 1) };
        screenBuffer = new CHAR_INFO[width * height];
    }

    void clearScreen() {
        COORD topLeft = { 0, 0 };
        DWORD written;
        FillConsoleOutputCharacter(hConsole, ' ', width * height, topLeft, &written);
        FillConsoleOutputAttribute(hConsole, FOREGROUND_RED | FOREGROUND_INTENSITY, width * height, topLeft, &written);
        SetConsoleCursorPosition(hConsole, topLeft);
    }

    void updateScreenBuffer() {
        for (int i = 0; i < width * height; i++) {
            screenBuffer[i].Char.AsciiChar = ' ';
            screenBuffer[i].Attributes = FOREGROUND_GREEN | FOREGROUND_INTENSITY;
        }

        for (int x = 0; x < width; x++) {
            screenBuffer[x].Char.AsciiChar = '\xDB'; // Solid block for top border
            screenBuffer[x + (height - 1) * width].Char.AsciiChar = '\xDB'; // Bottom border
        }
        for (int y = 0; y < height; y++) {
            screenBuffer[y * width].Char.AsciiChar = '\xDB'; // Left border
            screenBuffer[y * width + width - 1].Char.AsciiChar = '\xDB'; // Right border
        }

        for (auto &segment : snake) {
            int index = segment.second * width + segment.first;
            screenBuffer[index].Char.AsciiChar = (segment == snake.back() ? '\xFE' : '\xB2'); //Head

        }

        int fruitIndex = fruit.second * width + fruit.first;
        screenBuffer[fruitIndex].Char.AsciiChar = '*';

        WriteConsoleOutput(hConsole, screenBuffer, bufferSize, { 0, 0 }, &screenRect);
    }

    void spawnFruit() {
        srand(time(0));
        while (true) {
            fruit = { rand() % (width - 2) + 1, rand() % (height - 2) + 1 };
            bool occupied = false;
            for (auto &segment : snake) {
                if (segment == fruit) {
                    occupied = true;
                    break;
                }
            }
            if (!occupied) break;
        }
    }

    void moveSnake() {
        int headX = snake.back().first;
        int headY = snake.back().second;
        int newX = headX + direction.first;
        int newY = headY + direction.second;

        if (newX <= 0 || newX >= width - 1 || newY <= 0 || newY >= height - 1) {
            running = false;
            return;
        }

        for (auto &segment : snake) {
            if (segment.first == newX && segment.second == newY) {
                running = false;
                return;
            }
        }

        snake.push_back({ newX, newY });

        if (make_pair(newX, newY) == fruit) {
            score += 10;
            if (score > high_score) {
                high_score = score;
            }
            spawnFruit();
        } else {
            snake.pop_front();
        }
    }

    void changeDirection(char key) {
        switch (key) {
            case 'a': case 'A': if (direction != make_pair(1, 0)) direction = { -1, 0 }; break;
            case 'd': case 'D': if (direction != make_pair(-1, 0)) direction = { 1, 0 }; break;
            case 'w': case 'W': if (direction != make_pair(0, 1)) direction = { 0, -1 }; break;
            case 's': case 'S': if (direction != make_pair(0, -1)) direction = { 0, 1 }; break;
        }
    }

public:
    SnakeGame(int w = 20, int h = 10) : width(w), height(h), running(true), score(0), frameDelay(100) {
        snake.push_back({ w / 2, h / 2 });
        direction = { 1, 0 };
        spawnFruit();
        initScreenBuffer();
    }

    ~SnakeGame() {
        delete[] screenBuffer;
    }

    void countdown() {
        for (int i = 3; i > 0; --i) {
            cout << "\rGame starts in " << i << " seconds..." << flush;
            Sleep(1000);
        }
        cout << "\r                          \r";
    }

    void run() {
        clearScreen();
        countdown();

        while (running) {
            if (_kbhit()) {
                char key = _getch();
                changeDirection(key);
            }

            moveSnake();
            updateScreenBuffer();
            Sleep(frameDelay);
        }

        cout << "\nGame Over! Final Score: " << score << endl;
        cout << "High Score: " << high_score << endl;
    }
};

//  Initialize static variable
int SnakeGame::high_score = 0;

int main() {
    char choice;

    while (true) {
        SnakeGame game;
        game.run();

        //  FLUSH INPUT BUFFER
        cin.clear();
        cin.ignore(numeric_limits<streamsize>::max(), '\n');

        cout << "Do you want to play again? (y/n): ";
        while (true) {
            cin >> choice;
            choice = tolower(choice);

            if (choice == 'y') {
                system("cls");
                break;
            } else if (choice == 'n') {
                cout << "Thanks for playing!" << endl;
                cout << "Press any key to exit..." << endl;
                _getch();  // Wait for key press before exiting
                return 0;
            } else {
                cout << "Invalid input. Enter 'y' for yes or 'n' for no: ";
                cin.clear();
                cin.ignore(numeric_limits<streamsize>::max(), '\n');
            }
        }
    }
}
