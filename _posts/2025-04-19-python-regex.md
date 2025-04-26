---
layout: single
title:  "Python: Regular Expressions"
categories: Programming
tags: Python
show_date: true
toc: true
toc_sticky: true
header:
  teaser: /assets/images/python.png
author:
  name     : "Python"
  avatar   : "/assets/images/python.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar
---

# Using Regular Expressions to Validate an IP Address

#### Example: Regex Number Range 0-255

 In this range have one, two and three digit numbers. 

Now our 

single digit numbers are 0-9, [0-9]

two digit numbers are 10-99 , [1-9]  [0-9]

but three digit numbers are 100-255.  

The three digit  numbers need to be split futher into 100-199  and 200-255, and the latter further into 200-249  and 250-255 . Combining all these with the  alternation operator results in: 
` ([0-9]|[1-9]  [0-9]|1[0-9]{2}|2 [0-4]  [0-9]|25[0-5]))` or               
`([0-9]|[1-9] [0-9]|1 [0-9] [0-9]|2[0-4] [0-9]|25[0-5])`.

```python
# write a function with regex to validate a number is a valid IP octet
import re
def is_valid_octet(octet):
    """
    Validate an octet of an IP address using regular expression.
    """
    #pattern = "^(25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9]?[0-9])$"
    pattern = "^([0-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])$"

    return bool(re.match(pattern, octet))
#Test cases
print(is_valid_octet("0"))
print(is_valid_octet("255"))
print(is_valid_octet("256"))
print(is_valid_octet("01"))
print(is_valid_octet("10"))
print(is_valid_octet("10.20"))
print(is_valid_octet("10.20.30"))   
```

### Output

```
True
True
False
False
True
False
False

```

```
def is_valid_ip(ip):
    """
    Validate an IP address using regular expression.
    """
    # Split the IP address into octets
    octets = ip.split('.')
    
    # Check if there are exactly four octets
    if len(octets) != 4:
        return False
    
    # Check each octet
    for octet in octets:
        # Check if the octet is a digit and within the range of 0 to 255
        if not is_valid_octet(octet):
            return False
        
        # Check if the octet does not have leading zeros
        if len(octet) > 1 and octet[0] == '0':
            return False
    
    return True

# Test cases
print(is_valid_ip("10.20.300.1"))

```

### Output

```
False
```



and a another way of implementing the same functionality

```py
# Write a python fuction to validate a given IP address
# The function should return True if the IP address is valid, and False otherwise
# An IP address is valid if it consists of four octets, each ranging from 0 to 255, separated by dots.
# For example, the following IP addresses are valid:
def is_valid_ip(ip):
    # Split the IP address into octets
    octets = ip.split('.')
    
    # Check if there are exactly four octets
    if len(octets) != 4:
        return False
    
    # Check each octet
    for octet in octets:
        # Check if the octet is a digit and within the range of 0 to 255
        if not octet.isdigit() or not (0 <= int(octet) <= 255):
            return False
        
        # Check if the octet does not have leading zeros
        if len(octet) > 1 and octet[0] == '0':
            return False
    
    return True
# Test cases
print(is_valid_ip("10.20.30.01"))
print(is_valid_ip("10.20.30.256"))
print(is_valid_ip("10.20.30"))
print(is_valid_ip("0.02.0.0"))

def validate_list_of_ips(ip_list):
    """
    Validate a list of IP addresses.
    """
    return [ip for ip in ip_list if is_valid_ip(ip)]
# Test cases
print(validate_list_of_ips(["10.20.30.1","10.0.0.0","10.20.30.256","10.20.30"]))

def find_valid_ips(ip_list):
    """
    Find valid IP addresses from a list.
    """
    return list((filter(lambda ip: is_valid_ip(ip), ip_list)))
# Test cases
print(find_valid_ips(["10.20.30.1","10.0.0.0","10.20.30.256","10.20.30"]))

# write a regular expression to validate an IP address
#simplify the regex to validate an IP address
import re
def is_valid_ip_regex(ip):
    """
    Validate an IP address using regular expression.
    """
    pattern = "^((25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9]?[0-9])\.){3}(25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9]?[0-9])$"
    return bool(re.match(pattern, ip))

print(is_valid_ip_regex("10.20.30.256"))

```
### Output
```
False
False
False
False
['10.20.30.1', '10.0.0.0']
['10.20.30.1', '10.0.0.0']
False
```

