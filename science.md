# Science

- [Science](#science)
  - [Math](#math)
    - [Number theory](#number-theory)
    - [Trigonometry](#trigonometry)
    - [Geometry](#geometry)
    - [Statistics](#statistics)
    - [Convenient stuff](#convenient-stuff)
      - [Test if a positive number is a power of two](#test-if-a-positive-number-is-a-power-of-two)
      - [Check if a number is close within the two ends of an interval](#check-if-a-number-is-close-within-the-two-ends-of-an-interval)

## Math

### Number theory

- positive numbers: integers > 0
- natura numbers: positive numbers, plus, sometimes, 0
  - use `nonnegative integer` to avoid ambiguities

### Trigonometry

- tan(θ) = opposite/adjacent
- θ = arctan(opposite/adjacent)

### Geometry

Collision test between rectangles aligned with the axes. Essentially, r1.top_left < r2.bottom_right, and r1.bottom_right > r2.top_left.

```
r2.left > r1.right && r2.right < r1.left && r2.top > r1.bottom && r2.bottom < r1.top
```

### Statistics

Stddev: sqrt( Ε( (xᵢ-avg)² ) / n) )

In words: root of the mean of the square of difference.

### Convenient stuff

#### Test if a positive number is a power of two

`n & (n - 1) == 0`

#### Check if a number is close within the two ends of an interval

Example: Check if on a field 10 units wide, an element is within (0, 1), or (9, 10).

Intuitive explanation: x is far from the center more than (half width - interval).

```sh
$width - x < $interval ||
         x < $interval

# Subtract $width/2:

$width/2 - x < $interval - $width/2 ||
x - $width/2 < $interval - $width/2

# We can't make absolute here, otherwise it's always false, as (interval - width/2) is negative, which
# would make the equation always false.

# Change the sign:

x - $width/2 > $width/2 - $interval ||
$width/2 - x > $width/2 - $interval

# Make absolute!

|x - $width/2| > $width/2 - $interval
```
