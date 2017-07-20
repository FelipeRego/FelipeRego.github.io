---
title: 'Calculating Compound Annual Growth Rates (CAGR) in Python'
layout: post
date: 2016-08-10
permalink: /blog/2016/08/10/CAGR-Function-In-Python
tags:
  - Jupyter
  - Functions
  - CAGR
  - Python
---

Compound Annual Growth Rates, also known as GAGR, is a popular metric in the financial and business worlds. CAGR measures the mean growth rates of money or units / quantities of something over the years. In finance, it is widely used to calculate investment returns over a period of time (more than one year longer). But while the concepts is mainly used in finance and investment settings, CAGR can also be used to calculate growth related to sales quantities, website user/visitors, etc. Ultimately, it can be a great calculation to understand growth rates of things over a long period of time.

The formula for CAGR is defined as:


$$ CAGR = \left ( \frac{L}{F} \right )^{\left ( \frac{1}{N} \right )}-1 $$


From the formula above you can see that to compute CAGR, you need three parameters or inputs:

**F** = F here means the First value in your series of values.

**L** = L here means the Last value in your series of values.

**N** = N is the number of years between your First and Last value in your series of values.

Note in the formula above that it contains an exponential term. This term helps offset any volatility over the period analysed and it assumes the values or quantities are compounded over the period of time. In essence, CAGR is a metric that describes the growth rate of a value or unit over a period of time if it had grown at a steady rate.

For example, let's imagine you invested $\$1,000 of your money in a bunch of shares 6 years ago. And let's suppose you did not invest again in those same shares since. Today (or six years later), the value of that investment is worth $1,980.

The $1,000 would be the **F** in our formula, **L** would be $1,980 and **N** would be 6 (number of years).

You'd like to know what the compound growth rate was for that investment over that period of time. Since we now know the formula for CAGR, we can compute the compound growth rate of our investment:


```python
CAGR = (1980.0/1000)**(1/6.0)-1
print ('Your investment had a CAGR of {:.2%} '.format(CAGR))
```

    Your investment had a CAGR of 12.06% 


The above output suggests that our investment of $\$1,000 six years ago had a growth rate of 12% each year. To see how this works, try and write down $1,000 and then multiply it by 1.1206. Do the same for the next five resulting values and you should get exactly $1,980 (or thereabouts given decimal points).

That is, 12% is the rate of growth that would take you to the ending value, from the starting value, in the number of years given, if growth had been at the same rate every year. (In reality, growth rates are rarely constant).

Additionally, you could say that 12% growth every year is not a bad return afterall. It would be wise to compare this growth rate with something else (which is one of the powers of CAGR, that is the ability to make comparison of growth rates for different businesses or investments within the same industry, for example). We're not going to focus on that in this article (but if you want to do so think along the lines of [Opportunity Cost](https://en.wikipedia.org/wiki/Opportunity_cost)).

Now, in python we could create a custom-made function to calculate CAGR. Because if you think about it, the way we firstly structured the syntax above, it required you to pass the values rather manually and we also constructed the formula from the scratch which you'd have to do it again and again everytime you wanted to calculate CAGR for other values. This is far from ideal.

Gladly, defining custom functions in python to calculate CAGR for any set of values is a breeze. The function will always require the three inputs to calculate CAGR but you won't have to write it from scratch all over again. We'll apply it in the context of any set of values.

Let's also ask our users to type the parameters needed to calculate CAGR. This way, it gives users the ability to try different numbers or to re-use our function for multiple scenarios (which is one of the many benefits of functions):


```python
# Below we ask the user to tell us what was the initial value / quantity to be considered,
# the end or last value achieved and the period that it took from the first to the last value.

first = float(raw_input('What is the value / quantity you started with: '))
last = float(raw_input('What is the value / quantity you ended up with: '))
periods = float(raw_input('How many years have passed from beginning to the end of your series: '))
    
def CAGR(first, last, periods):
    return (last/first)**(1/periods)-1

print ('Your investment had a CAGR of {:.2%} '.format(CAGR(first, last, periods)))
```

    What is the value / quantity you started with: 1000.0
    What is the value / quantity you ended up with: 1980
    How many years have passed from beginning to the end of your series: 6.0
    Your investment had a CAGR of 12.06% 


The function above provides us with the same result as above. But if I were to repeat the process with a new set of numbers, I could re-run the code above and input different values and I would, consequently, end up with CAGR for those new numbers. Try it out for yourself!

Today we discussed a very quick example using python functions to calculate growth rates using CAGR.  
