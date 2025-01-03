+++
title = "Advent of Code 2021"
date = 2021-12-01
draft = false

[taxonomies]
categories = ["Programming"]
tags = ["Python"]
+++

This year I will do [Advent of Code 2021](https://adventofcode.com/). Considering my programming skills, Python will be used. This article will record simple descriptions of the problems, some of my solution and little notes, which should be updated everyday between December 1st and December 25th. The complete code and data be available on [GitHub](https://github.com/xiongdong57/AdventofCode2021).
<!-- more -->

## utils.py
```Python
def parse_data(day: int, parser=str, sep='\n'):
    with open(f'input/day{day:02d}.txt') as f:
        return [parser(line) for line in f.read().split(sep)]


def convert_to_int(data: List):
    return [int(elem) for elem in data]


def print_map(system, fillin='.'):
    # print the {(x, y): val} map
    x_max = max(system, key=lambda x: x[0])[0]
    x_min = min(system, key=lambda x: x[0])[0]
    y_max = max(system, key=lambda x: x[1])[1]
    y_min = min(system, key=lambda x: x[1])[1]
    for dy in range(y_min, y_max + 1):
        for dx in range(x_min, x_max + 1):
            print(system.get((dx, dy), fillin), end='')
        print()
    print()
```

## [Day 1: Sonar Sweep](https://adventofcode.com/2021/day/1)
You had the following report:  
199  
200  
208  
210  
200  
207  
240  
269  
260  
263  

1. How many measurements are larger than the previous measurement?
2. Consider sums of a three-measurement sliding window. How many sums are larger than the previous sum?

```Python
def day01_1(data):
    increase_count = 0
    for i in range(1, len(data)):
        if data[i] > data[i - 1]:
            increase_count += 1
    return increase_count


def day01_2(data):
    data_with_3_window = [sum(data[i:i+3]) for i in range(len(data) - 2)]
    return day01_1(data_with_3_window)
```

## [Day 2: Dive!](https://adventofcode.com/2021/day/2)

Input be like:   
forward 5  
down 5  
forward 8  
up 3  
down 8  
forward 2  

part1 with rule:
- forward X increases the horizontal position by X units.
- down X increases the depth by X units.
- up X decreases the depth by X units.

part2 with rule:
- down X increases your aim by X units.
- up X decreases your aim by X units.
- forward X does two things:
  - It increases your horizontal position by X units.
  - It increases your depth by your aim multiplied by X.

```Python
def day02_1(data):
    horizon = 0
    depth = 0
    for action, num in data:
        if action == 'forward':
            horizon += int(num)
        elif action == 'down':
            depth += int(num)
        elif action == 'up':
            depth -= int(num)
        else:
            raise ValueError('Invalid action: {}'.format(action))
    return abs(horizon) * abs(depth)


def day02_2(data):
    aim = 0
    horizon = 0
    depth = 0
    for action, num in data:
        if action == 'down':
            aim += int(num)
        elif action == 'up':
            aim -= int(num)
        elif action == 'forward':
            horizon += int(num)
            depth += aim * int(num)
        else:
            raise ValueError('Invalid action: {}'.format(action))
    return abs(horizon) * abs(depth)

data = parse_data(day=2, parser=lambda x: x.split(' '))
```

## [Day 3: Binary Diagnostic](https://adventofcode.com/2021/day/3)

Input be like:
00100  
11110  
10110  
10111  
10101  
01111  
00111  
11100  
10000  
11001  
00010  
01010  

part1 with rule: 
- gamma rate can be determined by finding the most common bit in the corresponding position of all numbers in the diagnostic report
- epsilon rate is calculated in a similar way; rather than use the most common bit, the least common bit from each position is used
- What is the power consumption of the submarine(gamma rate multiplied by epsilon rate)

part2 with rule:
- Keep only numbers selected by the bit criteria for the type of rating value for which you are searching. Discard numbers which do not match the bit criteria.
- If you only have one number left, stop; this is the rating value for which you are searching. Otherwise, repeat the process, considering the next bit to the right.

The bit criteria depends on which type of rating value you want to find:
- To find oxygen generator rating, determine the most common value (0 or 1) in the current bit position, and keep only numbers with that bit in that position. If 0 and 1 are equally common, keep values with a 1 in the position being considered.
- To find CO2 scrubber rating, determine the least common value (0 or 1) in the current bit position, and keep only numbers with that bit in that position. If 0 and 1 are equally common, keep values with a 0 in the position being considered.

 What is the life support rating of the submarine(oxygen generator rating multiplied by CO2 scrubber rating)

```Python
def common_bit(lst, descending=False):
    one_count = lst.count('1')
    zero_count = lst.count('0')
    if descending:
        return '1' if one_count >= zero_count else '0'
    else:
        return '0' if one_count >= zero_count else '1'


def day03_1(data):
    gamma_rate_str = ''
    epsilon_rate_str = ''
    for i in range(len(data[0])):
        bits = [elem[i] for elem in data]
        gamma_rate_str += common_bit(bits, descending=True)
        epsilon_rate_str += common_bit(bits, descending=False)

    return int(gamma_rate_str, 2) * int(epsilon_rate_str, 2)


def gen_bit_criteria(data, descending=True):
    filtered_data = data[:]
    for i in range(len(data[0])):
        bits = [elem[i] for elem in filtered_data]
        filter_bit = common_bit(bits, descending=descending)
        filtered_data = [
            elem for elem in filtered_data if elem[i] == filter_bit]
        if len(filtered_data) == 1:
            return filtered_data[0]


def day03_2(data):
    oxygen_generator_rate_str = gen_bit_criteria(data, descending=True)
    CO2_scrubber_rate_str = gen_bit_criteria(data, descending=False)
    return int(oxygen_generator_rate_str, 2) * int(CO2_scrubber_rate_str, 2)

data = parse_data(day=3, parser=str)
```

## [Day 4: Giant Squid](https://adventofcode.com/2021/day/4)

Bingo is played on a set of boards each consisting of a 5x5 grid of numbers. Numbers are chosen at random, and the chosen number is marked on all boards on which it appears. (Numbers may not appear on all boards.) If all numbers in any row or any column of a board are marked, that board wins.

Input be like:  
7,4,9,5,11,17,23,2,0,14,21,24,10,16,13,6,15,25,12,22,18,20,8,19,3,26,1

22 13 17 11  0  
 8  2 23  4 24  
21  9 14 16  7  
 6 10  3 18  5  
 1 12 20 15 19  

 3 15  0  2 22  
 9 18 13 17  5  
19  8  7 25 23  
20 11 10 24  4  
14 21 16 12  6  

14 21 17 24  4  
10 16 15  9 19  
18  8 23 26 20  
22 11 13  6  5  
 2  0 12  3  7  

score is the sum of all unmarked numbers multiply that sum by the number that was just called when the board won.

part1:  What will your final score be if you choose that board
part2:  figure out which board will win last and choose that one, figure out which board will win last and choose that one

```Python
def convert_to_int(data: List):
    return [int(elem) for elem in data]


def parse_input():
    data = parse_data(day=4, parser=str.splitlines, sep='\n\n')
    nums = convert_to_int(data[0][0].split(','))
    boards = [[convert_to_int(line.split()) for line in board]
              for board in data[1:]]
    return nums, boards


def board_marks_valid(board: List[List], marked_nums: set):
    cols = [set([row[i] for row in board])
            for i in range(len(board[0]))]
    rows = [set(row) for row in board]
    lines = cols + rows
    return any(line.issubset(marked_nums) for line in lines)


def day04_1(nums, boards):
    for i, num in enumerate(nums):
        for board in boards:
            marked_nums = set(nums[:i+1])
            if board_marks_valid(board, marked_nums):
                all_unmark_nums_sum = sum([elem
                                           for row in board
                                           for elem in row
                                           if elem not in marked_nums])
                return num * all_unmark_nums_sum


def day04_2(nums, boards):
    remain_boards = boards[:]
    for i, num in enumerate(nums):
        marked_nums = set(nums[:i+1])
        remain_boards = [board
                         for board in remain_boards
                         if not board_marks_valid(board, marked_nums)]
        if len(remain_boards) == 1:
            last_board = remain_boards[0]
        if len(remain_boards) == 0:
            all_unmark_nums_sum = sum([elem
                                       for row in last_board
                                       for elem in row
                                       if elem not in marked_nums])
            return num * all_unmark_nums_sum
```

If this puzzle is more complex, maybe numpy is a good way to go.

## [Day 5: Hydrothermal Venture](https://adventofcode.com/2021/day/5)
For lines:  
0,9 -> 5,9  
8,0 -> 0,8  
9,4 -> 3,4  
2,2 -> 2,1  
7,0 -> 7,4  
6,4 -> 2,0  
0,9 -> 2,9  
3,4 -> 1,4  
0,0 -> 8,8  
5,5 -> 8,2  

you need to determine the number of points where at least two lines overlap.

part1: Consider only horizontal and vertical lines. At how many points do at least two lines overlap?

part2: Because of the limits of the hydrothermal vent mapping system, the lines in your list will only ever be horizontal, vertical, or a diagonal line at exactly 45 degrees.Consider all of the lines. How many points do at least two lines overlap?

```Python
def gen_line_points(x1, y1, x2, y2, diagonal_line=False):
    points = []
    x_min = min(x1, x2)
    x_max = max(x1, x2)
    y_min = min(y1, y2)
    y_max = max(y1, y2)
    if x1 == x2:
        for y in range(y_min, y_max + 1):
            points.append((x1, y))
    elif y1 == y2:
        for x in range(x_min, x_max + 1):
            points.append((x, y1))
    elif diagonal_line:
        ascending = ((x1 == x_min and y1 == y_min) or
                     (x2 == x_min and y2 == y_min))
        for x in range(x_min, x_max + 1):
            if ascending:
                points.append((x, y_min + (x - x_min)))
            else:
                points.append((x, y_max - (x - x_min)))
    return points


def solve(data, diagnoal_line):
    picture = defaultdict(int)
    for line in data:
        x1, y1, x2, y2 = [int(elem) for elem in line]
        for point in gen_line_points(x1, y1, x2, y2,
                                     diagonal_line=diagnoal_line):
            picture[point] += 1
    return sum(1 for value in picture.values() if value > 1)


def day05_1(data):
    return solve(data, diagnoal_line=False)


def day05_2(data):
    return solve(data, diagnoal_line=True)


data = parse_data(day=5,
                  parser=lambda x: re.findall(
                    r'(\d+),(\d+) -> (\d+),(\d+)', x)[0])
```

Data structure is really a key thing to solve the problem with simple and clean solution.

## [Day 6: Lanternfish](https://adventofcode.com/2021/day/6)

Each lanternfish creates a new lanternfish once every 7 days and a new lanternfish need slightly longer before it's capable of producing more lanternfish: two more days for its first cycle. We model each fish as a single number that represents the number of days until it creates a new lanternfish.

Input be like:  
3,4,3,1,2

part1: How many lanternfish would there be after 80 days?  
part2: ow many lanternfish would there be after 256 days

```Python
def simulate(states: List):
    new_gen_state = states.count(0) * [6, 8]
    updated_states = [elem - 1 for elem in states if elem != 0]
    return updated_states + new_gen_state


def simulate_2(state: defaultdict):
    new_state = defaultdict(int)
    for key, value in state.items():
        if key == 0:
            new_state[6] += value
            new_state[8] += value
        else:
            new_state[key - 1] += value
    return new_state


def day06_1(data):
    state = data[:]
    for _ in range(80):
        state = simulate(state)
    return len(state)


def day06_2(data):
    state = defaultdict(int)
    for elem in data:
        if elem not in state.keys():
            state[elem] = data.count(elem)
    for _ in range(256):
        state = simulate_2(state)
    return sum(state.values())

data = parse_data(day=6, parser=lambda x: x.split(','))[0]
data = convert_to_int(data)
```

Again, data structure is the key to the right solution.

## [Day 7: The Treachery of Whales](https://adventofcode.com/2021/day/7)

List of the horizontal position of each crab:  
16,1,2,0,4,2,7,1,2,14

part1 with rule:   
Each change of 1 step in horizontal position of a single crab costs 1 fuel. Determine the horizontal position that the crabs can align to using the least fuel possible. 

How much fuel must they spend to align to that position?

part2 with rule:  
ach change of 1 step in horizontal position costs 1 more unit of fuel than the last: the first step costs 1, the second step costs 2, the third step costs 3, and so on

How much fuel must they spend to align to that position?

```Python
def fuel_cost(seq, pos):
    return sum(abs(elem - pos) for elem in seq)


def day07_1(data):
    pos_min = min(data)
    pos_max = max(data)
    return min(fuel_cost(data, pos) for pos in range(pos_min, pos_max + 1))


def fuel_cost_v2(seq, pos):
    # basic math: 1 + 2 + ... + n = n(n+1)/2
    fuels = 0
    for elem in seq:
        distance = abs(elem - pos)
        fuels += int(distance * (distance + 1) / 2)
    return fuels


def day07_2(data):
    pos_min = min(data)
    pos_max = max(data)
    return min(fuel_cost_v2(data, pos) for pos in range(pos_min, pos_max + 1))

data = parse_data(day=7, parser=lambda x: x.split(','))[0]
data = convert_to_int(data)
```

## [Day 8: Seven Segment Search](https://adventofcode.com/2021/day/8)

Input be like:  
acedgfb cdfbe gcdfa fbcad dab cefabd cdfgeb eafb cagedb ab | cdfeb fcadb cdfeb cdbaf

before | is the 0-9 numbers, after | can be translate to digits.

Code be like:
```text
  0:      1:      2:      3:      4:  
 aaaa    ....    aaaa    aaaa    ....  
b    c  .    c  .    c  .    c  b    c  
b    c  .    c  .    c  .    c  b    c  
 ....    ....    dddd    dddd    dddd  
e    f  .    f  e    .  .    f  .    f  
e    f  .    f  e    .  .    f  .    f  
 gggg    ....    gggg    gggg    ....  
  
  5:      6:      7:      8:      9:  
 aaaa    aaaa    aaaa    aaaa    aaaa  
b    .  b    .  .    c  b    c  b    c  
b    .  b    .  .    c  b    c  b    c  
 dddd    dddd    ....    dddd    dddd  
.    f  e    f  .    f  e    f  .    f  
.    f  e    f  .    f  e    f  .    f  
 gggg    gggg    ....    gggg    gggg  
```
part1: In the output values, how many times do digits 1, 4, 7, or 8 appear?(hints: 1, 4, 7, 8 use a unique number of segments)

part2: For each entry, determine all of the wire/segment connections and decode the four-digit output values. What do you get if you add up all of the output values?

```Python
def gen_data():
    patterns = parse_data(day=8, parser=lambda x: x.split(' | ')[0])
    digits = parse_data(day=8, parser=lambda x: x.split(' | ')[1])
    patterns = [pattern.split() for pattern in patterns]
    digits = [digit.split() for digit in digits]
    return patterns, digits


def day08_1(patterns, digits):
    segments_map = {
        1: 2,
        4: 4,
        7: 3,
        8: 7
    }
    only_digits_counts = 0
    for line in digits:
        for digit in line:
            if len(digit) in segments_map.values():
                only_digits_counts += 1
    return only_digits_counts


def translate_to_chars(translator, chars: str):
    traslate_map = str.maketrans(''.join(translator), 'abcdefg')
    return chars.translate(traslate_map)


def translate_to_num(chars: str):
    char_to_num = {
        'abcefg': '0',
        'cf': '1',
        'acdeg': '2',
        'acdfg': '3',
        'bcdf': '4',
        'abdfg': '5',
        'abdefg': '6',
        'acf': '7',
        'abcdefg': '8',
        'abcdfg': '9'
    }
    key = ''.join(sorted(chars))
    return char_to_num.get(key, '')


def origin_chars_to_num(origin_chars, translator):
    chars = translate_to_chars(translator, origin_chars)
    return translate_to_num(chars)


def valid(pattern, translator):
    nums = [origin_chars_to_num(ch, translator) for ch in pattern]
    mark = ''.join(sorted(nums))
    return mark == '0123456789'


def solve_tranlator(pattern):
    # brute force, may be some clever dfs also can solve this
    for translator in permutations('abcdefg', 7):
        if valid(pattern, translator):
            return translator


def day08_2(patterns, digits):
    all_sum = 0
    for pattern, digit_nums in zip(patterns, digits):
        translator = solve_tranlator(pattern)
        num = ''
        for digit in digit_nums:
            num += origin_chars_to_num(digit, translator)
        all_sum += int(num)
    return all_sum


patterns, digits = gen_data()
```

Part1 is easy, after careful understanding the matiral. Part2 first comes to DFS, then I find it's hard to mapping the iterate rule. Then when broswing reddit, someone mentioned brute force. May be  I should calc how many possible combinations and the result number is quit small(less than 10000). Finally, using brute force to solve it.


## [Day 9: Smoke Basin](https://adventofcode.com/2021/day/9)

consider the following heightmap:  
2199943210  
3987894921  
9856789892  
8767896789  
9899965678  

Your first goal is to find the low points - the locations that are lower than any of its adjacent locations.Most locations have four adjacent locations (up, down, left, and right). The risk level of a low point is 1 plus its height.  

Part1: What is the sum of the risk levels of all low points on your heightmap?

A basin is all locations that eventually flow downward to a single low point. Therefore, every low point has a basin, although some basins are very small. Locations of height 9 do not count as being in any basin, and all other locations will always be part of exactly one basin.

The size of a basin is the number of locations within the basin, including the low point.

Part2: What do you get if you multiply together the sizes of the three largest basins?

```Python
def is_lowest_adjacent(height_map, loc):
    x, y = loc
    height = height_map[loc]
    # only consider up, down, left and right
    directions = [(0, 1), (0, -1), (1, 0), (-1, 0)]
    return all(height_map.get((x+dx, y+dy), 9) > height
               for dx, dy in directions)


def make_map(data):
    height_map = defaultdict(int)
    for row in range(len(data)):
        for col in range(len(data[row])):
            height_map[(row, col)] = int(data[row][col])
    return height_map


def day09_1(data):
    height_map = make_map(data)
    risk_level = 0
    for loc, height in height_map.items():
        if is_lowest_adjacent(height_map, loc):
            risk_level += height + 1
    return risk_level


def visit_basins(height_map, loc, visited=[]):
    # BFS
    visited.append(loc)
    x, y = loc
    directions = [(0, 1), (0, -1), (1, 0), (-1, 0)]
    for dx, dy in directions:
        new_loc = (x+dx, y+dy)
        if (new_loc not in visited and
           new_loc in height_map and
           height_map.get(new_loc) < 9):
            visit_basins(height_map, new_loc, visited)
    return len(visited)


def day09_2(data):
    height_map = make_map(data)
    basins = []
    for loc in height_map.keys():
        if is_lowest_adjacent(height_map, loc):
            basins.append(visit_basins(height_map, loc, []))
    basins = sorted(basins, reverse=True)
    return basins[0] * basins[1] * basins[2]
```

Today is a simple useage of BFS.


## [Day 10: Syntax Scoring](https://adventofcode.com/2021/day/10)

From today, I will not decribe the puzzle, the complete description will be linked.

- What is the total syntax error score for those errors?
- Find the completion string for each incomplete line, score the completion strings, and sort the scores. What is the middle score

```Python
def reduce_chunk(chunk: str):
    if any(ch in chunk for ch in ['()', '[]', '{}', '<>']):
        new_chunk = (chunk
                     .replace('()', '')
                     .replace('[]', '')
                     .replace('{}', '')
                     .replace('<>', ''))
        return reduce_chunk(new_chunk)
    else:
        return chunk


def day10_1(data):
    score = 0
    score_map = {')': 3, ']': 57, '}': 1197, '>': 25137}
    for chunk in data:
        remain_chunk = reduce_chunk(chunk)
        corupted_chars = [ch
                          for ch in remain_chunk
                          if ch in [')', ']', '}', '>']]
        if corupted_chars:
            score += score_map[corupted_chars[0]]
    return score


def reverse_chunk(chunk):
    # reverse the incomplete chunk, will complete the chunk
    # such as: ([{{ and }}])
    new_chunk = chunk[::-1]
    chunk_map = {'(': ')', '[': ']', '{': '}', '<': '>'}
    return ''.join([chunk_map[ch] for ch in new_chunk])


def day10_2(data):
    scores = []
    score_map = {')': 1, ']': 2, '}': 3, '>': 4}
    for chunk in data:
        remain_chunk = reduce_chunk(chunk)
        corupted_chars = [ch
                          for ch in remain_chunk
                          if ch in [')', ']', '}', '>']]
        if not corupted_chars:
            score = 0
            for ch in reverse_chunk(remain_chunk):
                score = score * 5 + score_map[ch]
            scores.append(score)
    middle_index = int(len(scores) / 2)
    return sorted(scores)[middle_index]
```

A simple usage of recursion. Another approach is to use stack, if the c is open_close, put it into stack, if the c is close_open, pop the stack, after line is completed, the stack should be reduced to corupted and incomplete. And you can here to return the flag of the corupted and incomplete.

## [Day 11: Dumbo Octopus](https://adventofcode.com/2021/day/11)

Input be like a 10*10 number map, represent a enery map. After each iteration, the num changes follow the rule:
- First, the energy level of each octopus increases by 1.
- Then, any octopus with an energy level greater than 9 flashes. This increases the energy level of all adjacent octopuses by 1, including octopuses that are diagonally adjacent. If this causes an octopus to have an energy level greater than 9, it also flashes. This process continues as long as new octopuses keep having their energy level increased beyond 9. (An octopus can only flash at most once per step.)
- Finally, any octopus that flashed during this step has its energy level set to 0, as it used all of its energy to flash.

```Python
def make_map(data):
    energy_map = defaultdict(int)
    for row in range(len(data)):
        for col in range(len(data[row])):
            energy_map[(row, col)] = int(data[row][col])
    return energy_map


def get_neighbors(loc, energy_map):
    x, y = loc
    directions = [(0, 1), (1, 1), (1, 0), (1, -1),
                  (0, -1), (-1, -1), (-1, 0), (-1, 1)]
    return [(x + dx, y + dy)
            for dx, dy in directions
            if (x + dx, y + dy) in energy_map]


def update_map(energy_map, flashed=[]):
    energy_map = energy_map.copy()
    for loc, energy in energy_map.items():
        if energy > 9 and loc not in flashed:
            for neighbor in get_neighbors(loc, energy_map):
                if neighbor not in flashed:
                    energy_map[neighbor] += 1
            flashed.append(loc)
    if all(eneryg <= 9 for loc, eneryg in energy_map.items()
       if loc not in flashed):
        return energy_map
    else:
        return update_map(energy_map, flashed)


def simulate(energy_map):
    flashes = 0
    energy_map = energy_map.copy()
    for loc, enery in energy_map.items():
        energy_map[loc] = enery + 1
    energy_map = update_map(energy_map, [])
    for loc, energy in energy_map.items():
        if energy > 9:
            energy_map[loc] = 0
            flashes += 1
    return flashes, energy_map


def day11_1(data):
    energy_map = make_map(data)
    all_flashes = 0
    for _ in range(100):
        flashes, energy_map = simulate(energy_map)
        all_flashes += flashes
    return all_flashes


def day11_2(data):
    energy_map = make_map(data)
    counter = 0
    while True:
        counter += 1
        _, energy_map = simulate(energy_map)
        if all(energy == 0 for energy in energy_map.values()):
            return counter
```

For part one, I made a mistake, I think the update_map will not be recursive at first, and take some time and steps to reliaze it. After fix this, the solution is quite simple.

For part two, I first thought it will be some large number, but it is not. So the solution not require much effort.

Maybe next yeaer, I should build a Grid class be represent the map.

When browsing the internet, I found [programmer named Jocelyn Stericker using rust to solve Advent of Code 2021](https://www.youtube.com/c/JocelynStericker) and [Peter Norvig using python to solve Advent of Code 2021](https://github.com/norvig/pytudes/blob/main/ipynb/Advent-2021.ipynb). Usually, I will solve the puzzle on my own first, then see their solution, which is often shorter and clever.

## [Day 12: Passage Pathing](https://adventofcode.com/2021/day/12)

Input be like:  
start-A  
start-b  
A-c  
A-b  
b-d  
A-end  
b-end  

- big caves (written in uppercase, like A) and small caves (written in lowercase, like b) .all paths you find should visit small caves at most once, and can visit big caves any number of times.
- part1: How many paths through this cave system are there that visit small caves at most once
- part2: After reviewing the available paths, you realize you might have time to visit a single small cave **twice**. However, the caves named start and end can only be visited exactly once each. Given these new rules, how many paths through this cave system are there?

```Python
def gen_map(data):
    system = defaultdict(list)
    for start, end in data:
        system[start].append(end)
        system[end].append(start)
    return system


def path_visit(start='start', end='end', system=[], gen_node=None):
    frontier = [[start]]
    while frontier:
        path = frontier.pop(0)
        node = path[-1]
        if node == end:
            yield path
        for next_node in gen_node(node, system, path):
            path2 = path + [next_node]
            frontier.append(path2)


def successors(node, system, path):
    return [node
            for node in system[node]
            if not(node in path and ('a' <= node[0] <= 'z'))]


def successors_v2(node, system, path):
    return [node
            for node in system[node]
            if not(
                (path + [node]).count('start') > 1 or
                (path + [node]).count('end') > 1 or
                (node in path and
                 'a' <= node[0] <= 'z' and
                 is_biger_than_twice(path + [node]))
            )]


def is_biger_than_twice(path):
    smalls = [node
              for node in path
              if 'a' <= node[0] <= 'z']
    return len(smalls) - len(set(smalls)) > 1


def day12_1(data):
    system = gen_map(data)
    paths = list(path_visit(start='start',
                            end='end',
                            system=system,
                            gen_node=successors))
    return len(paths)


def day12_2(data):
    system = gen_map(data)
    paths = list(path_visit(start='start',
                            end='end',
                            system=system,
                            gen_node=successors_v2))
    return len(paths)
```

## [Day 13: Transparent Origami](https://adventofcode.com/2021/day/13)

Input be like:  

6,10  
0,14  
9,10  
0,3  
10,4  
4,11  
6,0  
6,12  
4,1  
0,13  
10,12  
3,4  
3,0  
8,4  
1,10  
2,14  
8,10  
9,0  
  
fold along y=7  
fold along x=5  

- How many dots are visible after completing just the first fold instruction on your transparent paper
- Finish folding the transparent paper according to the instructions. The manual says the code is always eight capital letters. What is the code?

```Python
def gen_data():
    data = parse_data(day=13, parser=str, sep='\n\n')
    dots = [line.split(',') for line in data[0].split('\n')]
    folds = [re.findall(r'.*?(\w)=(\d+)', line)[0]
             for line in data[1].split('\n')]

    return dots, folds


def make_map(dots):
    system = defaultdict()
    for x, y in dots:
        system[(int(x), int(y))] = '#'
    return system


def update_dot(dota, dotb):
    return '#' if '#' in [dota, dotb] else '.'


def update_fold_system(system, axis, num):
    new_system = defaultdict()
    for (x, y), dot in system.items():
        if axis == 'x':
            if x <= num:
                new_system[(x, y)] = dot
            else:
                new_system[(2*num - x, y)] = update_dot(system[(x, y)], dot)
        if axis == 'y':
            if y <= num:
                new_system[(x, y)] = dot
            else:
                new_system[(x, 2*num - y)] = update_dot(system[(x, y)], dot)
    return new_system


def day13_1(dots, folds):
    system = make_map(dots)
    for axis, num in folds[:1]:
        system = update_fold_system(system, axis, int(num))
    return list(system.values()).count('#')


def print_map(system):
    x = max(system, key=lambda x: x[0])[0]
    y = max(system, key=lambda x: x[1])[1]
    for dy in range(y + 1):
        for dx in range(x + 1):
            print(system.get((dx, dy), ' '), end='')
        print()
    print()


def day13_2(dots, folds):
    system = make_map(dots)
    for axis, num in folds:
        system = update_fold_system(system, axis, int(num))
    # ZKAUCFUC
    print_map(system)
```

## [Day 14: Extended Polymerization](https://adventofcode.com/2021/day/14)

Input is two part, first is the polymer template, second is a list of pair insertion rules.

NNCB  
  
CH -> B  
HH -> N  
CB -> H  
NH -> C  
HB -> C  
HC -> B  
HN -> C  
NN -> C  
BH -> H  
NC -> B  
NB -> B  
BN -> B  
BB -> N  
BC -> B  
CC -> N  
CN -> C  

A rule like AB -> C means that when elements A and B are immediately adjacent, element C should be inserted between them. These insertions all happen simultaneously.Note that these pairs overlap: the second element of one pair is the first element of the next pair. Also, because all pairs are considered simultaneously, inserted elements are not considered to be part of a pair until the next step.

- part1: after 10 steps, what is quantity of the most common element and subtract the quantity of the least common element
- part2: after 400 steps, what is quantity of the most common element and subtract the quantity of the least common element

```Python
def gen_data():
    pair_insertion_ruls = defaultdict()
    data = parse_data(day=14, sep='\n\n')
    template = data[0]
    for line in data[1].split('\n'):
        key, val = line.split(' -> ')
        pair_insertion_ruls[key] = val
    return template, pair_insertion_ruls


def simulate(template, pair_insertion_ruls):
    new_template = ''
    for i in range(len(template) - 1):
        left_char = template[i]
        right_char = template[i + 1]
        insert_char = pair_insertion_ruls[left_char + right_char]
        new_template += left_char + insert_char
    new_template += template[-1]
    return new_template


def day14_1(template, pair_insertion_ruls):
    for _ in range(10):
        template = simulate(template, pair_insertion_ruls)

    counter = Counter(template)
    most_common = counter.most_common(1)[0][1]
    least_common = counter.most_common()[-1][1]
    return most_common - least_common


def day14_2(template, pair_insertion_ruls):
    symbols_occurences = defaultdict(int)
    for x in template:
        symbols_occurences[x] += 1

    twograms_occurences = defaultdict(int)
    for i in range(len(template) - 1):
        x = template[i]
        y = template[i + 1]
        twograms_occurences[x + y] += 1

    for _ in range(40):
        new_insertions = []
        for pair, symbol in pair_insertion_ruls.items():
            if pair in twograms_occurences:
                new_insertions.append(
                    (pair, symbol, twograms_occurences[pair]))
                del twograms_occurences[pair]
        for pair, symbol, cnt in new_insertions:
            symbols_occurences[symbol] += cnt
            twograms_occurences[pair[0] + symbol] += cnt
            twograms_occurences[symbol + pair[1]] += cnt
    return max(symbols_occurences.values()) - min(symbols_occurences.values())
```

The idea to solve part 1 is to simulate and store the template, which works fine. But for part 2 the length of final template is more than 2 trillion. There is no data strutures to store it, so a different approach is needed.

Part 2 is mostly reference some great post on this [subreddit](https://www.reddit.com/r/adventofcode/).

## [Day 15: Chiton](https://adventofcode.com/2021/day/15)

Input is a 2D array of integers, representing the risk map. Such as:
1163751742  
1381373672  
2136511328  
3694931569  
7463417111  
1319128137  
1359912421  
3125421639  
1293138521  
2311944581  

Part1: What is the lowest total risk of any path from the top left to the bottom right?

Part2: The actual map is the current map repeat to right or down five times. And after each repeation, the risk are 1 higher, but risk levels above 9 wrap back around to 1. With this new information, what is the lowest total risk of any path from the top left to the bottom right?

```Python
from queue import PriorityQueue


def make_map(data):
    return {(row, col): int(data[row][col])
            for row in range(len(data))
            for col in range(len(data[row]))}


def print_map(energy_map):
    loc = list(energy_map.keys())[-1]
    x, y = loc
    for row in range(x + 1):
        for col in range(y + 1):
            print(energy_map[(row, col)], end='')
        print()
    print()


class Graph:
    def __init__(self, num_of_vertices):
        self.v = num_of_vertices*num_of_vertices
        self.edges = defaultdict()
        self.nodes = [(i, j)
                      for i in range(num_of_vertices)
                      for j in range(num_of_vertices)]

    def add_edge(self, u, v, weight):
        if u in self.nodes and v in self.nodes:
            self.edges[(u, v)] = weight

    def add_edge_with_map(self, risk_map):
        for node, risk in risk_map.items():
            for neighbor in self.neighbors(node):
                if neighbor in risk_map:
                    self.edges[(neighbor, node)] = risk

    def neighbors(self, loc):
        x, y = loc
        directions = [(0, 1), (1, 0), (0, -1), (-1, 0)]
        return [(x + dx, y + dy) for dx, dy in directions]


def dijkstra(graph, source):
    dist = {node: float('inf') for node in graph.nodes}
    prev = {node: None for node in graph.nodes}
    dist[source] = 0
    q = PriorityQueue()
    q.put((0, source))
    while not q.empty():
        u = q.get()[1]
        for v in graph.neighbors(u):
            if ((u, v) in graph.edges and
               dist[v] > dist[u] + graph.edges[(u, v)]):
                dist[v] = dist[u] + graph.edges[(u, v)]
                q.put((dist[v], v))
                prev[v] = u
    return dist, prev


def print_result(previous_nodes, shortest_path, start_node, target_node):
    path = []
    node = target_node

    while node != start_node:
        path.append(node)
        node = previous_nodes[node]

    # Add the start node manually
    path.append(start_node)

    print("We found the following best path with a value of {}.".format(
        shortest_path[target_node]))
    print(" -> ".join(str(elem) for elem in reversed(path)))


def solver(risk_map):
    last_element = list(risk_map.keys())[-1]

    graph = Graph(last_element[0] + 1)
    graph.add_edge_with_map(risk_map)

    source = list(risk_map.keys())[0]

    dist, _ = dijkstra(graph, source)
    return dist[last_element]


def day15_1(data):
    risk_map = make_map(data)
    return solver(risk_map)


def below_to_nine(num):
    return num % 9 if num != 9 else 9


def extend_map(risk_map):
    new_risk_map = defaultdict(int)
    dimension = list(risk_map.keys())[-1][0] + 1
    for i in range(5):
        for loc, risk in risk_map.items():
            new_risk_map[(loc[0], loc[1]+i*dimension)] = below_to_nine(risk+i)
    final_map = defaultdict(int)
    for i in range(5):
        for loc, risk in new_risk_map.items():
            final_map[(loc[0] + i*dimension, loc[1])] = below_to_nine(risk+i)
    return final_map


def day15_2(data):
    risk_map = make_map(data)
    risk_map = extend_map(risk_map)
    return solver(risk_map)
```

Today I learned about the Dijkstra's algorithm, which is used to find the shortest path between two nodes in a graph. Since I know little about it, a lot of time was spent on the algorithm. But after implementing it, the code become much easier.

## [Day 16: Packet Decoder](https://adventofcode.com/2021/day/16)

You have a hexadecimal input string(such as: D2FE28), first it will be convert to binary, follow a rule(such as: A=1010) which could be parse into data and oporation packet.

Part1: Decode the structure of your hexadecimal-encoded BITS transmission; what do you get if you add up the version numbers in all packets?

Part2: With the given operation rule, What do you get if you evaluate the expression represented by your hexadecimal-encoded BITS transmission?

```Python
hex_map = {
    "0": "0000",
    "1": "0001",
    "2": "0010",
    "3": "0011",
    "4": "0100",
    "5": "0101",
    "6": "0110",
    "7": "0111",
    "8": "1000",
    "9": "1001",
    "A": "1010",
    "B": "1011",
    "C": "1100",
    "D": "1101",
    "E": "1110",
    "F": "1111"
}


def parse_literals(bits):
    five_bits = bits[:5]
    if five_bits[0] == '0':
        return five_bits
    return five_bits + parse_literals(bits[5:])


def parse_packets(bits):
    version = int(bits[:3], 2)
    type_ID = int(bits[3:6], 2)
    if type_ID == 4:
        # literal value
        num_bits = parse_literals(bits[6:])
        num_filted = ''.join(num_bits[i]
                             for i in range(len(num_bits))
                             if i % 5 != 0)
        num = int(num_filted, 2)
        return {"version": version,
                "type_ID": type_ID,
                "literal": num,
                "sub-packets": None,
                "bits": bits[:6] + num_bits}
    else:
        # an operator
        bit_label = bits[6]
        if bit_label == '0':
            total_sub_length = int(bits[7:7+15], 2)
            sub_bits = bits[7+15:]
            occupy_bits = bits[:7+15]
            sub_packets = []
            while True:
                sub = parse_packets(sub_bits)
                sub_packets.append(sub)
                sub_bits = sub_bits[len(sub["bits"]):]
                occupy_bits += bits[len(occupy_bits):
                                    len(occupy_bits)+len(sub["bits"])]
                if (sum(len(sub["bits"]) for sub in sub_packets)
                   >= total_sub_length):
                    return {"version": version,
                            "type_ID": type_ID,
                            "operator": bit_label,
                            "sub-packets": sub_packets,
                            "bits": occupy_bits}
        elif bit_label == '1':
            num_sub_packets = int(bits[7:7+11], 2)
            sub_bits = bits[7+11:]
            occupy_bits = bits[:7+11]
            sub_packets = []
            for _ in range(num_sub_packets):
                sub = parse_packets(sub_bits)
                sub_packets.append(sub)
                sub_bits = sub_bits[len(sub["bits"]):]
                occupy_bits += bits[len(occupy_bits):
                                    len(occupy_bits)+len(sub["bits"])]
            return {"version": version,
                    "type_ID": type_ID,
                    "operator": bit_label,
                    "sub-packets": sub_packets,
                    "bits": occupy_bits}


def sum_version(packets):
    if not packets["sub-packets"]:
        return packets["version"]
    else:
        return packets["version"] + sum(sum_version(sub)
                                        for sub in packets["sub-packets"])


def day16_1(data):
    bits = ''.join(hex_map[elem] for elem in data[0])
    packets = parse_packets(bits)
    return sum_version(packets)


def evaluate(packets):
    if packets["type_ID"] == 4:
        return packets["literal"]
    elif packets["type_ID"] == 0:
        # sum
        return sum(evaluate(sub) for sub in packets["sub-packets"])
    elif packets["type_ID"] == 1:
        # product
        return reduce(lambda x, y: x*y,
                      [evaluate(sub) for sub in packets["sub-packets"]])
    elif packets["type_ID"] == 2:
        # min
        return min(evaluate(sub) for sub in packets["sub-packets"])
    elif packets["type_ID"] == 3:
        # max
        return max(evaluate(sub) for sub in packets["sub-packets"])
    elif packets["type_ID"] == 5:
        # greater than
        return (1
                if (evaluate(packets["sub-packets"][0]) >
                    evaluate(packets["sub-packets"][1]))
                else 0)
    elif packets["type_ID"] == 6:
        # less than
        return (1
                if (evaluate(packets["sub-packets"][0]) <
                    evaluate(packets["sub-packets"][1]))
                else 0)
    elif packets["type_ID"] == 7:
        # equal
        return (1
                if (evaluate(packets["sub-packets"][0]) ==
                    evaluate(packets["sub-packets"][1]))
                else 0)


def day16_2(data):
    bits = ''.join(hex_map[elem] for elem in data[0])
    packets = parse_packets(bits)
    return evaluate(packets)
```

It's a long day, since there are so many rules to parse the packets. The tricky part is to figure out the data structure to represent the packets. After relizing the packet may be recursive containing other packet, I finally choose json-like data structure to store the packets. 

The code is a little bit messy, maybe I will refactor it later or maybe not.

## [Day 17: Trick Shot](http://adventofcode.com/2021/day/17)

Input represents the an area of grid, which covered by trajectory. The probe starts from (0, 0), with each iter, the location change by rule with velocity who's loc also change ervery step. The velocity is represented by a tuple of (x, y). For special velocity, the probe will within or not within the target area after some steps.

Part1: Find the initial velocity that causes the probe to reach the highest y position and still eventually be within the target area after any step. What is the highest y position it reaches on this trajectory.

Part2: How many distinct initial velocity values cause the probe to be within the target area after any step

```python
def move_once(probe_loc, velocity):
    x, y = probe_loc
    dx, dy = velocity
    new_probe_loc = (x + dx, y + dy)
    if dx == 0:
        new_velocity_x = dx
    elif dx > 0:
        new_velocity_x = dx - 1
    else:
        new_velocity_x = dx + 1
    new_velocity = (new_velocity_x, dy - 1)
    return new_probe_loc, new_velocity


def simulate(x0, x1, y0, y1, velocity):
    probe_loc = (0, 0)
    path = []
    while True:
        probe_loc, velocity = move_once(probe_loc, velocity)
        path.append(probe_loc)
        x, y = probe_loc
        if x > x1 or y < y0:
            within_target = any(x0 <= x <= x1 and
                                y0 <= y <= y1
                                for x, y in path)
            return within_target, path


def solver(x0, x1, y0, y1):
    mem = {(x, y): simulate(x0, x1, y0, y1, (x, y))
           for x in range(-x1 - 1, x1 + 1)
           for y in range(y0 - 1, -y0 + 1)}
    return mem


def day17_1(x0, x1, y0, y1):
    mem = solver(x0, x1, y0, y1)
    mem = {k: max(elem[1] for elem in v[1])
           for k, v in mem.items()
           if v[0]}
    max_loc = max(mem, key=mem.get)
    return mem[max_loc]


def day17_2(x0, x1, y0, y1):
    mem = solver(x0, x1, y0, y1)
    mem = {k: v for k, v in mem.items() if v[0]}
    return len(mem.keys())

data = parse_data(day=17,
                    parser=lambda x: re.findall(
                        r'.*?x=(\d+)..(\d+), y=-(\d+)..-(\d+)', x))
x0, x1, y0, y1 = data[0][0]
x0, x1, y0, y1 = int(x0), int(x1), -int(y0), -int(y1)
```

Using brute force to solve this problem.

## [Day 18: Snailfish](http://adventofcode.com/2021/day/18)

This is a special math.

- formal be like: [[1,9],[8,5]]
- each formal must be reduced by the rule:
    - If any pair is nested inside four pairs, the leftmost such pair explodes
    - If any pair is nested inside four pairs, the leftmost such pair explodes
    - During reduction, at most one action applied
    - To explode a pair, the pair's left value is added to the first regular number to the left of the exploding pair (if any), and the pair's right value is added to the first regular number to the right of the exploding pair (if any)
    - To split a regular number, replace it with a pair; the left element of the pair should be the regular number divided by two and rounded down, while the right element of the pair should be the regular number divided by two and rounded up
- The magnitude of a pair is 3 times the magnitude of its left element plus 2 times the magnitude of its right element

Part1： Add up all of the snailfish numbers from the homework assignment in the order they appear. What is the magnitude of the final sum?

Part2: The magnitude of a pair is 3 times the magnitude of its left element plus 2 times the magnitude of its right element?

```Python
def gen_explode_parts(exp):
    depth = 0
    for i, char in enumerate(exp):
        if char == '[':
            depth += 1
        elif char == ']':
            depth -= 1

        if depth == 5:
            break

    for j, char in enumerate(exp[i:]):
        if char == ']':
            break

    if depth == 5:
        # pair within four pairs, left part and right part
        return exp[i:i+j+1], exp[:i], exp[i+j+1:]
    else:
        return None, None, None


def explode(exp):
    def add_left(m):
        return str(int(m.group(0)) + left_num)

    def add_right(m):
        return str(int(m.group(0)) + right_num)

    exp = str(exp)
    exp_four_pair, left_exp, right_exp = gen_explode_parts(exp)
    if not exp_four_pair:
        return exp

    left_num, right_num = ast.literal_eval(exp_four_pair)
    exp_left = re.sub(r"\d+(?=\D*$)", add_left, left_exp)
    exp_right = re.sub(r"\d+", add_right, right_exp, count=1)

    final_exp = exp_left + '0' + exp_right
    return ast.literal_eval(final_exp)


def split(exp):
    def subsplit(m):
        num = int(m.group(0))
        return f"[{math.floor(int(num) / 2)},{math.ceil(num / 2)}]"

    exp = str(exp)
    out = re.sub(r"\d{2}", subsplit, exp, count=1)
    return ast.literal_eval(out)


def add(a, b):
    return [a, b]


def reduce(exp):
    out = explode(exp)
    if str(out) == str(exp):
        # explode until there is no explode and then split
        out = split(exp)

    if str(out) == str(exp):
        return exp
    else:
        return reduce(out)


def calc_magnitude(exp):
    def magnitude(m):
        a = ast.literal_eval(m.group(0))
        return str(a[0] * 3 + a[1] * 2)

    exp = str(exp)
    out = re.sub(r'\[\d+, \d+\]', magnitude, exp)
    out = ast.literal_eval(out)

    if isinstance(out, list):
        return calc_magnitude(out)
    else:
        return out


def day18_1(data):
    data = copy.deepcopy(data)
    running = data.pop(0)
    while data:
        next_add = data.pop(0)
        running = add(running, next_add)
        running = reduce(running)
    return calc_magnitude(running)


def day18_2(data):
    scores = []
    for elem in combinations(data, 2):
        left, right = elem
        score1 = calc_magnitude(reduce(add(left, right)))
        score2 = calc_magnitude(reduce(add(right, left)))
        scores.append(score1)
        scores.append(score2)
    return max(scores)
```

At first, I use nested list to represent the formula. But after some try, I can't found a clear way to explode(split works ok). Almost giving up, then the [Aoc Subreddit](https://www.reddit.com/r/adventofcode) helped me, I see someone use tree structure to represent the formula and other people use string to represent the formula. String? Really? When I see the [code](https://topaz.github.io/paste/#XQAAAQDMCwAAAAAAAAA0m0pnuFI8c+fPp4HB1KcQKnBu+WCk1EVjq93Wu6MmEjgQZO0R0cACtyObjsZhwTf3oOb2ujEhwIrJbER41xleoj0A+3ovoDJWPmAj5gWoFfXX7qRpA49tiHhhxInXZ0x0+oBycxaAp7uOKcRuAT1PpSNaxTT4WQ+cZ6QzAKIaFJoRIvtVJPExXtg9l2G7ugtsSdLRCFtYOSLiuFwiKWw/1/2ddZ8Yk3tgNCSpfrsN9JzeJfJTLv5qM8GPToy1St2XxPrSHQPsqrI4HRKwslnQXHeVrTsSZsVOMXrEsc165FW1FQQqQqviOyw6mchlPM69/fmhh2h24RIXbOg7XHOifNyePqZA8pe5KlbNtEdgG0qhW/1DV3ZFL6Ia1nW4kgaej0OtYjGnyfZOki3E+Ik3/ZZ89mhZdsqkhtVuHNn7MqPq+KEtus2jmNwc3qMFt34fCIZykq0KHdqqQZIMmVwlja90EFoSGSmJTilsomnK6540XulRnJcVc436LIMdDnjXR5OI29V3DOxALI/zoVKtIlLBPb9R/heQOOloIL0wSaQHOKTv7vSTXrSnSaDw0/3Xs9Dz1MyT/+aEw8kLU9fOgXYJn1jgq862gXI8VXnX2toEgOLY+uWFtiE92jqzpnMSI/g7ndAcCmu5nGDcSbRa8tkaXU/TizR7t1zEruemwrQceymySnPVTyeBvaiFAcrnifZeJ4fWnjv+jTQx9ugpcV3BiEJCKKrK4f5sy+lqTUXoWzKqCBxFi++UzpaXjldOk0B8YsZxgk5N/FzoCr2gbKNJ0lcfxvFKYK3UNGa+FTcI4egAvJ7Q2p+ZDE+LXBC2XmugmS7xpOqHpde5unXSINM/MhPda1T+BopN4PFIJ1lCHi1wf6Wmeoa7otoabg7UzMuajVL7a2UE7FnrPwPbqMZWERofVM92HBTQGjahBFipL1700/DzshTrQe8x4aq5qEvfVdLXPOgn79pL/UOAmEoPJijQDzsuPynLtFksYx/cno5rsAnWNjOOvVXoun4HAC5Ky0jofhUZ8IMIcxoZhO+9FDPHE3tzJXhes+xHbf5O0bZ6ayv6Lcam0dHg8M6aQdFjB4Inzk4IzYPSksGiczXmvRBRN1WjNiHTGhxZ2E10zjXsfjr+j2cl7QGAmOHRHgotpoZwyEao+Olo9mKtrCWii6hPP1lpD3lN5tiisf9+N5KApOrVRHB9EUvVsoqqdcCnrhjgrBEgpnsKs11rSQM9rNdi8jcPsF62UEMwX6T770iWE+1dVDZPGqVunilny5ARwU7krIt/ghYWa1RZH/i5PXVgurYoFTbH6+Xq8EWjbsGfEVanLk51XvZgu9A61bjgnqRYvGCB/DujaaECyGdbyz+zsNlv4L91xBInvOtgO9z25wKuvDezgBegaW66vajmpcQU//qqsm0=), I know string will work better. So I change the solution to use string. Amazing!


## [Day 19: Beacon Scanner](https://adventofcode.com/2021/day/19)

- scanners report some beacon and the scanner may be rotated by 90 degree in xyz axes and facing positive or negative(24 different orientations). 
- two scanners near overlapped at least 12 beacons.

Part1: Assemble the full map of beacons. How many beacons are there?

Part2: Assemble the full map of beacons. How many beacons are there?

```Python
class Scanner:
    def __init__(self, beacons) -> None:
        self.beacons = beacons
        self.final = set()
        self.offset = None

    def xy_rotation(self):
        yield lambda x, y, z: (x, y, z)
        yield lambda x, y, z: (y, -x, z)
        yield lambda x, y, z: (-x, -y, z)
        yield lambda x, y, z: (-y, x, z)

    def yz_rotation(self):
        yield lambda x, y, z: (x, y, z)
        yield lambda x, y, z: (x, z, -y)
        yield lambda x, y, z: (x, -y, -z)
        yield lambda x, y, z: (x, -z, y)

    def xz_rotation(self):
        yield lambda x, y, z: (x, y, z)
        yield lambda x, y, z: (z, y, -x)
        yield lambda x, y, z: (-x, y, -z)
        yield lambda x, y, z: (-z, y, x)

    def rotations(self):
        for xy in self.xy_rotation():
            for yz in self.yz_rotation():
                for xz in self.xz_rotation():
                    rotated_beacons = set()
                    for beacon in self.beacons:
                        rotated_beacons.add(xz(*yz(*xy(*beacon))))
                    yield rotated_beacons

    def translate(self, beacons, offset):
        return set([(beacon[0] + offset[0],
                    beacon[1] + offset[1],
                    beacon[2] + offset[2]) for beacon in beacons])


def solve(data):
    scanners = []
    for part in data:
        beacons = set()
        for line in part.split('\n'):
            if not line.startswith('--'):
                loc = [int(num) for num in line.split(',')]
                beacons.add(tuple(loc))
        scanners.append(Scanner(beacons))

    scanners[0].final = scanners[0].beacons
    scanners[0].offset = (0, 0, 0)

    fixed_scanner = set()
    fixed_scanner.add(scanners[0])

    while len(fixed_scanner) < len(scanners):
        for scanner in scanners:
            if scanner in fixed_scanner:
                continue

            fixed_beacons = set().union(*[s.final for s in fixed_scanner])
            for r in scanner.rotations():
                for floc in fixed_beacons:
                    for loc in r:
                        offset = (floc[0] - loc[0],
                                  floc[1] - loc[1],
                                  floc[2] - loc[2])
                        shifted = scanner.translate(r, offset)
                        if len(shifted.intersection(fixed_beacons)) >= 12:
                            scanner.final = shifted
                            scanner.offset = offset
                            fixed_scanner.add(scanner)
                            break
    return fixed_scanner, scanners


def day19_1(data):
    fixed_scanner, _ = solve(data)
    return len(set().union(*[s.final for s in fixed_scanner]))


def manhattan_distance(s1, s2):
    loc1 = s1.offset
    loc2 = s2.offset
    return (abs(loc1[0] - loc2[0]) +
            abs(loc1[1] - loc2[1]) +
            abs(loc1[2] - loc2[2]))


def day19_2(data):
    _, scanners = solve(data)
    return max(manhattan_distance(s1, s2)
               for s1, s2 in combinations(scanners, 2))
```

Today，I cheated and not found the solution by myself, the origin code is [here](https://github.com/JamesMCo/Advent-Of-Code/tree/master/2021/19). I just understand the logic and rewrite some code.

Honesty is the best policy.

## [Day 20: Trench Map](https://adventofcode.com/2021/day/20)

You get a 2D image(# represent light, . represent dark) and an algorithm(a seqence contains # and .) to enhence the image.

With the enhence rule, the image(infinite) will change the light and dark pixel.

- the pixel beyound the image will be initialized as dark(represent by .)
- to determine the pixel after enhencing, combine the nine neighbors(from left-top to right-down, including itself) and convert # to 1 and . to 0
- then convert the binary number to decimal number(such as '000100010' to 34)
- use the decimal number to determine the pixel after enhencing(index of the algorithm)
- all pixel change simultaneously

Part1: Start with the original input image and apply the image enhancement algorithm twice, being careful to account for the infinite size of the images. How many pixels are lit in the resulting image?

Part2: Start again with the original input image and apply the image enhancement algorithm 50 times. How many pixels are lit in the resulting image?

```Python
def gen_data():
    data = parse_data(day=20, parser=str, sep='\n\n')
    algorithm = data[0].replace('\n', '')
    image = defaultdict()
    for y, line in enumerate(data[1].split('\n')):
        for x, char in enumerate(line):
            image[(x, y)] = char
    return algorithm, image


def nine_locs(x, y):
    return [(x-1, y-1), (x, y-1), (x+1, y-1),
            (x-1, y), (x, y), (x+1, y),
            (x-1, y+1), (x, y+1), (x+1, y+1)]


def translate(loc, image, algorithm, step):
    code = ''
    for nloc in nine_locs(*loc):
        if step % 2 == 0 and algorithm[0] == '#':
            if nloc in image:
                code += '1' if image.get(nloc) == '#' else '0'
            else:
                code += '1'
        else:
            code += '1' if image.get(nloc) == '#' else '0'
    index = int(code, 2)
    return algorithm[index]


def simulate(image, algorithm, step):
    new_image = defaultdict()
    locs_to_explore = set()
    for loc in image.keys():
        for near_loc in nine_locs(*loc):
            locs_to_explore.add(near_loc)
    for loc in locs_to_explore:
        new_image[loc] = translate(loc, image, algorithm, step)
    return new_image


def day20_1(algorithm, image):
    for step in range(1, 3):
        image = simulate(image, algorithm, step)
    return list(image.values()).count('#')


def day20_2(algorithm, image):
    for step in range(1, 51):
        image = simulate(image, algorithm, step)
    return list(image.values()).count('#')
```

The tricky part is the algorithm start with a # and end with a ., which means the infinate pixel far away from the origin image will blink every(2 enhence steps). 

The 1st step, the nine neighbors of the pixel are all ., so the binary number will be 000000000, which will change the pixel to #. 

The 2nd step, the nine neighbors of the pixel are all #, so the binary number will be 111111111, which will change the pixel to ".". 

Didn't realize that at first, so spending a lot of time to debug. Auctally, it's easy if you find out the above tricky part.


## [Day 21: Dirac Dice](https://adventofcode.com/2021/day/21)

Two player play a dirac dice game.

- each player roll a dice three times
- the player move(around 1 to 10) follow the dice number(if greater than 10, then move to 1)
- with each move, the player's score increase by the position of the player(after move)

Part1: 

- the dice is deterministic with 100-sided(meaning the dice rolls 1, 2...100...1)
- the player who's score is greater than 1000 wins
- question: what do you get if you multiply the score of the losing player by the number of times the die was rolled during the game?

Part2:

- the dice is quantum, when you roll it, the universe splits into 3 copys: one where the outcome of the roll is 1, one where the outcome of the roll is 2, and one where the outcome of the roll is 3.
- the player who's score is greater than 24 wins
- question: find the player that wins in more universes; in how many universes does that player win?

```Python
def play_game(p_pos, p_score, dice_pos):
    move_pos = 0
    for i in range(3):
        if (dice_pos + i) <= 100:
            move_pos += (dice_pos + i)
        else:
            move_pos += (dice_pos + i) - 100

    if (p_pos + move_pos) % 10 == 0:
        p_pos = 10
    else:
        p_pos = (p_pos + move_pos) % 10

    p_score += p_pos
    dice_pos = dice_pos + 3 if (dice_pos + 3) <= 100 else dice_pos + 3 - 100

    return p_pos, p_score, dice_pos


def day21_1():
    p1_pos, p2_pos = (7, 1)
    p1_score, p2_score = 0, 0
    dice_pos, dice_rolls = 1, 0
    while True:
        p1_pos, p1_score, dice_pos = play_game(p1_pos, p1_score, dice_pos)
        dice_rolls += 3
        if p1_score >= 1000:
            return p2_score * dice_rolls

        p2_pos, p2_score, dice_pos = play_game(p2_pos, p2_score, dice_pos)
        dice_rolls += 3
        if p2_score >= 1000:
            return p1_score * dice_rolls


def play_once(p_score, p_pos, rolls):
    if (p_pos + sum(rolls)) % 10 == 0:
        p_pos = 10
    else:
        p_pos = (p_pos + sum(rolls)) % 10
    p_score += p_pos
    return p_pos, p_score


@lru_cache(maxsize=None)
def count_wins(current_player, p1_pos, p1_score, p2_pos, p2_score):
    if p1_score >= 21:
        return 1, 0
    if p2_score >= 21:
        return 0, 1

    wins = [0, 0]
    for rolls in product(range(1, 4), repeat=3):
        if current_player == 0:
            new_pos, new_score = play_once(p1_score, p1_pos, rolls)
            win0, win1 = count_wins(1, new_pos, new_score, p2_pos, p2_score)
        else:
            new_pos, new_score = play_once(p2_score, p2_pos, rolls)
            win0, win1 = count_wins(0, p1_pos, p1_score, new_pos, new_score)
        wins[0] += win0
        wins[1] += win1
    return wins


def day21_2():
    p1_pos, p2_pos = (7, 1)
    p1_score, p2_score = 0, 0
    wins = count_wins(0, p1_pos, p1_score, p2_pos, p2_score)
    return max(wins)
```

Part one is easy, just follow the rules. Part two is kind of hard to code. I finally refrenced the solution from reddit post.

## [Day 22: Reactor Reboot](https://adventofcode.com/2021/day/22)

The reboot process is followed by steps to turn on or turn off the grid(like: on x=11..13,y=11..13,z=11..13, off x=9..11,y=9..11,z=9..11). 

Part1: execute the reboot steps. Afterward, considering only cubes in the region x=-50..50,y=-50..50,z=-50..50, how many cubes are on?

Part2: Starting again with all cubes off, execute all reboot steps. Afterward, considering all cubes, how many cubes are on?

```Python
def parse_line(line):
    def parse_loc(line):
        line_min, line_max = line[2:].split('..')
        return int(line_min), int(line_max)

    action, cubes = line.split(' ')
    x, y, z = cubes.split(',')
    x_min, x_max = parse_loc(x)
    y_min, y_max = parse_loc(y)
    z_min, z_max = parse_loc(z)
    return (action, x_min, x_max, y_min, y_max, z_min, z_max)


def day22_1(data):
    down_limit = -50
    up_limit = 50
    system = defaultdict()
    for line in data:
        action, x_min, x_max, y_min, y_max, z_min, z_max = line
        if (x_min >= up_limit or x_max <= down_limit or
           y_min >= up_limit or y_max <= down_limit or
           z_min >= up_limit or z_max <= down_limit):
            continue
        for x in range(x_min, x_max + 1):
            for y in range(y_min, y_max + 1):
                for z in range(z_min, z_max + 1):
                    system[(x, y, z)] = action
    return len([k for k, v in system.items() if v == 'on'])


def get_diff_cubes(cube_light, other_cub):
    x0, x1, y0, y1, z0, z1 = cube_light
    x3, x4, y3, y4, z3, z4 = other_cub
    if (x0 > x4 or x1 < x3 or y0 > y4 or y1 < y3 or z0 > z4 or z1 < z3):
        # no overlap
        return [(x0, x1, y0, y1, z0, z1)]
    sub_cubes = []
    left_x = (x0, max(x0, x3) - 1)
    mid_x = (max(x0, x3), min(x1, x4))
    right_x = (min(x1, x4) + 1, x1)

    left_y = (y0, max(y0, y3) - 1)
    mid_y = (max(y0, y3), min(y1, y4))
    right_y = (min(y1, y4) + 1, y1)

    left_z = (z0, max(z0, z3) - 1)
    mid_z = (max(z0, z3), min(z1, z4))
    right_z = (min(z1, z4) + 1, z1)

    for x, y, z in product([left_x, mid_x, right_x],
                           [left_y, mid_y, right_y],
                           [left_z, mid_z, right_z]):
        if x == mid_x and y == mid_y and z == mid_z:
            # skip middle cube(overlap with other cube)
            continue
        elif x[0] <= x[1] and y[0] <= y[1] and z[0] <= z[1]:
            # cube substracted
            # 3*9 - 1 = 26 sub cubes(except the most middle cube)
            sub_cubes.append((x[0], x[1], y[0], y[1], z[0], z[1]))

    return sub_cubes


def count_dots(cube):
    x_min, x_max, y_min, y_max, z_min, z_max = cube
    return (x_max - x_min + 1) * (y_max - y_min + 1) * (z_max - z_min + 1)


def day22_2(data):
    light_cubes = []
    for line in data:
        action, *cube = line
        new_cubes = []
        for cube_light in light_cubes:
            new_cubes += get_diff_cubes(cube_light, cube)
        light_cubes = new_cubes[:]
        if action == 'on':
            light_cubes.append(cube)
    return sum(count_dots(cube) for cube in light_cubes)
```

## [Day 23: Amphipod](https://adventofcode.com/2021/day/23)

You get a 2D map(# represents a wall, . represents an open space) of the area(below is an example). Moving each ABCD need different evergy. 

Part1: Your task is to find the least energy for move everyone to it's room(ABCD as blow).

```Text
#############
#...........#
###B#C#B#D###
  #A#D#C#A#
  #########
```

Part2: The actual map is folded, a two line to the map(blow is an example). And find the least energy for move everyone to it's room.

```Text
#############
#...........#
###B#C#B#D###
  #D#C#B#A#
  #D#B#A#C#
  #A#D#C#A#
  #########
```

```Python
def gen_data():
    data = parse_data(day=23)
    state = []
    state.append(data[1][1])
    for loc in range(1, 6):
        state.append(data[1][2 * loc])
    state.append(data[1][11])

    for row in range(4):
        for line in range(4):
            state.append(data[line+2][3+row*2])
    return state


def display(state):
    print('#############')
    print("#" + state[0] + '.'.join(state[1:6]) + state[6] + "#")
    for line in range(2):
        t = "###"
        for row in range(4):
            t += state[7 + row + line] + '#'
        t += '##'
        print(t)
    print('#############')


def gen_state(s, a, b):
    return s[:a] + s[b] + s[a+1:b] + s[a] + s[b+1:]


Homes = {'A': 0, 'B': 1, 'C': 2, 'D': 3}
Costs = {'A': 1, 'B': 10, 'C': 100, 'D': 1000}


def move_home(s, cost):
    global Homes
    global Costs
    for hp in range(7):
        if s[hp] == '.':
            continue
        i = hp
        a = s[i]
        r = Homes[a]
        ofs = 7 + r*4
        line = 3
        while line > 0 and s[ofs+line] == a:
            line -= 1
        if s[ofs+line] != '.':
            continue
        cb = (2+line)*Costs[a]
        # go right
        while i < r + 1 and s[i+1] == '.':
            cb += Costs[a]*2 if i > 0 else Costs[a]
            i += 1
        # go left
        while i > r + 2 and s[i-1] == '.':
            cb += Costs[a]*2 if i < 6 else Costs[a]
            i -= 1
        if i != r + 1 and i != r + 2:
            continue
        return move_home(gen_state(s, hp, ofs+line), cost + cb)
    return (s, cost)


def move_out(s):
    global Homes
    global Costs
    # First move everybody in, if possible
    valid = []
    # Then try to get out, if possible
    for row in range(4):
        ofs = 7 + row*4
        line = 0
        while line < 4 and s[ofs+line] == '.':
            line += 1
        if line == 4:
            continue
        a = s[ofs+line]
        if (row == Homes[a] and
           (line == 3 or
           all(s[i] == a for i in range(ofs + line + 1, ofs+4)))):
            continue
        rr = row + 2
        cb = Costs[a]*line
        while rr < 7 and s[rr] == '.':
            cb += 2*Costs[a] if rr < 6 else Costs[a]
            valid.append(move_home(gen_state(s, rr, ofs+line), cb))
            rr += 1
        ll = row + 1
        cb = Costs[a]*line
        while ll >= 0 and s[ll] == '.':
            cb += 2*Costs[a] if ll > 0 else Costs[a]
            valid.append(move_home(gen_state(s, ll, ofs+line), cb))
            ll -= 1
    return valid


def search(start):
    queue = []
    for _ in range(100000):
        queue.append([])
    queue[0].append(start)
    previous = {start: None}
    mind = {start: 0}
    for cost in range(50000):
        for state in queue[cost]:
            if mind[state] < cost:
                continue
            valid = move_out(state)
            if all(state[i] == "." for i in range(7)) and len(valid) == 0:
                paths = []
                while state:
                    paths.append(state)
                    state = previous[state]
                return cost, paths
            for (nstate, ncost) in valid:
                if nstate in mind and mind[nstate] <= cost+ncost:
                    continue
                previous[nstate] = state
                mind[nstate] = cost + ncost
                queue[cost + ncost].append(nstate)


def solver(data):
    cost, _ = search(''.join(data))
    return cost
```

A Dijkstra search with greedy heuristic. I didn't solve this with my own. The code is referenced from [here](https://github.com/p88h/aoc2021/blob/main/other/day23.py). My origin approach is too complicated to apply a Dijkstra search. 

It's still fun to play to understand how clever other people's solution is.


## [Day 24: Arithmetic Logic Unit](https://adventofcode.com/2021/day/24)

You are going to build a new kind of computer. With blow rule:
- inp a - Read an input value and write it to variable a.
- add a b - Add the value of a to the value of b, then store the result in variable a.
- mul a b - Multiply the value of a by the value of b, then store the result in variable a.
- div a b - Divide the value of a by the value of b, truncate the result to an integer, then store the result in variable a. (Here, "truncate" means to round the value toward zero.)
- mod a b - Divide the value of a by the value of b, then store the remainder in variable a. (This is also called the modulo operation.)
- eql a b - If the value of a and b are equal, then store the value 1 in variable a. Otherwise, store the value 0 in variable a.
- submarine model numbers are always fourteen-digit numbers consisting only of digits 1 through 9.
- after MONAD has finished running all of its instructions, it will indicate that the model number was valid by leaving a 0 in variable z. However, if the model number was invalid, it will leave some other non-zero value in z

Part1: What is the largest model number accepted by MONAD?

Part2: What is the smallest model number accepted by MONAD?

```Python
import z3


def solve(program):
    solver = z3.Optimize()
    digits = [z3.BitVec(f'd_{i}', 64) for i in range(14)]

    for d in digits:
        solver.add(d >= 1)
        solver.add(d <= 9)
        digit_input = iter(digits)

    zero, one = z3.BitVecVal(0, 64), z3.BitVecVal(1, 64)
    registers = {r: zero for r in 'wxyz'}

    for i, line in enumerate(program):
        vars = line.split()
        if 'inp' in line:
            registers[vars[-1]] = next(digit_input)
            continue
        operator, a, b = vars
        b = registers[b] if b in registers else int(b)
        c = z3.BitVec(f'v{i}', 64)
        if operator == 'add':
            solver.add(c == registers[a] + b)
        elif operator == 'mul':
            solver.add(c == registers[a] * b)
        elif operator == 'mod':
            solver.add(registers[a] >= 0)
            solver.add(b > 0)
            solver.add(c == registers[a] % b)
        elif operator == 'div':
            solver.add(b != 0)
            solver.add(c == registers[a] / b)
        elif operator == 'eql':
            solver.add(c == z3.If(registers[a] == b, one, zero))
        else:
            raise ValueError(f'Unknown operator: {operator}')
        registers[a] = c

    solver.add(registers['z'] == 0)

    for func in (solver.maximize, solver.minimize):
        solver.push()
        func(sum((10 ** i) * d for i, d in enumerate(digits[::-1])))
        solver.check()
        print(f'{func.__name__}')
        m = solver.model()
        print(''.join([str(m[d]) for d in digits]))
        solver.pop()
```

I havn't heard of Z3. The solution is from [reddit](https://www.reddit.com/r/adventofcode/comments/rnejv5/2021_day_24_solutions/).

There is another approach is to analysis the input sequence(which will have special case to solve this problem because of the simularity of each-14 part). But I still like the Z3 approach, which is more elegant and generic.


## [Day 25: Sea Cucumber](https://adventofcode.com/2021/day/25)

- you got a 2D map of the area(. represent emply, > represent east-facing creature and V represent south-facing creature, example as blow). 
- At each step, east-facing creature will move forward one step if the next-east is empty, south-facing creature will move down one step if the next-south is empty.
- sea cucumbers that move off the right edge of the map appear on the left edge, and sea cucumbers that move off the bottom edge of the map appear on the top edge.

```Text
v...>>.vv>
.vv>>.vv..
>>.>v>...v
>>v>>.>.v.
v>v.vv.v..
>.>>..v...
.vv..>.>v.
v.v..>>v.v
....v..v.>
```

Question: To find a safe place to land your submarine, the sea cucumbers need to stop moving. How many steps does it take for the sea cucumbers to stop moving?

```Python
def gen_data():
    data = parse_data(day=25, parser=str)
    system = defaultdict()
    for y, line in enumerate(data):
        for x, c in enumerate(line):
            system[(x, y)] = c
    return system


def move_once(system):
    new_system = defaultdict()
    x_max, y_max = max(system)
    for loc, symbol in system.items():
        # move east-facing
        if loc in new_system:
            continue
        x, y = loc
        next_right_loc = (x + 1, y) if x < x_max else (0, y)
        if symbol == '>' and system[next_right_loc] == '.':
            new_system[next_right_loc] = '>'
            new_system[loc] = '.'
        else:
            new_system[loc] = symbol

    final_system = defaultdict()
    for loc, symbol in new_system.items():
        # move south-facing
        if loc in final_system:
            continue
        x, y = loc
        next_down_loc = (x, y + 1) if y < y_max else (x, 0)
        if symbol == 'v' and new_system[next_down_loc] == '.':
            final_system[next_down_loc] = 'v'
            final_system[loc] = '.'
        else:
            final_system[loc] = symbol

    return final_system


def day25(system):
    num = 0
    while True:
        num += 1
        new_system = move_once(system)
        if all(new_system[loc] == symbol for loc, symbol in system.items()):
            return num
        else:
            system = new_system
```

A kind of simple simulation.

## Conclusion 

At last, this year of adventofcode come to the end. Although some of the problems are too hard, I can't solve it on my own, but it's still a lot of fun to follow the clear thought and solution shared through reddit, github and youtube, from which I have learned so many things. 

Thank you Eric the authors of the advent of code, the community and people who share their thought and solution.