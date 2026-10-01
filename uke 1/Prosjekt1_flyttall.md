### Task 1

| num 1     | num 2 | diff (magnitude) | Error |
| --------- | ----- | ---------------- | ----- |
| $10^{10}$ | 10    | $10^{9}$         | no    |
| $10^{15}$ | 10    | $10^{14}$        | no    |
| $10^{17}$ | 10    | $10^{16}$        | no    |
| $10^{18}$ | 10    | $10^{17}$        | yes   |
| $10^{20}$ | 10    | $10^{19}$        | yes   |

| num 1     | num 2    | diff (magnitude) | Error |
| --------- | -------- | ---------------- | ----- |
| $10^{10}$ | $10^{5}$ | $10^{4}$         | no    |
| $10^{15}$ | $10^{5}$ | $10^{10}$        | no    |
| $10^{17}$ | $10^{5}$ | $10^{12}$        | no    |
| $10^{18}$ | $10^{5}$ | $10^{13}$        | yes   |
| $10^{20}$ | $10^{5}$ | $10^{15}$        | yes   |

| num 1     | num 2     | diff (magnitude) | Error |
| --------- | --------- | ---------------- | ----- |
| $10^{21}$ | $10^{20}$ | $10$             | no    |
| $10^{22}$ | $10^{20}$ | $10^{2}$         | no    |
| $10^{23}$ | $10^{20}$ | $10^{3}$         | yes   |
| $10^{25}$ | $10^{20}$ | $10^{5}$         | yes   |
| $10^{30}$ | $10^{20}$ | $10^{10}$        | yes   |

IEEE 754 floating point
$(-1)^S * (1.M)_2*2^e$ 
S is the sign bit, does +-, if it is 0 its a positive number
M the mantisse, contains the precision of the number
e the exponent has the scale of the number


**Floating point summation error** 
If there is a big difference in magnitude the mantisse bits of the smaller number has to add more zeroes (at the most significant ones) to line up the bits (to do the addition). This has the possibility to push the significant digits out of the 52 bit range, on the right side (least significant). This can be done because the exponent can compensate for this.

**Why not add to the big number**
This would cause the same issue, but instead you would push the most significant bits out of the 52 bits, by adding 0 to the least Signiant. 

**Example:**
$1024 + 0.001$

$1024 =2^{10}$, normalized: $1.0000000000*2^{10}$ 
Positive: sign bit = 0
exponent = 10 + 15 (bias) = 25
Mantissa: all zero
Bitstring: 0 | 11001 | 0000000000

$0.001 \approx 1.0000011000 *2^{-10}$,
Positive: sign bit = 0
exponent = -10 + 15 (bias) = 5
mantissa: 0000011000
Bitstring: 0 | 00101 | 0000011000

Exponent difference: 25 - 5 = 20
Bump by 20 $0.00000000000000000001 0000011000 *2^{10}$

**Moving by more than 10 bits removes all info:**
==1.0000011000==, 11 bits in the window
==0.0000000001==0000011000, only the implicit 1 remains in the window. 
So if the exponent difference is 11, (2^11 = 2048, 2048 times larger), we lose all info

Both normalized number need to have the **same exponent** for us to be able to do the addition.




Fix
 a + b - a  = a - a + b
change order so difference in significate bits are smaller



CODE:
```
def ordinary_sum(values):
    # Start med summen 0.
    total = 0.0

    # Legg til ett tall om gangen, fra venstre mot høyre.
    for value in values:
        total = total + value
    return total 
```

```
import math

values = [1.0, 2.0, 3.0]

ordinary = ordinary_sum(values)
reference = math.fsum(values)

print("ordinary_sum =", ordinary)
print("referanse    =", reference)
print("absolutt feil=", abs(ordinary - reference))
```

### Task 2
CODE: 
```
def make_attack(n, scale):
    # n bestemmer hvor stort eksperimentet skal være.
    # scale kan brukes til å styre størrelsesforskjellen mellom tallene.
    # TODO: lag og returner din egen liste.
    values = [1e5*n*scale, scale, -1e5 * n*scale]
    
    return values
```

```
import math
for n in [10, 100, 1000]:
    values = make_attack(n, 1e16)
    ordinary = ordinary_sum(values)
    reference = math.fsum(values)
    error = abs(ordinary - reference)
    print(
        f"n={n:5d}   "
        f"ordinary={ordinary:.12g}   "
        f"reference={reference:.12g}   "
        f"error={error:.3e}"
    )
```

The strength of the attack is affected by the magnitude difference between the small and large numbers. The larger the difference the higher the error, with max error at exponent difference of 53 (2^53=10^16 magnitude difference)

The attack grows stronger as n increases, its max strength is reached when it has the value 9*10^16 where all info is lost. The attack is weaker as n is not great enough to significantly change the exponent, thus there are still some bits of information left. 

### Task 3
```
import math

def target_attack():
	# TODO: konstruer tallene dine her.
    values = [1e100, 10**-12, -1e100, 1]
    return values

values = target_attack()
ordinary = ordinary_sum(values)
reference = math.fsum(values)

print("antall tall  =", len(values))
print("ordinary_sum =", ordinary)
print("referanse    =", reference)
print("feil         =", abs(ordinary - reference))
```

$|S_{ref}-1| < 10^{-10}$ 
`antall tall = 4` 
`ordinary_sum = 1.0`, goes to 1 as show earlier (Task 1) 
`referanse = 1.000000000001` 
`feil = 1.000088900582341e-12`
This does not make sense as the error should be 1e-12, but its is actually 
1.000088900582341e-12 (1e100 + 1e-12 -1e100 + 1 -1 = 10^-12)

The error in feil comes from the rounding rule of IEEE 754. 1+ 1e-12 need to be rounded, and that is why the error is not exactly 1e-12

Why 1e-12:
 it must stay under 10^-10, and above 10^-16 (to not be hidden by wipeout)

NOTE: 
Ordinary_sum has wipeout error (here 1e-12), which is caused by huge difference in exponent (2^52)
fsum has rounding noise (2^-53) near 1.0, this is the smallest gap between numbers near 1.0. 
10^-10 caps maximium wipeout error


No construction can push error meaningfully above 10^-11
terms close to 2^-52 get removed due to IEEE rounding 
Given a bound for the reference, |Ref - 1| < 10^-10, there is no construction of values that create an error in ordinary sum that is greater than 10^-10.



### Task 4
The rounding threshold and wipeout threshold are the same boundary, both are set by 2^-53, relative to the large number.
There's no gap between "too small to survive `ordinary_sum`'s addition" and "too small for even `math.fsum` to represent as different from the accumulator alone."

ROUNDING FLOOR at x := x * 2^-52   

Single big + small addition can never show a detectable error (wipeout and threshold rounding error are the same)
Floating point addition is not assosiative. Adding many small terms, may lead them to survive a addition with a large number
the runtime's _summation order_ determines how much of the small terms' total contribution gets rescued vs. lost

### Task 5
From task 4 we determine that its better to start with a sum of small numbers, in the hope that they will accumulative to a number that is of enough significance. 

We use the absolute value as there, because we need the numerical value. 
Consider: $[-10000,  1, 10000]$ 
The numbers are in the accenting order, but the exponent difference is still large. This is caused by the numerical value, to fix this sort based on acceding absolute value.  
```
def defense_order(values):
	# Lag en kopi slik at originaldataene ikke endres.
    reordered = list(values)
    
    # Summer små tall før store tall.
    reordered.sort(key=abs)
    
    return reordered
```


```
def defense_order(values):
	# Lag en kopi slik at originaldataene ikke endres.
    reordered = list(values)
    
    # Summer små tall før store tall.
    reordered.sort(key=abs)
    
    return reordered
```



### Task 7

Nauerman summation 
```
def better_sum(values):
    total = 0.0
    correction = 0.0
    
    for value in values:
        t = total + value
	    if abs(total) >= abs(value):
		    # Beregn avrundingsfeilen fra addisjonen.
		    correction += (total - t) + value
		else:
		    # Beregn avrundingsfeilen fra addisjonen, med value som største tall.
		    correction += (value - t) + total
	
		# Oppdater hovedsummen til den nye, avrundede summen.
		total = t
        
    return total + correction
```
The main thing here is to keep terms that are too small in their own "correction" term, hoping that it will be big enough to be represented later. 


### Questions:

1. The strongest attack happened when there was a $10^{16}$  or $(2^{52})$ difference in magnitude, this is because all the bits in the mantissa needs to be shifted to make the exponents match. 
2. The error is caused by the first summation, where there is a sufficient exponent difference [[#Lower bound]].
3. The larger the difference between the numbers the higher the error
4. If you limit large numbers there can still be meaning full exponent difference, ie 1e1 vs 1e-20
5. With just positive numbers the errors are still possible but you need to use negative exponents. And there you meet the rounding threshold for the IEEE 754 standard
6. The best defense was using by ascending absolute value this was because it started with the small terms, letting them accumulate in the hope of reaching a total sum with enough value to not get wipeout error.
7. Ascending absolute value sorting can still lead to ordering in the summation at each step, ie there can still be big exponent. $[|-1|, 1, 1e52]$ 
8. Different summation algorithms can lead to error using DFP because there is a limited number of bits represent a given number, to do a summation the exponents of the elements need to match. To do this the small number is shifted so that it matches the exponent of the big number. This causes wipeout as the mantissa bits are outside the valid 52 bit window. 
#### Lower bound
The lower bound for difference where error manifests: 
$k * s ≥ L × 2^{-53}$ 
The error manifests when the combined value of the small terms (k·s) is above the rounding floor of L, which is L × 2⁻⁵³, because if k·s falls above this floor, the final addition (k·s + L) does change the value of L once rounded to the nearest representable double.