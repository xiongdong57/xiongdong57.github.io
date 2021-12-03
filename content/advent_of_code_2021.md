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
