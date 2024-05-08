me salen estos errores en mi codi

S D:\Juan Pablo Hernandez Reyes 4D\ea2\P3-EAS02> mingw32-make
g++ -Isrc/include -c *.cpp
g++ *.o -o main -Lsrc/lib -lsfml-graphics -lsfml-window -lsfml-system -lopengl32
PS D:\Juan Pablo Hernandez Reyes 4D\ea2\P3-EAS02> mingw32-make
g++ -Isrc/include -c *.cpp
grid.cpp:8:7: error: redefinition of 'class Grid'
    8 | class Grid {
      |       ^~~~
In file included from grid.cpp:1:
grid.hpp:7:7: note: previous definition of 'class Grid'
    7 | class Grid {
      |       ^~~~
mingw32-make: *** [makefile:4: compile] Error 1   
PS D:\Juan Pablo Hernandez Reyes 4D\ea2\P3-EAS02> 










corrige los errores que te mostre para que compile correctamente este codigo

#include "grid.hpp"
#include <SFML/Graphics.hpp>
#include <vector>

using namespace sf;
using namespace std;

class Grid {
private:
    int rows;
    int cols;
    int sizeX;
    int sizeY;
    vector<vector<int>> grid;
    vector<vector<int>> next;

public:
    Grid(int rows, int cols, int width, int height);

    void drawTo(RenderWindow &window);

    void click(int x, int y);

    void update();

private:
    int contarEspaciosDisponibles(int i, int j);
};

Grid::Grid(int rows, int cols, int width, int height)
{
    this->rows = rows;
    this->cols = cols;
    this->sizeX = width / cols;
    this->sizeY = height / rows;
    for (int i = 0; i < rows; i++)
    {
        this->grid.push_back({});
        for (int j = 0; j < cols; j++)
        {
            this->grid[i].push_back(0);
        }
    }
    this->next = vector<vector<int>>(rows, vector<int>(cols, 0));
}

void Grid::drawTo(RenderWindow &window)
{
    for (int i = 0; i < rows; i++)
    {
        for (int j = 0; j < cols; j++)
        {
            RectangleShape rect(Vector2f(sizeX, sizeY));
            rect.setOutlineColor(Color::White);
            if (grid[j][i] == 0)
            {
                rect.setFillColor(Color::Black);
            }
            else
            {
                rect.setFillColor(Color::White);
            }
            rect.setOutlineThickness(1);
            rect.setPosition(Vector2f(j * rect.getSize().x, i * rect.getSize().y));
            window.draw(rect);
        }
    }
}

void Grid::click(int x, int y)
{
    int indexX = x / this->sizeX;
    int indexY = y / this->sizeY;
    grid[indexX][indexY] = 1;
}

void Grid::update()
{
    for (int i = 0; i < this->rows; i++)
    {
        for (int j = 0; j < this->cols; j++)
        {
            if (grid[i][j] == 1) {
                int availableSpaces = contarEspaciosDisponibles(i, j);
                if (availableSpaces > 0) {
                    // Randomly choose one of the available spaces
                    int moveDirection = rand() % availableSpaces;

                    // Move the sand to the chosen direction
                    if (j < cols - 1 && grid[i + 1][j] == 0 && (moveDirection == 0 || (moveDirection == 1 && grid[i + 2][j - 1] == 1))) {
                        next[i + 1][j] = 1;
                    } else if (j > 0 && grid[i + 1][j - 1] == 0 && (moveDirection == 1 || (moveDirection == 0 && grid[i + 2][j + 1] == 1))) {
                        next[i + 1][j - 1] = 1;
                    } else if (j < cols - 1 && grid[i + 2][j] == 0) {
                        next[i + 2][j] = 1;
                    } else if (j > 0 && grid[i + 2][j - 1] == 0) {
                        next[i + 2][j - 1] = 1;
                    } else if (j < cols - 1 && grid[i + 1][j] == 0) {
                        next[i + 1][j] = 1;
                    } else if (j > 0 && grid[i + 1][j - 1] == 0) {
                        next[i + 1][j - 1] = 1;
                    }
                }
            }
        }
    }
    this->grid = this->next;
}

int Grid::contarEspaciosDisponibles(int i, int j)
{
    int availableSpaces = 0;
    if (i < rows - 2 && (grid[i + 1][j] == 0 || (j > 0 && grid[i + 1][j - 1] == 0))) {
        availableSpaces++;
    }
    if (j < cols - 1 && i < rows - 2 && (grid[i + 1][j] == 0 || (j > 0 && grid[i + 1][j - 1] == 0))) {
        availableSpaces++;
    }
    return availableSpaces;
}

int main()
{
    int width = 800;
    int height = 600;
    int rows = 40;
    int cols = 30;

    RenderWindow window(VideoMode(width, height), "Sand Simulation");

    Grid grid(rows, cols, width, height);

    while (window.isOpen())
    {
        Event event;
        while (window.pollEvent(event))
        {
            if (event.type == Event::Closed)
                window.close();
            if (event.type == Event::MouseButtonPressed)
            {
                if (event.mouseButton.button == Mouse::Left)
                {
                    grid.click(event.mouseButton.x, event.mouseButton.y);
                }
            }
        }

        grid.update();

        window.clear();
        grid.drawTo(window);
        window.display();
    }

    return 0;
}
