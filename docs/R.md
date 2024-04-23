This guidelines have beeing taken and modified from: http://adv-r.had.co.nz/Style.html

## Object names

Variable and function names should be lowercase. Use an underscore (_) to separate words within a name. Generally, variable names should be nouns and function names should be verbs. Strive for names that are concise and meaningful (this is not easy!).

```
# Good
day_one
day_1

# Bad
first_day_of_the_month
DayOne
dayone
djm1
```

Dots (.) should be used to declare frequently used results of a basic function applied to a base variable.

```
numbers <- c(1, 2, 2, 5, 6, 3)
numbers.mean <- mean(numbers)
numbers.sd <- sd(numbers)
```

Where possible, avoid using names of existing functions and variables. Doing so will cause confusion for the readers of your code.

```
# Bad
T <- FALSE
c <- 10
mean <- function(x) sum(x)
```

## Spacing

Place spaces around all infix operators (=, +, -, <-, etc.). The same rule applies when using = in function calls. Always put a space after a comma, and never before (just like in regular English).

```
# Good
average <- mean(feet / 12 + inches, na.rm = TRUE)

# Bad
average<-mean(feet/12+inches,na.rm=TRUE)
```

There’s a small exception to this rule: :, :: and ::: don’t need spaces around them.

```
# Good
x <- 1:10
base::get

# Bad
x <- 1 : 10
base :: get
```

Place a space before left parentheses, except in a function call.

```
# Good
if (debug) do(x)
plot(x, y)

# Bad
if(debug)do(x)
plot (x, y)
Extra spacing (i.e., more than one space in a row) is ok if it improves alignment of equal signs or assignments (<-).

list(
  total = a + b + c, 
  mean  = (a + b + c) / n
)
```

Do not place spaces around code in parentheses or square brackets (unless there’s a comma, in which case see above).

```
# Good
if (debug) do(x)
diamonds[5, ]

# Bad
if ( debug ) do(x)  # No spaces around debug
x[1,]   # Needs a space after the comma
x[1 ,]  # Space goes after comma not before
```

Is it ok to use spaces to make more readable a function definition that runs over multiple lines. In that case, indent the second line to where the definition starts:

```
long_function_name <- function(a = "a long argument", 
                               b = "another argument",
                               c = "another long argument") {
  # As usual code is indented by two spaces.
}
```

## Curly braces

An opening curly brace should never go on its own line and should always be followed by a new line. A closing curly brace should always go on its own line, unless it’s followed by else.

Always indent the code inside curly braces.

```
# Good

if (y < 0 && debug) {
  message("Y is negative")
}

if (y == 0) {
  log(x)
} else {
  y ^ x
}

# Bad

if (y < 0 && debug)
message("Y is negative")

if (y == 0) {
  log(x)
} 
else {
  y ^ x
}
```

It’s ok to leave very short statements on the same line:

```
if (y < 0 && debug) message("Y is negative")
```

## Line length

Strive to limit your code to 80 characters per line. This fits comfortably on a printed page with a reasonably sized font. If you find yourself running out of room, this is a good indication that you should encapsulate some of the work in a separate function.

## Indentation

When indenting your code, use four spaces. Never mix tabs and spaces. If you are working in an old script which uses tabs, keep using tabs until the script is complete and parse them into sets of four spaces.

## Assignment

Use <-, not =, for assignment.

```
# Good
x <- 5
# Bad
x = 5
```

## Organisation

Commenting guidelines
Comment your code. Each line of a comment should begin with the comment symbol and a single space: #. Comments should explain the why, not the what.

Use commented lines of - and = to break up your file into easily readable chunks.

```
# Load data ---------------------------

# Plot data ---------------------------
```