**bizdays** computes business days between two dates based on the definition of nonworking days (usually holidays and weekends—nonworking weekdays).
It also computes other collateral effects like adjust dates for the next or previous business day, check whether a date is a business day, create generators of business days sequences, and so forth.

## Install

**bizdays** is avalilable at PyPI, so it is pip instalable.

	pip install bizdays

## Using

Business days calculations are done defining a `Calendar` object.

```python
from bizdays import Calendar
cal = Calendar(holidays, ['Sunday', 'Saturday'])
```

where `holidays` is a sequence of dates which represents nonworking dates and the second argument, `weekdays`, is a sequence with nonworking weekdays.
`holidays` must be a sequence of strings with ISO formatted dates or `datetime.date` objects and `weekdays` a sequence of weekdays in words.

Once you have a `Calendar` you can

```python
>>> cal.isbizday('2014-01-12')
False
>>> cal.isbizday('2014-01-13')
True
>>> cal.bizdays('2014-01-13', '2015-01-13')
253
>>> cal.adjust_next('2015-12-25')
datetime.date(2015, 12, 28)
>>> cal.adjust_next('2015-12-28')
datetime.date(2015, 12, 28)
>>> cal.adjust_previous('2014-01-01')
datetime.date(2013, 12, 31)
>>> cal.adjust_previous('2014-01-02')
datetime.date(2014, 1, 2)
>>> cal.seq('2014-01-02', '2014-01-07')
<generator object seq at 0x1092b02d0>
>>> list(cal.seq('2014-01-02', '2014-01-07'))
[datetime.date(2014, 1, 2), datetime.date(2014, 1, 3), datetime.date(2014, 1, 6), datetime.date(2014, 1, 7)]
>>> cal.offset('2014-01-02', 5)
datetime.date(2014, 1, 9)
>>> cal.getdate('15th day', 2002, 5)
datetime.date(2002, 5, 15)
>>> cal.getdate('15th bizday', 2002, 5)
datetime.date(2002, 5, 22)
>>> cal.getdate('last wed', 2002, 5)
datetime.date(2002, 5, 29)
>>> cal.getdate('first fri before last day ', 2002, 5)
datetime.date(2002, 5, 24)
```

In this example I used the list of holidays released by [ANBIMA](http://www.anbima.com.br/feriados/feriados.asp).

> **Important note on date arguments and returning dates**
> 
> As you can see in the examples all date arguments are strings ISO formatted (`YYYY-mm-dd` or `%Y-%m-%d`), but they can also be passed as `datetime.date` objects.
> All returning dates are `datetime.date` objects (or a sequence of it), unless you set `iso=True`, that will return an ISO formatted string.

> The `startdate` and `enddate` of a `Calendar` are defined accordingly the first and last given holidays.

### bizdays

To compute the business days between two dates you call `bizdays` passing `from` and `to` dates as arguments.

```python
>>> cal.bizdays('2012-12-31', '2013-01-03')
2
```

### getdate

You specify dates by its position or related to other dates, for example:

```python
>>> cal.getdate('15th day', 2002, 5)
datetime.date(2002, 5, 15)
```

it returns the 15th day of 2002 may. You can also reffer to the whole year.

```python
>>> cal.getdate('150th day', 2002)
datetime.date(2002, 5, 30)
```

It accepts `day`, `bizday` and weekdays by: `sun`, `mon`, `tue`, `wed`, `thu`, `fri`, and `sat`.

```python
>>> cal.getdate('last day', 2006)
datetime.date(2006, 12, 31)
>>> cal.getdate('last bizday', 2006)
datetime.date(2006, 12, 29)
>>> cal.getdate('last mon', 2006)
datetime.date(2006, 12, 25)
```

For postion use: `first`, `second`, `third`, `1st`, `2nd`, `3rd`, `[n]th`, and `last`.

#### Using date postions as a reference

You can find before and after other date positions (using date positions as a reference).

```python
>>> cal.getdate('last mon before 30th day', 2006, 7)
datetime.date(2006, 7, 24)
>>> cal.getdate('second bizday after 15th day', 2006)
datetime.date(2006, 1, 18)
```

#### following and preceding

Several contracts, by default, always expiry in the same day, for example, 1st Januray, which isn't a business day, so instead of carrying your code with awful checks you could call `following` which returns the given date
whether it is a business day or the next business day.

```python
>>> cal.following('2013-01-01')
datetime.date(2013, 1, 2)
>>> cal.following('2013-01-02')
datetime.date(2013, 1, 2)
```

We also have `preceding`, although I suppose it is unusual, too.

```python
>>> cal.preceding('2013-01-01')
datetime.date(2012, 12, 31)
```

#### modified_following and modified_preceding

`modified_following` and `modified_preceding` are common functions used to specify maturity of contracts.
They work the same way `following` and `preceding` but once the returning date is a different month it is adjusted to the `following` or `preceding` business day in the same month.

```python
>>> dt = cal.getdate('last day', 2002, 3)
>>> dt
datetime.date(2002, 3, 31)
>>> cal.modified_following(dt, iso=True)
'2002-03-28'
>>> cal.isbizday('2002-03-29')
False
>>> dt = cal.getdate('first day', 2002, 6)
>>> dt
datetime.date(2002, 6, 1)
>>> cal.modified_preceding(dt, iso=True)
'2002-06-03'
```

### seq

To execute calculations through sequential dates, sometimes you must consider only business days.
For example, you want to compute the price of a bond from its issue date up to its maturity.
You have to walk over business days in order to carry the contract up to maturity.
To accomplish that you use the `seq` method (stolen from R) which returns a sequence generator of business days.

```python
>>> for dt in cal.seq('2012-12-31', '2013-01-03'):
...     print dt
... 
2012-12-31
2013-01-02
2013-01-03
```

### offset

This method offsets the given date by `n` days respecting the calendar, so it obligatorily returns a business day.

```python
>>> cal.offset('2013-01-02', 1)
datetime.date(2013, 1, 3)
>>> cal.offset('2013-01-02', 3)
datetime.date(2013, 1, 7)
>>> cal.offset('2013-01-02', 0)
datetime.date(2013, 1, 2)
```

Obviously, if you want to offset backwards you can use `-n`.

```python
>>> print cal.offset('2013-01-02', -1)
2012-12-31
>>> print cal.offset('2013-01-02', -3)
2012-12-27
```

Once the given date is a business day there is no problems, but if instead it isn't a working day the offset can lead to unexpected results. For example:

```python
>>> cal.offset('2013-01-01', 1)
datetime.date(2013, 1, 2)
>>> cal.offset('2013-01-01', 0)
datetime.date(2013, 1, 1)
>>> cal.offset('2013-01-01', -1)
datetime.date(2012, 12, 31)
```

## Actual Calendar

The Actual Calendar can be defined as

```python
>>> cal = Calendar(name='actual')
>>> cal
Calendar: actual
Start: 1970-01-01
End: 2071-01-01
Holidays: 0
Financial: True
```

The Actual Calendar doesn't consider holidays, nor nonworking weekdays for counting business days between 2 dates.
This is the same of subtracting 2 dates, and adjust methods will return the given argument.
But the idea of using the Actual Calendar is working with the same interface for any calendar you work with.
When you price financial instruments you don't have to check if it uses business days or not.

> `startdate` and `enddate` defaults to `1970-01-01` and `2071-01-01`, but they can be set during Calendar's instanciation.

## Vectorized operations

The Calendar's methods: `isbizday`, `bizdays`, `adjust_previous`, `adjust_next`, and `offset`, have a vectorized counterparty, inside `Calendar.vec` attribute.

```python
>>> cal = Calendar.load('Test.cal')
>>> dates = ('2002-01-01', '2002-01-02', '2002-01-03')
>>> tuple(cal.vec.adjust_next(dates))
(datetime.date(2002, 1, 2),
 datetime.date(2002, 1, 2),
 datetime.date(2002, 1, 3))
>>> list(cal.vec.bizdays('2001-12-31', dates))
[0, 1, 2]
```

These functions accept sequences and single values, but always return generators.
In `bizdays` call a date and a sequence have been passed, computing business days between that date and all the others.

### Recycle rule

Once you pass 2 sequences for `bizdays` and `offset` and those sequences doesn't have the same length, no problem.
The shorter collection is cycled to fit the longer's length.

```python
>>> dates = ('2002-01-01', '2002-01-02', '2002-01-03', '2002-01-04', '2002-01-05')
>>> tuple(cal.vec.offset(dates, (1, 2, 3)))
(datetime.date(2002, 1, 3),
 datetime.date(2002, 1, 4),
 datetime.date(2002, 1, 8),
 datetime.date(2002, 1, 7),
 datetime.date(2002, 1, 9))
```

> These methods work well with sequences but not with generators, since I haven't found an easy way to find out which generator is the shorter.