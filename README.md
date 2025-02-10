# TSSD

#include <iostream>
#include <deque>
#include <chrono>
#include <cstdlib>
#include <conio.h>
#include <windows.h>

using namespace std;

class SnakeGame {
private:
    int width, height;
    deque<pair<int, int>> snake;  
    pair<int, int> direction;    
    pair<int, int> fruit;
    bool running;
    int fruit_count;
    chrono::steady_clock::time_point lastTime;
    int frameDelay;
    HANDLE hConsole;
    CHAR_INFO *screenBuffer;
    COORD bufferSize;
    SMALL_RECT screenRect;

    void initScreenBuffer() {
        hConsole = GetStdHandle(STD_OUTPUT_HANDLE);
        bufferSize = { (short)width, (short)height };
        screenRect = { 0, 0, (short)(width - 1), (short)(height - 1) };
        screenBuffer = new CHAR_INFO[width * height];
    }

    void updateScreenBuffer() {
        for (int i = 0; i < width * height; i++) {
            screenBuffer[i].Char.AsciiChar = ' ';
            screenBuffer[i].Attributes = FOREGROUND_GREEN | FOREGROUND_INTENSITY;
        }

        // Draw border
        for (int x = 0; x < width; x++) {
            screenBuffer[x].Char.AsciiChar = '#';
            screenBuffer[x + (height - 1) * width].Char.AsciiChar = '#';
        }
        for (int y = 0; y < height; y++) {
            screenBuffer[y * width].Char.AsciiChar = '#';
            screenBuffer[y * width + width - 1].Char.AsciiChar = '#';
        }

        // Draw snake
        for (auto &segment : snake) {
            int index = segment.second * width + segment.first;
            screenBuffer[index].Char.AsciiChar = (segment == snake.back() ? '@' : 'o');
        }

        // Draw fruit
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
            fruit_count++;
            if (fruit_count >= 7) {
                running = false;
                return;
            }
            spawnFruit();
        }
        else {
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
    SnakeGame(int w = 20, int h = 10) : width(w), height(h), running(true), fruit_count(0), frameDelay(100) {
        snake.push_back({ w / 2, h / 2 });
        direction = { 1, 0 };
        spawnFruit();
        lastTime = chrono::steady_clock::now();
        initScreenBuffer();
    }

    ~SnakeGame() {
        delete[] screenBuffer;
    }

    void run() {
        auto startTime = chrono::steady_clock::now();
        auto currentTime = startTime;

        cout << "Game starts in 3 seconds..." << flush;
        while (chrono::duration_cast<chrono::seconds>(currentTime - startTime).count() < 3) {
            currentTime = chrono::steady_clock::now();
            cout << "\rGame starts in " << 3 - chrono::duration_cast<chrono::seconds>(currentTime - startTime).count() << " seconds..." << flush;
        }
        cout << endl;

        while (running) {
            auto currentTime = chrono::steady_clock::now();
            chrono::duration<int, std::milli> elapsedTime = chrono::duration_cast<chrono::milliseconds>(currentTime - lastTime);

            if (elapsedTime.count() >= frameDelay) {
                if (_kbhit()) {
                    char key = _getch();
                    changeDirection(key);
                }

                moveSnake();
                updateScreenBuffer();
                lastTime = chrono::steady_clock::now();
            }
        }
        cout << (fruit_count >= 7 ? "You Win!" : "Game Over!") << endl;
    }
};

int main() {
    char choice;

    while (true) {
        SnakeGame game;
        game.run();

        cout << "Do you want to play again? (y/n): ";
        while (true) {
            cin >> choice;
            choice = tolower(choice);

            if (choice == 'y') {
                break;
            } else if (choice == 'n') {
                cout << "Thanks for playing!" << endl;
                return 0;
            } else {
                cout << "Invalid input. Enter 'y' for yes or 'n' for no: ";
                cin.clear();
                cin.ignore(numeric_limits<streamsize>::max(), '\n');
            }
        }
    }
}
