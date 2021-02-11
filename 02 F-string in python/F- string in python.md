# F - String


## Python string formatting at a glance

```cmd
    String Formatting Method 	Expression
- %-formatting 	                'text %s' % variable
- str.format() 	                'text {}.'.format(variable)
- f-string 	                    f'text {variable} text .....'
```
<br>


Here we are filling the place holder with values.<br>
Below is an example of format method.
```python
first_name = 'Naheed'
last_name = 'Rayan'

sentence = 'My name is {} {}'.format(first_name , last_name)
print(sentence)
```

> output:<br>
My name is Naheed Rayan

<br>

Here we will use the f-string
```python
first_name = 'Naheed'
last_name = 'Rayan'

sentence = f'My name is {first_name} {last_name}'
print(sentence)
```

> output:<br>
My name is Naheed Rayan

<br>

We can use function and methods directly with f-string
```python
first_name = 'Naheed'
last_name = 'Rayan'

sentence = f'My name is {first_name.upper()} {last_name.upper()}'
print(sentence)
```

> output:<br>
My name is NAHEED RAYAN

<br>


Use of dictionary.

```python
person = {'name' : 'Naheed' , 'age': 23}

sentence = f"My name is {person['name']} and I am {person['age']} years old."
```
>output:<br>
My name is Naheed and I am 23 years old.

<br>

We can also do calculation.
```python
print(f"4 times 11 is {4*11}") 
```

>output:<br>
4 times 11 is 44

<br>

## ``How to print braces/brackets using Python f-strings?``

You can print braces/brackets using the Python f-strings by using two braces to print single brace and 4 braces to print double braces.

```bash
>>> f'{{"Single Braces"}}'
'{"Single Braces"}'
>>> f'{{{"This will also print Single Braces"}}}'
'{This will also print Single Braces}'
>>> f'{{{{"This will print Double Braces"}}}}'
'{{"This will print Double Braces"}}'
>>> 
```
<br>


## ``Is Python f-strings faster?``

Yes, Python f-strings are faster than any other string formatting in Python i.e.<br> ``str.format`` and ``%-formatting``. The following examples show that the f-strings <br>are twice faster than ``str.format()`` and atleast 80% faster than ``%-formatting.``

<br>
<br>

## ``How to print raw f-strings in Python?``

We can also print raw f-strings by using {variable!r}

```python
>>> name = 'Fred'
>>> f'He said his name is {name!r}.'
"He said his name is 'Fred'."
```
<br>
<br>

# **Different types of padding**

## ``How to pad string with zero in Python f-strings (0-Padding)?``

We can pad a string with zeros (0-Padding) in Python f-strings by adding {variable:0N} inside the braces, where N stands for total number of digits-


```cmd
>>> for n in range(1,11):
...     print(f'The number is {n:02}')
... 
The number is 01
The number is 02
The number is 03
The number is 04
The number is 05
The number is 06
The number is 07
The number is 08
The number is 09
The number is 10
>>> 
```

<br>

## ``How to add decimal places in a float in Python f-strings (Formatting a float)?``


We can define the fixed number of digits after decimal in a float using fstrings by adding {variable:.Nf}, where N stands for total number of decimal places. This doesn’t just strip off the rest of the decimal points, but round it properly.


```cmd
>>> from math import pi
>>> print(f'The value of pi is {pi:.4f}')
The value of pi is 3.1416
>>> 
```

<br>


## ``How to format dates in Python f-strings?``

We can also format the dates in Python f-string by using datetime module and adding the desired format in curly braces {date:directive}.You can find more directives (i.e. %A, etc.) from official page of datetime module.:-


```cmd
>>> import datetime
>>> date = datetime.date(1991, 10, 12)
>>> f'{date} was on a {date:%A}'
'1991-10-12 was on a Saturday'
>>>f'The date is {date:%A, %B %d, %Y}.'
>>>'The date is Saturday, October 12, 1991.'
```

<br>
<br>

## ``How to add space padding in Python f-strings?``

You can add spaces (padding) to a string in Python f strings by adding {variable:N} where N is total length. If the variable is 1 and N is 4, it will add three spaces before 1, hence making the total length 4.

```cmd
>>> for n in range(1,11):
...     print(f'The number is {n:4}')
... 
The number is    1
The number is    2
The number is    3
The number is    4
The number is    5
The number is    6
The number is    7
The number is    8
The number is    9
The number is   10
>>> 
```
<br>

## ``How to justify a string in Python f-strings?``

We can justify a string in Python f-strings using {variable:>N} where N is the total length.


```cmd
s1 = 'Python'
s2 = 'ython'
s3 = 'thon'
s4 = 'hon'
s5 = 'on'
s6 = 'n'

print(f'{s1:>6}')
print(f'{s2:>6}')
print(f'{s3:>6}')
print(f'{s4:>6}')
print(f'{s5:>6}')
print(f'{s6:>6}')
...
Python
 ython
  thon
   hon
    on
     n

....
```



# Python f-strings padding at glance
## 	Padding 	Expression
- 	0 Padding  	{variable:0N}
- 	Decimal Places 	{variable:.Nf}
- 	Date Formatting 	{date : Directive}
- 	Space Padding 	{variable:N}
- 	Justified string 	{variable:>N}

<br>
<br>

# The End