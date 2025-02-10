# TSSD
//202404005 : Shubham Chavda
//202401486 : Triambika Vashisth
//202401189 : Satvik Parihar
//202404007 : Devershee Samirbhai Dave

#include "Snake.h"
#include "GameEngine.h"
#include <iostream>

using namespace std;

int main() {
    char choice;
    
    do {
        int width = 20, height = 10;
        Snake snake(width, height);
        GameEngine engine(width, height);
        
        engine.run(snake);
        
        cout << "Do you want to play again? (y/n): ";
        cin >> choice;
        choice = tolower(choice);
    } while (choice == 'y');
    
    cout << "Thanks for playing!" << endl;
    return 0;
}
