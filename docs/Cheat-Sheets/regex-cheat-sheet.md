# Regex Cheat Sheet

Regular expressions are a pattern-matching language built into Python via the `re` module.
For data scientists, they are the primary tool for extracting structured information from
unstructured text — parsing log files, cleaning messy columns, validating formats, and
tokenizing strings before NLP pipelines.

---

## Core re Functions

### re.search()

Scans through the entire string and returns the first match object, or `None` if no match.
Use this when you want to know whether a pattern exists anywhere in a string.

```python
import re

text = "Order placed on 2024-03-15 for customer #4892"
match = re.search(r'\d{4}-\d{2}-\d{2}', text)

if match:
    print(match.group())   # Output: 2024-03-15
    print(match.start())   # Output: 16
    print(match.end())     # Output: 26
```

### re.match()

Only matches at the **beginning** of the string. Returns `None` if the pattern does not
start at position 0. Use this for validating that a string starts with a specific format.

```python
log_line = "ERROR 2024-03-15 disk full"
match = re.match(r'(ERROR|WARN|INFO)', log_line)

if match:
    print(match.group())  # Output: ERROR

# Does NOT match if pattern is not at the start
result = re.match(r'\d{4}', log_line)
print(result)             # Output: None
```

### re.fullmatch()

Requires the pattern to match the **entire** string. Use this for strict validation — email
format checks, ZIP codes, identifiers — where partial matches are wrong.

```python
zip_pattern = r'\d{5}(-\d{4})?'

print(re.fullmatch(zip_pattern, '90210'))        # Match object
print(re.fullmatch(zip_pattern, '90210-1234'))   # Match object
print(re.fullmatch(zip_pattern, '90210 USA'))    # Output: None
```

### re.findall()

Returns a list of all non-overlapping matches as strings. If the pattern contains a
capturing group, returns a list of the captured groups instead. Use for bulk extraction.

```python
text = "Temps recorded: 98.6F, 101.2F, 99.8F, 97.4F"
temps = re.findall(r'\d+\.\d+', text)
print(temps)  # Output: ['98.6', '101.2', '99.8', '97.4']

# With a capturing group — returns only the captured part
tags = "price=29.99 qty=5 discount=0.10"
values = re.findall(r'(\w+)=[\d.]+', tags)
print(values)  # Output: ['price', 'qty', 'discount']
```

### re.finditer()

Like `findall()` but returns an iterator of match objects instead of strings. Use this
when you need match positions, span information, or multiple groups per match.

```python
text = "Call 800-555-0100 or 800-555-0199 for support"
for m in re.finditer(r'(\d{3})-(\d{3})-(\d{4})', text):
    print(m.group(), 'at position', m.start())
    # Output: 800-555-0100 at position 5
    # Output: 800-555-0199 at position 19
```

### re.sub()

Replaces all matches of a pattern with a replacement string or the result of a function.
Use for text normalization, scrubbing PII, or reformatting fields.

```python
# Replace multiple whitespace chars with a single space
messy = "Name:  John   Smith   Age:  42"
clean = re.sub(r'\s+', ' ', messy)
print(clean)  # Output: Name: John Smith Age: 42

# Reformat date from MM/DD/YYYY to YYYY-MM-DD using backreferences
date_str = "Invoice date: 03/15/2024"
reformatted = re.sub(r'(\d{2})/(\d{2})/(\d{4})', r'\3-\1-\2', date_str)
print(reformatted)  # Output: Invoice date: 2024-03-15

# Use a function as replacement — mask credit card numbers
def mask(m):
    return 'XXXX-XXXX-XXXX-' + m.group()[-4:]

cc = "Card: 4111-1111-1111-1234"
print(re.sub(r'\d{4}-\d{4}-\d{4}-\d{4}', mask, cc))
# Output: Card: XXXX-XXXX-XXXX-1234
```

### re.split()

Splits a string on every match of the pattern. More powerful than `str.split()` because
the delimiter can be a pattern rather than a fixed string.

```python
# Split on any combination of punctuation and whitespace
text = "apples, oranges;  bananas|grapes"
tokens = re.split(r'[\s,;|]+', text)
print(tokens)  # Output: ['apples', 'oranges', 'bananas', 'grapes']

# Keep the delimiter in the result using a capturing group
parts = re.split(r'(\d+)', 'Item10Qty5Price20')
print(parts)  # Output: ['Item', '10', 'Qty', '5', 'Price', '20', '']
```

---

## Pattern Basics

### Literal characters

Ordinary characters match themselves. Case matters unless `re.IGNORECASE` is set.
Reserved metacharacters — `. ^ $ * + ? { } [ ] \ | ( )` — must be escaped with `\`.

```python
text = "The cost is $9.99 (plus tax)"
# Match literal dollar sign — escape it
match = re.search(r'\$\d+\.\d{2}', text)
print(match.group())  # Output: $9.99
```

### . (any character except newline)

Matches any single character except `\n`. Use `re.DOTALL` to include newlines.

```python
entries = ["2024-03-15", "2024/03/15", "2024.03.15"]
pattern = r'\d{4}.\d{2}.\d{2}'
for e in entries:
    print(re.match(pattern, e).group())
# Output: 2024-03-15
# Output: 2024/03/15
# Output: 2024.03.15
```

### ^ and $ (anchors)

`^` matches the start of the string (or start of each line with `re.MULTILINE`).
`$` matches the end of the string (or end of each line with `re.MULTILINE`).

```python
lines = ["ERROR: disk full", "WARNING: low memory", "ERROR: connection timeout"]
errors = [l for l in lines if re.match(r'^ERROR', l)]
print(errors)
# Output: ['ERROR: disk full', 'ERROR: connection timeout']

# $ to validate format at end
print(bool(re.search(r'\.csv$', 'sales_data.csv')))   # Output: True
print(bool(re.search(r'\.csv$', 'sales_data.csv.bak')))  # Output: False
```

### | (alternation) and () (grouping)

`|` is OR — matches either the left or right side. `()` groups subpatterns and also
creates a capturing group.

```python
text = "Status: APPROVED. Previous status: PENDING."
matches = re.findall(r'(APPROVED|PENDING|REJECTED)', text)
print(matches)  # Output: ['APPROVED', 'PENDING']

# Group to apply quantifier to a phrase
data = "hahahahaha"
print(re.match(r'(ha)+', data).group())  # Output: hahahahaha
```

---

## Character Classes

### [abc] and [^abc] (inclusion and exclusion)

`[abc]` matches any one of the listed characters.
`[^abc]` matches any character NOT in the list.

```python
text = "S3cr3t P@$$w0rd!"
# Keep only alphanumeric
cleaned = re.sub(r'[^a-zA-Z0-9]', '', text)
print(cleaned)  # Output: S3cr3tPw0rd

# Match vowels only
vowels = re.findall(r'[aeiouAEIOU]', "Machine Learning")
print(vowels)  # Output: ['a', 'i', 'e', 'a', 'i']
```

### [a-z], [0-9] (ranges)

Ranges define a contiguous set of characters. You can combine multiple ranges inside one class.

```python
# Validate a simple slug (lowercase letters, digits, hyphens)
def is_valid_slug(s):
    return bool(re.fullmatch(r'[a-z0-9\-]+', s))

print(is_valid_slug('model-v2-final'))   # Output: True
print(is_valid_slug('Model V2 Final'))   # Output: False
```

### \d, \D, \w, \W, \s, \S (shorthand classes)

| Shorthand | Equivalent  | Matches                          |
|-----------|-------------|----------------------------------|
| `\d`      | `[0-9]`     | Any digit                        |
| `\D`      | `[^0-9]`    | Any non-digit                    |
| `\w`      | `[a-zA-Z0-9_]` | Word character                |
| `\W`      | `[^\w]`     | Non-word character               |
| `\s`      | `[ \t\n\r\f\v]` | Whitespace                   |
| `\S`      | `[^\s]`     | Non-whitespace                   |

```python
log = "2024-03-15 ERROR  disk_usage=98%  host=prod-server-01"

# Extract all key=value pairs
pairs = re.findall(r'(\w+)=(\S+)', log)
print(pairs)
# Output: [('disk_usage', '98%'), ('host', 'prod-server-01')]

# Strip all non-digit characters from a phone field
raw_phone = "(800) 555-0100 ext. 42"
digits_only = re.sub(r'\D', '', raw_phone)
print(digits_only)  # Output: 80055501004
```

### Combining classes

Stack shorthands and literals inside `[]` to build precise classes.

```python
# Match a valid Python identifier (letter or underscore, then word chars)
identifiers = ["model_v2", "2bad", "_private", "valid-name"]
for name in identifiers:
    if re.fullmatch(r'[a-zA-Z_]\w*', name):
        print(f"{name}: valid")
    else:
        print(f"{name}: invalid")
# Output:
# model_v2: valid
# 2bad: invalid
# _private: valid
# valid-name: invalid
```

---

## Quantifiers

### * (zero or more) and + (one or more)

`*` matches 0 or more repetitions — the token is optional.
`+` matches 1 or more — at least one occurrence is required.

```python
texts = ["color", "colour", "colr"]
for t in texts:
    m = re.match(r'colou?r', t)      # u is optional with ?
    print(t, '->', bool(m))
# Output:
# color -> True
# colour -> True
# colr -> False

# + requires at least one digit
print(bool(re.search(r'\d+', 'abc')))    # Output: False
print(bool(re.search(r'\d+', 'abc123'))) # Output: True
```

### ? (zero or one) and {n}, {n,m} (exact and range)

`?` makes the preceding token optional.
`{n}` matches exactly n times. `{n,m}` matches between n and m times.

```python
# IPv4 address: each octet 1-3 digits
ip_pattern = r'\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b'
text = "Server at 192.168.1.100 and backup at 10.0.0.1"
ips = re.findall(ip_pattern, text)
print(ips)  # Output: ['192.168.1.100', '10.0.0.1']

# Exactly 4-digit year
years = re.findall(r'\b\d{4}\b', "Events in 2022, 2023, and 20245 were notable")
print(years)  # Output: ['2022', '2023']
```

### Greedy vs. lazy (? suffix)

By default quantifiers are **greedy** — they consume as much as possible.
Adding `?` after a quantifier makes it **lazy** — it matches as little as possible.

```python
html = "<b>bold text</b> and <i>italic text</i>"

# Greedy — matches from first < to last >
greedy = re.findall(r'<.+>', html)
print(greedy)
# Output: ['<b>bold text</b> and <i>italic text</i>']

# Lazy — matches each tag independently
lazy = re.findall(r'<.+?>', html)
print(lazy)
# Output: ['<b>', '</b>', '<i>', '</i>']

# Lazy to extract tag content
content = re.findall(r'<\w+>(.+?)</\w+>', html)
print(content)  # Output: ['bold text', 'italic text']
```

---

## Groups and Capturing

### (group) and group() vs groups()

Parentheses create a capturing group. `match.group(0)` is the full match.
`match.group(1)`, `match.group(2)` etc. are the captured subgroups.
`match.groups()` returns all subgroups as a tuple.

```python
log = "2024-03-15 14:32:07 ERROR user_id=4892 action=login"
m = re.search(r'(\d{4}-\d{2}-\d{2}) (\d{2}:\d{2}:\d{2}) (\w+)', log)

print(m.group(0))   # Output: 2024-03-15 14:32:07 ERROR
print(m.group(1))   # Output: 2024-03-15
print(m.group(2))   # Output: 14:32:07
print(m.group(3))   # Output: ERROR
print(m.groups())   # Output: ('2024-03-15', '14:32:07', 'ERROR')
```

### (?:...) non-capturing group

Groups the pattern for quantifiers or alternation without creating a capture.
Use when you need grouping but don't want to pollute `groups()`.

```python
# Group for alternation, but don't capture the unit
measurements = ["15px", "20em", "100px", "5rem"]
for m_str in measurements:
    m = re.match(r'(\d+)(?:px|em|rem)', m_str)
    if m:
        print(m.group(1), 'units:', m_str)
        # m.groups() returns only ('15',) not ('15', 'px')
# Output:
# 15 units: 15px
# 20 units: 20em
# 100 units: 100px
# 5 units: 5rem
```

### Named groups (?P<name>...)

Assigns a name to a capturing group. Access with `match.group('name')` or
`match.groupdict()`. Makes complex patterns self-documenting and robust to reordering.

```python
log_pattern = re.compile(
    r'(?P<date>\d{4}-\d{2}-\d{2})\s+'
    r'(?P<time>\d{2}:\d{2}:\d{2})\s+'
    r'(?P<level>ERROR|WARN|INFO)\s+'
    r'(?P<message>.+)'
)

line = "2024-03-15 14:32:07 ERROR disk usage exceeded 90%"
m = log_pattern.match(line)

print(m.group('date'))    # Output: 2024-03-15
print(m.group('level'))   # Output: ERROR
print(m.groupdict())
# Output: {'date': '2024-03-15', 'time': '14:32:07', 'level': 'ERROR', 'message': 'disk usage exceeded 90%'}
```

### Backreferences in patterns

`\1`, `\2` inside a pattern refer back to what group 1 or 2 actually matched.
Useful for detecting repeated words or matched delimiters.

```python
# Detect repeated consecutive words
text = "The the sales data is is ready for review"
dupes = re.findall(r'\b(\w+)\s+\1\b', text, re.IGNORECASE)
print(dupes)  # Output: ['the', 'is']

cleaned = re.sub(r'\b(\w+)\s+\1\b', r'\1', text, flags=re.IGNORECASE)
print(cleaned)  # Output: The sales data is ready for review
```

---

## Lookahead and Lookbehind

Lookaround assertions match a position, not characters. They do not consume characters,
so they do not appear in the match result. This is essential when the surrounding context
determines whether a match is valid, but you only want to extract part of it.

### (?=...) positive lookahead

Asserts that what follows the current position matches the pattern, without including it.

```python
text = "price: 100USD, weight: 50KG, distance: 200KM"
# Extract numbers followed by a unit
numbers = re.findall(r'\d+(?=USD|KG|KM)', text)
print(numbers)  # Output: ['100', '50', '200']
```

### (?!...) negative lookahead

Asserts that what follows does NOT match. Use to exclude specific suffixes.

```python
# Match 'python' not followed by '2'
versions = ["python3", "python2", "python3.10", "python"]
for v in versions:
    if re.search(r'python(?!2)', v):
        print(v)
# Output:
# python3
# python3.10
# python
```

### (?<=...) positive lookbehind

Asserts that what precedes the current position matches the pattern.
The lookbehind pattern must be fixed-width.

```python
text = "Revenue: $4500, Cost: $1200, Profit: $3300"
# Extract numbers after the dollar sign, without capturing $
amounts = re.findall(r'(?<=\$)\d+', text)
print(amounts)  # Output: ['4500', '1200', '3300']
```

### (?<!...) negative lookbehind

Asserts that what precedes does NOT match. Useful for disambiguating similar patterns.

```python
entries = ["99 bottles", "0.99 price", "2.99 price", "100 items"]
# Match integers not preceded by a decimal point or digits.digits
for entry in entries:
    m = re.search(r'(?<!\d\.)\b\d+\b', entry)
    if m:
        print(entry, '->', m.group())
# Output:
# 99 bottles -> 99
# 100 items -> 100
```

---

## Flags

Flags modify how the regex engine interprets the pattern. Pass as a second argument to
`re` functions, or combine with `|`.

### re.IGNORECASE (re.I)

Makes the pattern case-insensitive. Affects `[a-z]`, `\w`, and literal characters.

```python
text = "Python PYTHON python PyThOn"
matches = re.findall(r'python', text, re.IGNORECASE)
print(matches)  # Output: ['Python', 'PYTHON', 'python', 'PyThOn']
```

### re.MULTILINE (re.M)

Makes `^` match at the start of each line and `$` match at the end of each line,
instead of only the start and end of the full string.

```python
log = """INFO server started
ERROR disk full
INFO request received
ERROR timeout"""

errors = re.findall(r'^ERROR.+', log, re.MULTILINE)
print(errors)
# Output: ['ERROR disk full', 'ERROR timeout']
```

### re.DOTALL (re.S)

Makes `.` match any character including `\n`. Use when your pattern needs to span lines.

```python
html = """<div>
  First line
  Second line
</div>"""

# Without DOTALL — does not match across newlines
print(re.search(r'<div>.+</div>', html))           # Output: None

# With DOTALL
m = re.search(r'<div>(.+)</div>', html, re.DOTALL)
print(m.group(1).strip())
# Output:
# First line
#   Second line
```

### re.VERBOSE (re.X)

Allows whitespace and `#` comments inside the pattern. Makes complex patterns readable.
Literal spaces must be escaped as `\ ` or put inside `[]`.

```python
date_pattern = re.compile(r"""
    (?P<year>  \d{4})   # 4-digit year
    [-/.]               # separator: dash, slash, or dot
    (?P<month> \d{2})   # 2-digit month
    [-/.]               # separator
    (?P<day>   \d{2})   # 2-digit day
""", re.VERBOSE)

m = date_pattern.search("Event date: 2024/03/15")
print(m.groupdict())
# Output: {'year': '2024', 'month': '03', 'day': '15'}
```

---

## Compiled Patterns

### re.compile() for reuse

`re.compile()` compiles a pattern string into a regex object. When you apply the same
pattern to thousands of strings — like processing a DataFrame column — compiling once
avoids repeated parsing overhead.

```python
email_re = re.compile(r'[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}')

emails = [
    "Contact us at support@example.com",
    "No email here",
    "Two emails: a@b.com and x@y.org",
]

for line in emails:
    found = email_re.findall(line)
    print(found)
# Output: ['support@example.com']
# Output: []
# Output: ['a@b.com', 'x@y.org']
```

### Compiled pattern methods

A compiled pattern exposes the same methods as the `re` module functions, but without the
pattern argument. This makes call sites cleaner and catches pattern errors at compile time.

```python
log_re = re.compile(
    r'(?P<ip>\d{1,3}(?:\.\d{1,3}){3})'
    r'\s+(?P<method>GET|POST|PUT|DELETE)'
    r'\s+(?P<path>\S+)'
    r'\s+(?P<status>\d{3})'
)

access_log = [
    "192.168.1.1 GET /api/users 200",
    "10.0.0.5 POST /api/login 401",
    "172.16.0.3 DELETE /api/session 204",
]

records = [log_re.match(line).groupdict() for line in access_log]
for r in records:
    print(r)
# Output: {'ip': '192.168.1.1', 'method': 'GET', 'path': '/api/users', 'status': '200'}
# Output: {'ip': '10.0.0.5', 'method': 'POST', 'path': '/api/login', 'status': '401'}
# Output: {'ip': '172.16.0.3', 'method': 'DELETE', 'path': '/api/session', 'status': '204'}
```

---

## Practical Data Cleaning

### Extract email addresses

```python
import re

text = """
Team contacts: alice@datalab.io, bob.smith@corp.example.com,
invalid-email, charlie@, delta@analytics.co.uk
"""

email_re = re.compile(r'\b[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}\b')
emails = email_re.findall(text)
print(emails)
# Output: ['alice@datalab.io', 'bob.smith@corp.example.com', 'delta@analytics.co.uk']
```

### Extract URLs

```python
text = "See https://docs.python.org/3/library/re.html and http://example.com for details."

url_re = re.compile(r'https?://[^\s\)\]>,"\']+')
urls = url_re.findall(text)
print(urls)
# Output: ['https://docs.python.org/3/library/re.html', 'http://example.com']
```

### Extract and normalize phone numbers

```python
raw_phones = [
    "Call (800) 555-0100",
    "Phone: 800.555.0101",
    "Tel +1-800-555-0102",
    "800 555 0103",
]

phone_re = re.compile(r'[\+1\s\-]*\(?\d{3}\)?[\s.\-]+\d{3}[\s.\-]+\d{4}')

for entry in raw_phones:
    m = phone_re.search(entry)
    if m:
        digits = re.sub(r'\D', '', m.group())[-10:]  # last 10 digits
        formatted = f"({digits[:3]}) {digits[3:6]}-{digits[6:]}"
        print(formatted)
# Output: (800) 555-0100
# Output: (800) 555-0101
# Output: (800) 555-0102
# Output: (800) 555-0103
```

### Extract dates in multiple formats

```python
texts = [
    "Report generated on 03/15/2024",
    "Last updated 2024-03-15",
    "Deadline: March 15, 2024",
]

patterns = [
    (r'(\d{2})/(\d{2})/(\d{4})', lambda m: f"{m.group(3)}-{m.group(1)}-{m.group(2)}"),
    (r'(\d{4})-(\d{2})-(\d{2})', lambda m: m.group()),
    (r'(\w+ \d{1,2},? \d{4})',    lambda m: m.group()),
]

for text in texts:
    for pattern, formatter in patterns:
        m = re.search(pattern, text)
        if m:
            print(formatter(m))
            break
# Output: 2024-03-15
# Output: 2024-03-15
# Output: March 15, 2024
```

### Remove HTML tags

```python
html = """
<h1>Sales Report</h1>
<p>Total revenue: <b>$4,500</b> across <i>3 regions</i>.</p>
"""

tag_re = re.compile(r'<[^>]+>')
clean = tag_re.sub('', html)
# Collapse extra whitespace
clean = re.sub(r'\n+', '\n', clean).strip()
print(clean)
# Output:
# Sales Report
# Total revenue: $4,500 across 3 regions.
```

---

## Pandas Integration

### .str.contains(regex=True)

Filter rows where a column matches a pattern. `na=False` prevents errors on null values.

```python
import re
import pandas as pd

df = pd.DataFrame({'email': [
    'alice@company.com',
    'bob@gmail.com',
    'charlie@company.org',
    None,
    'dave@company.com',
]})

company_users = df[df['email'].str.contains(r'@company\.(com|org)', na=False)]
print(company_users)
# Output:
#                  email
# 0    alice@company.com
# 2  charlie@company.org
# 4     dave@company.com
```

### .str.extract()

Extract the first match of a pattern with capturing groups into separate columns.
Returns `NaN` for rows with no match.

```python
df = pd.DataFrame({'log': [
    '2024-03-15 ERROR disk full',
    '2024-03-16 INFO backup complete',
    '2024-03-17 WARN memory high',
]})

extracted = df['log'].str.extract(r'(\d{4}-\d{2}-\d{2})\s+(ERROR|WARN|INFO)\s+(.+)')
extracted.columns = ['date', 'level', 'message']
print(extracted)
# Output:
#          date  level          message
# 0  2024-03-15  ERROR        disk full
# 1  2024-03-16   INFO  backup complete
# 2  2024-03-17   WARN      memory high
```

### .str.extractall()

Extract all matches from each row. Returns a multi-index DataFrame where the second
index level is the match number. Use when a single cell may contain multiple values.

```python
df = pd.DataFrame({'text': [
    'Contact: a@x.com or b@y.com',
    'Email c@z.org for help',
    'No email here',
]})

all_emails = df['text'].str.extractall(r'([a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,})')
all_emails.columns = ['email']
print(all_emails)
# Output:
#           email
#   match
# 0 0      a@x.com
#   1      b@y.com
# 1 0      c@z.org
```

### .str.replace() with regex

Replace pattern matches across an entire column. Set `regex=True` explicitly — the
default changed in newer pandas versions and leaving it unset raises a warning.

```python
df = pd.DataFrame({'phone': [
    '(800) 555-0100',
    '800.555.0101',
    '800 555 0102',
]})

df['phone_clean'] = df['phone'].str.replace(r'\D', '', regex=True)
print(df)
# Output:
#             phone  phone_clean
# 0  (800) 555-0100   8005550100
# 1    800.555.0101   8005550101
# 2    800 555 0102   8005550102
```

### .str.findall()

Returns a list of all matches per row. Useful for counting occurrences or collecting
multiple values from a free-text column.

```python
df = pd.DataFrame({'review': [
    'Great product! 5 stars. Would buy again.',
    'Terrible. 1 star. Never again. 0 out of 10.',
    'Nice. 4 stars.',
]})

df['numbers_mentioned'] = df['review'].str.findall(r'\b\d+\b')
print(df[['review', 'numbers_mentioned']])
# Output:
#                                          review numbers_mentioned
# 0    Great product! 5 stars. Would buy again.               ['5']
# 1  Terrible. 1 star. Never again. 0 out of 10.       ['1', '0', '10']
# 2                                 Nice. 4 stars.              ['4']
```

---

## Common Gotchas

### Always use raw strings for patterns

Without `r''`, Python processes backslash escapes before the regex engine sees them.
`\n` becomes a newline, `\d` becomes a literal `d` (since `\d` is not a Python escape).
Always prefix pattern strings with `r`.

```python
import re

# Wrong — \d is not a Python escape so it becomes 'd' in some contexts
# but \b is a backspace character (\x08), not a word boundary
bad = re.findall('\b\d+\b', 'Score: 42')
print(bad)   # Output: [] — \b matched backspace, not a word boundary

# Correct
good = re.findall(r'\b\d+\b', 'Score: 42')
print(good)  # Output: ['42']
```

### Escaping special characters in dynamic patterns

When building a pattern from user input or variable data, escape it with `re.escape()`
to prevent special characters from being interpreted as metacharacters.

```python
# Safe search for a literal user-supplied string
query = "price (USD)"  # parentheses are metacharacters
text = "The price (USD) is 42"

# Wrong — raises re.error: unbalanced parenthesis
# re.search(query, text)

# Correct
safe_query = re.escape(query)
print(safe_query)  # Output: price\ \(USD\)
print(bool(re.search(safe_query, text)))  # Output: True
```

### Greedy quantifiers consuming too much

Greedy matching is the most common source of incorrect extractions. When a pattern
like `.*` appears between two markers, it will always try to jump to the last marker,
not the first one.

```python
html = '<a href="first.html">Link 1</a> and <a href="second.html">Link 2</a>'

# Greedy — matches from first <a to last </a>
greedy = re.findall(r'<a href=".*">', html)
print(greedy)
# Output: ['<a href="first.html">Link 1</a> and <a href="second.html">']

# Lazy — stops at first closing "
lazy = re.findall(r'<a href=".*?">', html)
print(lazy)
# Output: ['<a href="first.html">', '<a href="second.html">']

# Better — use a negated class to forbid the delimiter inside the match
negated = re.findall(r'<a href="([^"]+)">', html)
print(negated)
# Output: ['first.html', 'second.html']
```

### Unicode and \w behavior

By default in Python 3, `\w` matches Unicode word characters — not just ASCII letters.
This is usually what you want, but it can cause surprises when cleaning multilingual text.

```python
text = "café résumé naïve"

# \w includes accented characters in Python 3
words = re.findall(r'\w+', text)
print(words)  # Output: ['café', 'résumé', 'naïve']

# To restrict to ASCII only, use re.ASCII flag (or re.A)
ascii_words = re.findall(r'\w+', text, re.ASCII)
print(ascii_words)  # Output: ['caf', 'r', 'sum', 'na', 've']
# Note: accented chars are split at non-ASCII boundaries
```

### re.match() vs re.search() — the silent trap

`re.match()` only checks the beginning of the string. Forgetting this is a frequent bug
when validating fields — a string that starts with garbage will fail, but a string where
the pattern only appears later will also silently fail.

```python
text = "Note: the ID is ABC-12345"

# This silently returns None — the pattern is not at position 0
m = re.match(r'[A-Z]+-\d+', text)
print(m)  # Output: None

# Use search to find it anywhere
m = re.search(r'[A-Z]+-\d+', text)
print(m.group())  # Output: ABC-12345

# Use fullmatch when the entire string must conform
print(re.fullmatch(r'[A-Z]+-\d+', 'ABC-12345'))  # Match object
print(re.fullmatch(r'[A-Z]+-\d+', text))          # Output: None
```

---

> [!tip]
> Build complex patterns incrementally using `re.VERBOSE`. Test each subpattern in isolation before combining. Tools like regex101.com let you visualize match groups interactively — paste the pattern and test strings to debug before committing to code.

> [!warning]
> Never use regex to parse HTML or XML for production work. Use `BeautifulSoup` or `lxml` instead. Regex is appropriate for extracting simple values from known HTML snippets, but breaks on nested tags, attributes, and encoding edge cases.

> [!success]
> The fastest path to a working regex: start with the most restrictive pattern that matches valid inputs, then relax constraints one at a time to handle edge cases. Starting too broad (like `.*`) and narrowing leads to harder-to-debug patterns.
