---
title: i should probably refactor
tags: puzzle
comments: true
math: true
---

i should definitely refactor. 

<!--more-->

# problem

![grid](/assets/img/js%2004.24%20hooks%2010.png)

this grid can be partitioned into nine L-shaped "hooks": the largest 9-by-9 (so, 17 squares total), then 8-by-8, all the way down to 1-by-1 (which is just a square). partition the grid into said hooks, and place nine 9's in one of the hooks, eight 8's in another, and so on, such that:

* the occupied squares must form a connected region, where two squares are connected if they share an edge
* every 2-by-2 box must contain at least one empty square
* the hint cells are empty
* the cells orthogonally adjacent to a hint cell sum up to the hint

# solution

well, didn't we [do this before](/2023/07/06/js0623.html)? almost -- the constraints were different, but our general procedure for partitioning the grid, creating configurations, and high-level backtracking should be fine. it remains to modify the small-scale solving functions, namely `forced_board`. instead of worrying about each row and column, we instead try and solve each constraint separately, then taking the intersection of these solutions to see which cells are forced (hint: it's a lot of them). we'll be using terms brought over from the previous puzzle, so take a look at that first for a refresher.

for an individual constraint, and the board's configuration, we can use backtracking to solve it. each of the (up to) four orthogonally adjacent cells can either be filled or not, so we backtrack on each cell. 

```c++
// solves a single constraint by backtracking
// order: top, left, right, bottom
vector<board> solve_constraint(board &b, board &config, board &constraints, cell pos, int step, int sum) {
    int N = config.size();
    vector<board> result;
    // print_board(b);
    switch (step) {
        case 0:
            // std::cout << "Top" << std::endl;
            if (pos.first != 0) {
                auto c = pos + up;
                if (constraints[c.first][c.second] == -1) {
                    b[c.first][c.second] = config[c.first][c.second];
                    auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum + b[c.first][c.second]);
                    result.insert(result.end(), partial_solution.begin(), partial_solution.end());
                    b[c.first][c.second] = -1;
                } else std::cout << "How did we get here?" << std::endl;
                b[c.first][c.second] = -2;
                auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum);
                result.insert(result.end(), partial_solution.begin(), partial_solution.end());
                b[c.first][c.second] = -1;
            } else {
                auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum);
                result.insert(result.end(), partial_solution.begin(), partial_solution.end());
            }
            break;
        case 1:
            // std::cout << "Left" << std::endl;
            if (pos.second != 0) {
                auto c = pos + left;
                if (constraints[c.first][c.second] == -1) {
                    b[c.first][c.second] = config[c.first][c.second];
                    auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum + b[c.first][c.second]);
                    result.insert(result.end(), partial_solution.begin(), partial_solution.end());
                    b[c.first][c.second] = -1;
                } else std::cout << "How did we get here?" << std::endl;
                b[c.first][c.second] = -2;
                auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum);
                result.insert(result.end(), partial_solution.begin(), partial_solution.end());
                b[c.first][c.second] = -1;
            } else {
                auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum);
                result.insert(result.end(), partial_solution.begin(), partial_solution.end());
            }
            break;
        case 2:
            // std::cout << "Right" << std::endl;
            if (pos.second != N - 1) {
                auto c = pos + right;
                if (constraints[c.first][c.second] == -1) {
                    b[c.first][c.second] = config[c.first][c.second];
                    auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum + b[c.first][c.second]);
                    result.insert(result.end(), partial_solution.begin(), partial_solution.end());
                    b[c.first][c.second] = -1;
                } else std::cout << "How did we get here?" << std::endl;
                b[c.first][c.second] = -2;
                auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum);
                result.insert(result.end(), partial_solution.begin(), partial_solution.end());
                b[c.first][c.second] = -1;
            } else {
                auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum);
                result.insert(result.end(), partial_solution.begin(), partial_solution.end());
            }
            break;
        case 3:
            // std::cout << "Bottom" << std::endl;
            if (pos.first != N - 1) {
                auto c = pos + down;
                if (constraints[c.first][c.second] == -1) {
                    b[c.first][c.second] = config[c.first][c.second];
                    auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum + b[c.first][c.second]);
                    result.insert(result.end(), partial_solution.begin(), partial_solution.end());
                    b[c.first][c.second] = -1;
                } else std::cout << "How did we get here?" << std::endl;
                b[c.first][c.second] = -2;
                auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum);
                result.insert(result.end(), partial_solution.begin(), partial_solution.end());
                b[c.first][c.second] = -1;
            } else {
                auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum);
                result.insert(result.end(), partial_solution.begin(), partial_solution.end());
            }
            break;
        case 4:
            if (sum == constraints[pos.first][pos.second]) {
                result.push_back(copy_board(b));
                // print_solution(b);
            }
            break;
        default:
            std::cout << "How did we get here?" << std::endl;
            break;
    }
    return result;
}
```

with the list of solutions found for each constraint, we can then find the intersection of these solutions to see if certain cells are forced to be filled or empty. 

```c++
// gives cells jointly forced by all constraints
// weaker than a "true" forcing since intersections between constraints are ignored
bool forced_board(board &b, board &config, board &constraints) {
    int N = config.size();
    for (int i = 0; i < N; ++i) {
        for (int j = 0; j < N; ++j) {
            if (constraints[i][j] != -1) {
                auto b_temp = create_constrained_board(constraints);
                cell pos = {i, j};
                auto constraint_solution = solve_constraint(b_temp, config, constraints, pos);
                if (constraint_solution.size() == 0) return false;
                if (i != 0) { // can look up
                    auto c = pos + up;
                    int forced_value = constraint_solution[0][c.first][c.second];
                    bool all_same = true;
                    for (auto& sol : constraint_solution) {
                        all_same = all_same && (sol[c.first][c.second] == forced_value);
                    }
                    if (all_same) b[c.first][c.second] = forced_value;
                }
                if (i != N - 1) { // can look down
                    auto c = pos + down;
                    int forced_value = constraint_solution[0][c.first][c.second];
                    bool all_same = true;
                    for (auto& sol : constraint_solution) {
                        all_same = all_same && (sol[c.first][c.second] == forced_value);
                    }
                    if (all_same) b[c.first][c.second] = forced_value;
                }
                if (j != 0) { // can look left
                    auto c = pos + left;
                    int forced_value = constraint_solution[0][c.first][c.second];
                    bool all_same = true;
                    for (auto& sol : constraint_solution) {
                        all_same = all_same && (sol[c.first][c.second] == forced_value);
                    }
                    if (all_same) b[c.first][c.second] = forced_value;
                }
                if (j != N - 1) { // can look right
                    auto c = pos + right;
                    int forced_value = constraint_solution[0][c.first][c.second];
                    bool all_same = true;
                    for (auto& sol : constraint_solution) {
                        all_same = all_same && (sol[c.first][c.second] == forced_value);
                    }
                    if (all_same) b[c.first][c.second] = forced_value;
                }
            }
        }
    }

    return true;
}
```

that's basically it. these are the main changes, since `solve_puzzle` depends on `solve_config` and `solve_config`'s cell check, `fill_cell`, depends on existing functions; the only thing we really needed to change was `check_config` which wasn't that bad. it's also very, very fast. the solution

```
9 @ 9 9 @ @ @ @ @ 
7 @ @ 7 @ @ 7 7 9 
6 @ @ 5 5 5 8 @ 9 
6 3 4 4 @ 5 @ 7 9 
@ @ 2 @ 4 @ 8 @ @ 
@ @ 2 1 4 5 8 7 9 
@ @ 3 @ @ 3 @ @ 9 
6 6 6 6 @ @ 8 7 9 
@ 8 @ 8 8 @ 8 @ @ 
```

was found in less than half a second. 

we share our entire implementation below. i think i did a good job at optimizing things, but the code is just really badly written. (i even optimized each hook adding stage, since `check_config` can check partially-completed configurations if filled hooks completely surround a constraint.) if i had to refactor, i'd turn the directions into an array instead of having four `if` statements for every function. a `check_bounds` function would do wonders. 

```c++
#include <iostream>
#include <vector>
#include <string>
#include <cctype>
#include <stack>
#include <queue>
#include <algorithm>
#include <tuple>
#include <chrono>
#include <functional>
#include <numeric>

using std::vector;
using std::pair;
using std::tuple;
using std::make_tuple;
using std::get;
using std::string;
using std::stack;
using std::queue;
using std::prev_permutation;
using std::chrono::steady_clock;
using std::chrono::duration;
using std::chrono::milliseconds;
using std::chrono::duration_cast;
using std::function;
using std::accumulate;

typedef vector<vector<int>> board;
typedef pair<int, int> cell;

template <typename T, typename U> pair<T, U> operator+(const pair<T, U> &a, const pair<T, U> &b) {
    return {a.first + b.first, a.second + b.second};
}
template <typename T, typename U> pair<T, U> operator-(const pair<T, U> &a, const pair<T, U> &b) {
    return {a.first - b.first, a.second - b.second};
}
template <typename T, typename U> pair<T, U> operator*(const pair<T, U> &a, const int &b) {
    return {a.first * b, a.second * b};
}

const cell up = {-1, 0}, down = {1, 0}, left = {0, -1}, right = {0, 1};
const vector<cell> corners = { {0, 0}, {0, 1}, {1, 1}, {1, 0} };
const vector<cell> upper_left = {up, left, up + left}, upper_right = {up, right, up + right}, lower_left = {down, left, down + left}, lower_right = {down, right, down + right};
const vector<char> int_to_char = {'1', '2', '3', '4', '5', '6', '7', '8', '9'};
const board test_constraints = {
{0, -1, -1, -1, -1}, 
{-1, -1, 9, -1, 7},
{8, -1, -1, -1, -1}, 
{-1, -1, 15, -1, 12},
{10, -1, -1, -1, -1}
};
const board constraints = {
{-1, 18, -1, -1, -1, -1, 7, -1, -1},
{-1, -1, -1, -1, 12, -1, -1, -1, -1},
{-1, -1, 9, -1, -1, -1, -1, 31, -1},
{-1, -1, -1, -1, -1, -1, -1, -1, -1},
{-1, 5, -1, 11, -1, 22, -1, 22, -1},
{-1, -1, -1, -1, -1, -1, -1, -1, -1},
{-1, 9, -1, -1, -1, -1, 19, -1, -1},
{-1, -1, -1, -1, 14, -1, -1, -1, -1},
{-1, -1, 22, -1, -1, -1, -1, 15, -1}
};

board create_board(int);
board create_constrained_board(board &);
board copy_board(board const &);
bool two_by_two(board &, cell);
bool adjacent(board &, cell);
bool check_constraints(board &, cell, board &);
bool check_constraint(board &, cell, int);
bool check_config_constraint(board &, cell, board &);
bool connected(board &);
bool connected_incomplete(board &);
int answer(board &);
int unfilled(board &);
vector<int> components(board &, function<bool(char)>);
bool fill_cell(board &, cell, int, board &);
vector<cell> create_hook(cell, cell, cell);
bool fill_hook(board &, vector<cell> &, int, int);
vector<board> solve_constraint(board &, board &, board &, cell, int = 0, int = 0);
bool check_config(board &, board &);
bool forced_board(board &, board &, board &);
vector<int> create_hook_reqs(board &);
bool check_hook_reqs(vector<int> &);
bool solve_config(board &, board &, board &, vector<int> &, int);
bool solve_puzzle(int, board &);
void print_board(board &);
void print_solution(board &);

int main() {
    auto constraints_test = copy_board(test_constraints);
    // auto test_board = create_constrained_board(constraints_test);

    // auto solutions = forced_board(test_board, test_config, constraints_test);
    // std::cout << solutions << std::endl;
    // print_solution(test_board);

    auto solution_test = solve_puzzle(5, constraints_test);

    auto constraints_real = copy_board(constraints);
    auto solution = solve_puzzle(9, constraints_real);

    return 0;
}

// -1 represents an unfilled square, while -2 represents a space
board create_board(int N) {
    board result(N, vector<int>(N, -1));
    return result;
}

board create_constrained_board(board &constraints) {
    board result = copy_board(constraints);
    for (auto &r : result) {
        for (auto &i : r) if (i != -1) i = -2;
    }
    return result;
}

// makes deep copy of board
board copy_board(board const &b) {
    board result;
    for (auto& r : b) result.push_back(vector<int>(r));
    return result;
}

// satisfies the "at least one space in each 2x2 square" constraint
bool two_by_two(board &b, cell pos) {
    int N = b.size();
    // std::cout << "Checking at " << pos.first << ", " << pos.second << std::endl;
    // print_board(b);
    if (pos.first != 0) { // can check up
        if (pos.second != 0) { // can check left
            vector<cell> empty;
            for (auto c : upper_left) {
                c = c + pos;
                // std::cout << "Upper left value: " << b[c.first][c.second] << std::endl;
                if (b[c.first][c.second] == -2 || b[c.first][c.second] == -1) empty.push_back(c);
            }
            // if there's only one empty cell in this block force it to be blank
            // std::cout << "Upper left: " << empty.size() << std::endl;
            if (empty.size() == 0) return false;
        }
        if (pos.second != N - 1) { // can check right
            vector<cell> empty;
            for (auto c : upper_right) {
                c = c + pos;
                // std::cout << "Upper right value: " << b[c.first][c.second] << std::endl;
                if (b[c.first][c.second] == -2 || b[c.first][c.second] == -1) empty.push_back(c);
            }
            // if there's only one empty cell in this block force it to be blank
            // std::cout << "Upper right: " << empty.size() << std::endl;
            if (empty.size() == 0) return false;
        }
    }
    if (pos.first != N - 1) { // can check down
        if (pos.second != 0) { // can check left
            vector<cell> empty;
            for (auto c : lower_left) {
                c = c + pos;
                // std::cout << "Lower left value: " << b[c.first][c.second] << std::endl;
                if (b[c.first][c.second] == -2 || b[c.first][c.second] == -1) empty.push_back(c);
            }
            // if there's only one empty cell in this block force it to be blank
            // std::cout << "Lower left: " << empty.size() << std::endl;
            if (empty.size() == 0) return false;
        }
        if (pos.second != N - 1) { // can check right
            vector<cell> empty;
            for (auto c : lower_right) {
                c = c + pos;
                // std::cout << "Lower right value: " << b[c.first][c.second] << std::endl;
                if (b[c.first][c.second] == -2 || b[c.first][c.second] == -1) empty.push_back(c);
            }
            // if there's only one empty cell in this block force it to be blank
            // std::cout << "Lower right: " << empty.size() << std::endl;
            if (empty.size() == 0) return false;
        }
    }

    return true;
}

// check if cell is adjacent to any others
bool adjacent(board &b, cell pos) {
    int N = b.size();
    if (pos.first != 0) {
        auto c = pos + up;
        if (0 < b[c.first][c.second] && b[c.first][c.second] < 10) return true;
    }
    if (pos.first != N - 1) {
        auto c = pos + down;
        if (0 < b[c.first][c.second] && b[c.first][c.second] < 10) return true;
    }
    if (pos.second != 0) {
        auto c = pos + left;
        if (0 < b[c.first][c.second] && b[c.first][c.second] < 10) return true;
    }
    if (pos.second != N - 1) {
        auto c = pos + right;
        if (0 < b[c.first][c.second] && b[c.first][c.second] < 10) return true;
    }

    return false;
}

bool check_constraints(board &b, cell pos, board &constraints) {
    int N = b.size();
    bool result = true;
    if (pos.first != 0) {
        auto c = pos + up;
        if (constraints[c.first][c.second] != -1) result = result && check_constraint(b, c, constraints[c.first][c.second]);
    }
    if (pos.first != N - 1) {
        auto c = pos + down;
        if (constraints[c.first][c.second] != -1) result = result && check_constraint(b, c, constraints[c.first][c.second]);
    }
    if (pos.second != 0) {
        auto c = pos + left;
        if (constraints[c.first][c.second] != -1) result = result && check_constraint(b, c, constraints[c.first][c.second]);
    }
    if (pos.second != N - 1) {
        auto c = pos + right;
        if (constraints[c.first][c.second] != -1) result = result && check_constraint(b, c, constraints[c.first][c.second]);
    }
    // std::cout << "Point: (" << pos.first << ", " << pos.second << ") result: " << result << std::endl; 

    return result;
}

bool check_constraint(board &b, cell pos, int target) {
    int N = b.size();
    bool all_filled = true;
    if (pos.first != 0) {
        auto c = pos + up;
        if (0 < b[c.first][c.second] && b[c.first][c.second] < 10) target -= b[c.first][c.second];
        else if (b[c.first][c.second] == -2) {}
        else all_filled = false;
    }
    if (pos.first != N - 1) {
        auto c = pos + down;
        if (0 < b[c.first][c.second] && b[c.first][c.second] < 10) target -= b[c.first][c.second];
        else if (b[c.first][c.second] == -2) {}
        else all_filled = false;
    }
    if (pos.second != 0) {
        auto c = pos + left;
        if (0 < b[c.first][c.second] && b[c.first][c.second] < 10) target -= b[c.first][c.second];
        else if (b[c.first][c.second] == -2) {}
        else all_filled = false;
    }
    if (pos.second != N - 1) {
        auto c = pos + right;
        if (0 < b[c.first][c.second] && b[c.first][c.second] < 10) target -= b[c.first][c.second];
        else if (b[c.first][c.second] == -2) {}
        else all_filled = false;
    }

    return all_filled ? target == 0 : target >= 0;
}

bool check_config_constraint(board &config, cell pos, board &constraints) {
    int N = config.size();
    bool all_filled = true;
    int target = constraints[pos.first][pos.second];
    if (pos.first != 0) {
        auto c = pos + up;
        if (0 < config[c.first][c.second] && config[c.first][c.second] < 10) target -= config[c.first][c.second];
        else all_filled = false;
    }
    if (pos.first != N - 1) {
        auto c = pos + down;
        if (0 < config[c.first][c.second] && config[c.first][c.second] < 10) target -= config[c.first][c.second];
        else all_filled = false;
    }
    if (pos.second != 0) {
        auto c = pos + left;
        if (0 < config[c.first][c.second] && config[c.first][c.second] < 10) target -= config[c.first][c.second];
        else all_filled = false;
    }
    if (pos.second != N - 1) {
        auto c = pos + right;
        if (0 < config[c.first][c.second] && config[c.first][c.second] < 10) target -= config[c.first][c.second];
        else all_filled = false;
    }

    if (all_filled) {
        if (target > 0) return false;
        auto b = create_constrained_board(constraints);
        auto partial_solutions = solve_constraint(b, config, constraints, pos);
        return partial_solutions.size() > 0;
    }

    return true;
}

// check if the numbers form a connected region
bool connected(board &b) {
    int N = b.size();
    auto test = components(b, [](int c) {return 0 < c && c < 10;});
    return test.size() == 1 && test[0] == (N * N + N) / 2;
}

bool connected_incomplete(board &b) {
    int N = b.size();
    auto test = components(b, [](int c) {return c != -2;});
    return test.size() == 1;
}

// gets product of empty regions
int answer(board &b) {
    int result = 1;
    auto test = components(b, [](int c) {return c == -2;});
    for (auto& i : test) result *= i;
    return result;
}

// gets how many unfilled cells there are
int unfilled (board &b) {
    int result = 0;
    for (auto &v : b) {
        for (auto &c : v) result += c == -1;
    }
    return result;
}

// checks if filled regions are all orthogonally connected given a certain condition
vector<int> components(board &b, function<bool(char)> criterion) {
    int N = b.size();
    queue<cell> q;
    vector<int> result;
    vector<vector<bool>> visited(N, vector<bool>(N, false));
    for (int i = 0; i < N; ++i) {
        for (int j = 0; j < N; ++j) {
            if (!visited[i][j] && criterion(b[i][j])) {
                int count = 0;
                q.push({i, j});
                visited[i][j] = true;
                while (!q.empty()) {
                    if (q.front().first != 0) {
                        auto c = q.front() + up;
                        if (!visited[c.first][c.second] && criterion(b[c.first][c.second])) {
                            q.push(c);
                            visited[c.first][c.second] = true;
                        }
                    }
                    if (q.front().first != N - 1) {
                        auto c = q.front() + down;
                        if (!visited[c.first][c.second] && criterion(b[c.first][c.second])) {
                            q.push(c);
                            visited[c.first][c.second] = true;
                        }
                    }
                    if (q.front().second != 0) {
                        auto c = q.front() + left;
                        if (!visited[c.first][c.second] && criterion(b[c.first][c.second])) {
                            q.push(c);
                            visited[c.first][c.second] = true;
                        }
                    }
                    if (q.front().second != N - 1) {
                        auto c = q.front() + right;
                        if (!visited[c.first][c.second] && criterion(b[c.first][c.second])) {
                            q.push(c);
                            visited[c.first][c.second] = true;
                        }
                    }
                    ++count;
                    q.pop();
                }
                result.push_back(count);
            }
        }
    }

    return result;
}

// checks all constraints and fills cell if you can
bool fill_cell(board &b, cell pos, int target, board &constraints) {
    // bool result = target == -2 ? true : adjacent(b, pos);
    // std::cout << "Target cell: (" << pos.first << ", " << pos.second << "): " << target << std::endl;
    // std::cout << "Adjacent: " << result << std::endl;
    bool result = true;
    b[pos.first][pos.second] = target;
    // std::cout << "Target cell: (" << pos.first << ", " << pos.second << "): " << target << std::endl;
    // print_solution(b);
    result = result && check_constraints(b, pos, constraints) && (target == -2 ? true : two_by_two(b, pos));
    if (!result) b[pos.first][pos.second] = -1;
    // std::cout << result << std::endl;

    return result;
}

// creates a hook given its three endpoints are in clockwise order
vector<cell> create_hook(cell end1, cell corner, cell end2) {
    vector<cell> result = {end1};
    int size = std::max(abs((corner - end1).first), abs((corner - end1).second));
    if (size == 0) return result;
    cell first_leg = corner - end1;
    first_leg.first /= size, first_leg.second /= size;
    while (result.back() != corner) result.push_back(result.back() + first_leg);
    cell second_leg = end2 - corner;
    second_leg.first /= size, second_leg.second /= size;
    while (result.back() != end2) result.push_back(result.back() + second_leg);

    return result;
}

// checks if hook can feasibly be assigned number
bool fill_hook(board &b, vector<cell> &bounding_box, int dir, int target) {
    auto hook = create_hook(bounding_box[(dir + 3) & 3], bounding_box[dir], bounding_box[(dir + 1) & 3]);
    if (hook.size() < target) return false;
    if (target == 1 && hook.size() != 1) return false;
    if (target == 2 && hook.size() != 3) return false;
    switch (dir) {
        case 0:
            bounding_box[0] = bounding_box[0] + down + right;
            bounding_box[3] = bounding_box[3] + right;
            bounding_box[1] = bounding_box[1] + down;
            break;
        case 1:
            bounding_box[0] = bounding_box[0] + down;
            bounding_box[2] = bounding_box[2] + left;
            bounding_box[1] = bounding_box[1] + down + left;
            break;
        case 2:
            bounding_box[3] = bounding_box[3] + up;
            bounding_box[2] = bounding_box[2] + left + up;
            bounding_box[1] = bounding_box[1] + left;
            break;
        case 3:
            bounding_box[3] = bounding_box[3] + up + right;
            bounding_box[2] = bounding_box[2] + up;
            bounding_box[0] = bounding_box[0] + right;
            break;
        default: 
            std::cout << "How did we get here?" << std::endl;
            break;
    }
    for (auto &pos : hook) b[pos.first][pos.second] = target;

    return true;
}

// check a (potentially in-progress) configuration to see if it violates gcd constraints
bool check_config(board &config, board &constraints) {
    int N = config.size();
    bool result = true;
    for (int i = 0; i < N; ++i) {
        for (int j = 0; j < N; ++j) {
            if (constraints[i][j] != -1) result = result && check_config_constraint(config, {i, j}, constraints);
        }
    }

    return result;
}

// solves a single constraint by backtracking
// order: top, left, right, bottom
vector<board> solve_constraint(board &b, board &config, board &constraints, cell pos, int step, int sum) {
    int N = config.size();
    vector<board> result;
    // print_board(b);
    switch (step) {
        case 0:
            // std::cout << "Top" << std::endl;
            if (pos.first != 0) {
                auto c = pos + up;
                if (constraints[c.first][c.second] == -1) {
                    b[c.first][c.second] = config[c.first][c.second];
                    auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum + b[c.first][c.second]);
                    result.insert(result.end(), partial_solution.begin(), partial_solution.end());
                    b[c.first][c.second] = -1;
                } else std::cout << "How did we get here?" << std::endl;
                b[c.first][c.second] = -2;
                auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum);
                result.insert(result.end(), partial_solution.begin(), partial_solution.end());
                b[c.first][c.second] = -1;
            } else {
                auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum);
                result.insert(result.end(), partial_solution.begin(), partial_solution.end());
            }
            break;
        case 1:
            // std::cout << "Left" << std::endl;
            if (pos.second != 0) {
                auto c = pos + left;
                if (constraints[c.first][c.second] == -1) {
                    b[c.first][c.second] = config[c.first][c.second];
                    auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum + b[c.first][c.second]);
                    result.insert(result.end(), partial_solution.begin(), partial_solution.end());
                    b[c.first][c.second] = -1;
                } else std::cout << "How did we get here?" << std::endl;
                b[c.first][c.second] = -2;
                auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum);
                result.insert(result.end(), partial_solution.begin(), partial_solution.end());
                b[c.first][c.second] = -1;
            } else {
                auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum);
                result.insert(result.end(), partial_solution.begin(), partial_solution.end());
            }
            break;
        case 2:
            // std::cout << "Right" << std::endl;
            if (pos.second != N - 1) {
                auto c = pos + right;
                if (constraints[c.first][c.second] == -1) {
                    b[c.first][c.second] = config[c.first][c.second];
                    auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum + b[c.first][c.second]);
                    result.insert(result.end(), partial_solution.begin(), partial_solution.end());
                    b[c.first][c.second] = -1;
                } else std::cout << "How did we get here?" << std::endl;
                b[c.first][c.second] = -2;
                auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum);
                result.insert(result.end(), partial_solution.begin(), partial_solution.end());
                b[c.first][c.second] = -1;
            } else {
                auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum);
                result.insert(result.end(), partial_solution.begin(), partial_solution.end());
            }
            break;
        case 3:
            // std::cout << "Bottom" << std::endl;
            if (pos.first != N - 1) {
                auto c = pos + down;
                if (constraints[c.first][c.second] == -1) {
                    b[c.first][c.second] = config[c.first][c.second];
                    auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum + b[c.first][c.second]);
                    result.insert(result.end(), partial_solution.begin(), partial_solution.end());
                    b[c.first][c.second] = -1;
                } else std::cout << "How did we get here?" << std::endl;
                b[c.first][c.second] = -2;
                auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum);
                result.insert(result.end(), partial_solution.begin(), partial_solution.end());
                b[c.first][c.second] = -1;
            } else {
                auto partial_solution = solve_constraint(b, config, constraints, pos, step + 1, sum);
                result.insert(result.end(), partial_solution.begin(), partial_solution.end());
            }
            break;
        case 4:
            if (sum == constraints[pos.first][pos.second]) {
                result.push_back(copy_board(b));
                // print_solution(b);
            }
            break;
        default:
            std::cout << "How did we get here?" << std::endl;
            break;
    }
    return result;
}

// gives cells jointly forced by all constraints
// weaker than a "true" forcing since intersections between constraints are ignored
bool forced_board(board &b, board &config, board &constraints) {
    int N = config.size();
    for (int i = 0; i < N; ++i) {
        for (int j = 0; j < N; ++j) {
            if (constraints[i][j] != -1) {
                auto b_temp = create_constrained_board(constraints);
                cell pos = {i, j};
                auto constraint_solution = solve_constraint(b_temp, config, constraints, pos);
                if (constraint_solution.size() == 0) return false;
                if (i != 0) { // can look up
                    auto c = pos + up;
                    int forced_value = constraint_solution[0][c.first][c.second];
                    bool all_same = true;
                    for (auto& sol : constraint_solution) {
                        all_same = all_same && (sol[c.first][c.second] == forced_value);
                    }
                    if (all_same) b[c.first][c.second] = forced_value;
                }
                if (i != N - 1) { // can look down
                    auto c = pos + down;
                    int forced_value = constraint_solution[0][c.first][c.second];
                    bool all_same = true;
                    for (auto& sol : constraint_solution) {
                        all_same = all_same && (sol[c.first][c.second] == forced_value);
                    }
                    if (all_same) b[c.first][c.second] = forced_value;
                }
                if (j != 0) { // can look left
                    auto c = pos + left;
                    int forced_value = constraint_solution[0][c.first][c.second];
                    bool all_same = true;
                    for (auto& sol : constraint_solution) {
                        all_same = all_same && (sol[c.first][c.second] == forced_value);
                    }
                    if (all_same) b[c.first][c.second] = forced_value;
                }
                if (j != N - 1) { // can look right
                    auto c = pos + right;
                    int forced_value = constraint_solution[0][c.first][c.second];
                    bool all_same = true;
                    for (auto& sol : constraint_solution) {
                        all_same = all_same && (sol[c.first][c.second] == forced_value);
                    }
                    if (all_same) b[c.first][c.second] = forced_value;
                }
            }
        }
    }

    return true;
}

// specify how much of each integer needs to be filled in each hook
vector<int> create_hook_reqs(board &b) {
    int N = b.size();
    vector<int> result;
    for (int i = 1; i <= N; ++i) result.push_back(i);
    for (auto &v : b) {
        for (auto &c : v) if (0 < c && c < 10) result[c - 1]--;
    }
    return result;
}

// you can't have more than n of a number n
bool check_hook_reqs(vector<int> &hook_reqs) {
    for (auto &i : hook_reqs) if (i < 0) return false;
    return true;
}

// solve a particular permutation by backtracking
bool solve_config(board &b, board &config, board &constraints, vector<int> &hook_reqs, int steps_left) {
    if (steps_left == 0) {
        if (!connected(b)) return false;
        for (auto &i : hook_reqs) if (i != 0) return false;
        std::cout << "product of areas is " << answer(b) << "\n" << std::endl;
        print_solution(b);
        return true;
    }

    // print_solution(b);
    // std::cout << "Steps left: " << steps_left << std::endl;
    // for (auto& i : hook_reqs) std::cout << i << " ";
    // std::cout << std::endl;

    // if (steps_left == 1) print_solution(b);

    int N = b.size();
    if (accumulate(hook_reqs.begin(), hook_reqs.end(), 0) > steps_left) return false;

    for (int i = 0; i < N; ++i) {
        for (int j = 0; j < N; ++j) {
            if (b[i][j] == -1) {
                if (hook_reqs[config[i][j] - 1] && fill_cell(b, {i, j}, config[i][j], constraints)) {
                    // if (steps_left == 1) std::cout << "Able to fill number" << std::endl;
                    hook_reqs[config[i][j] - 1]--;
                    if (solve_config(b, config, constraints, hook_reqs, steps_left - 1)) return true;
                    hook_reqs[config[i][j] - 1]++;
                    b[i][j] = -1;
                }
                if (fill_cell(b, {i, j}, -2, constraints)) {
                    // if (steps_left == 1) std::cout << "Able to fill space" << std::endl;
                    if (solve_config(b, config, constraints, hook_reqs, steps_left - 1)) return true;
                    b[i][j] = -1;
                }
                b[i][j] = -1;
                return false;
            }
        }
    }

    return false;
}

// solves the entire puzzle
bool solve_puzzle(int N, board &constraints) {
    auto start = steady_clock::now();

    stack<tuple<board, vector<cell>, int, vector<bool>>> s;
    s.push(make_tuple(create_board(N), vector<cell> { {0, 0}, {0, N - 1}, {N - 1, N - 1}, {N - 1, 0} }, N, vector<bool>(N, true)));
    int valid_boards = 0;

    while (!s.empty()) {
        auto curr = s.top();
        s.pop();
        for (int dir = 0; dir < 4; ++dir) {
            for (int i = 0; i < N; ++i) {
                if (get<3>(curr)[i]) {
                    auto new_board = copy_board(get<0>(curr));
                    auto new_bounding_box = vector<cell>(get<1>(curr));
                    if (fill_hook(new_board, new_bounding_box, dir, i + 1)) {
                        if (check_config(new_board, constraints)) {
                            if (get<2>(curr) == 1) {
                                ++valid_boards;
                                auto b = create_constrained_board(constraints);
                                if (forced_board(b, new_board, constraints)) {
                                    if (connected_incomplete(b)) {
                                        auto hook_reqs = create_hook_reqs(b);
                                        if (check_hook_reqs(hook_reqs)) {
                                            // print_solution(b);
                                            auto cells_left = unfilled(b);
                                            if (solve_config(b, new_board, constraints, hook_reqs, cells_left)) {
                                                std::cout << "solution found after " << duration_cast<milliseconds>(steady_clock::now() - start).count() / 1000.0 << " seconds" << std::endl;
                                                return true;
                                            }
                                        }
                                    }
                                }
                            } else {
                                auto new_perm = vector<bool>(get<3>(curr));
                                new_perm[i] = false;
                                s.push(make_tuple(new_board, new_bounding_box, get<2>(curr) - 1, new_perm));
                            }
                        }
                    }
                }
            }
        }
    }

    return false;
}

// debug print
void print_board(board &b) {
    for (auto& r : b) {
        for (auto &c : r) std::cout << c << " ";
        std::cout << std::endl;
    }
    std::cout << std::endl;
}

// it's easier to see spaces in the solution
void print_solution(board &b) {
    for (auto& r : b) {
        for (auto &c : r) std::cout << (c == -2 ? '@' : (char) (c + '0')) << " ";
        std::cout << std::endl;
    }
    std::cout << std::endl;
}
```