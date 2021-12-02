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
