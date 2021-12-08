+++
title = "Advent of Code 2021"
date = 2021-12-01
draft = false

[taxonomies]
Categories = ["Programming"]
Tags = ["Python"]
+++

Another Advent of Code.
<!-- more -->

## utils.py
```Python
def parse_data(day: int, parser=str, sep='\n'):
    with open(f'input/day{day:02d}.txt') as f:
        return [parser(line) for line in f.read().split(sep)]
```

## day01
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

## day02

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

## day03

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

## day04

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

## day05
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

## day06

each lanternfish creates a new lanternfish once every 7 days. you can model each fish as a single number that represents the number of days until it creates a new lanternfish.

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

## day07

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

## day08

Input be like:  
acedgfb cdfbe gcdfa fbcad dab cefabd cdfgeb eafb cagedb ab | cdfeb fcadb cdfeb cdbaf

before | is the 0-9 numbers, after | can be translate to digits.

Code be like:

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

Part1 is easy after careful understanding the matiral. Part2 first comes to DFS, then I find it's hard to mapping the iterate rule. Then when broswing reddit, someone mentioned brute force. May be  I should calc how many possible combinations and the result number is quit small(less than 10000). Finally, using brute force to solve it.
