---
layout: default
title: Query Filtering
---

# Query Filtering

## Query Filters

You can restrict [extract methods](/reading/) and [conversion methods](/converting/) output by setting query options. To set those options you will need to use the methods described below. But keep in mind that:

* The query options methods are all chainable *except when they have to return a boolean*;
* The query options methods can be call in any sort of order before any extract/conversion method;

## Modifying content methods

### stripBOM($status)

`stripBom` only argument `$status` must be a `boolean`. This method specifies if the [BOM sequence](/bom/) must be removed or not from the CSV's first cell of the first row. The actual stripping will take place only if a BOM sequence is detected and the first row is selected in the resultset **or** if its offset is used as the first argument of the `Reader::fetchAssoc` method.

<p class="message-warning">The BOM sequence is never removed from the CSV document, it is only stripped from the resultset.</p>

## Modifying Extract methods return type

<p class="message-notice">new feature introduced in <code>version 8.0</code></p>

Sometimes you may need to modify the return type for some extract methods. To do so, the library exposes one new method:

- `setReturnType` : Set the return type for the next extract method call;

The return type must be one of the following constant:

- `AbstractCsv::TYPE_ARRAY`: to set the return type to be an `Array`;
- `AbstractCsv::TYPE_ITERATOR`: to set the return type to be an `Iterator`;

By default, and to preserve backward compatibility the return type is set to `Reader::TYPE_ARRAY`.

~~~php
$reader->setReturnType(Reader::TYPE_ITERATOR);
$result = $reader->fetchAssoc(); //$result is an iterator
$result = $reader->fetchAssoc(); //$result is an array
//everytime a query is issued the return type is resetted to Reader::TYPE_ARRAY
~~~

<p class="message-warning"><strong>WARNING:</strong> Not all the extract methods are affected by this mechanism but any call to any extract/conversion method will automatically reset the return type to <code>AbstractCsv::TYPE_ARRAY</code></p>

## Filtering methods

The filtering options **are the first settings applied to the CSV before anything else**. The filters follow the *First In First Out* rule.

### addFilter(callable $callable)

The `addFilter` method adds a callable filter function each time it is called. The function can take up to three parameters:

* the current csv row data;
* the current csv key;
* the current csv iterator object;

### removeFilter(callable $callable)

`removeFilter` method removes an already registered filter function. If the function was registered multiple times, you will have to call `removeFilter` as often as the filter was registered. **The first registered copy will be the first to be removed.**

### hasFilter(callable $callable)

`hasFilter` method checks if the filter function is already registered

### clearFilter()

`clearFilter` method removes all registered filter functions.

## Sorting methods

The sorting options are applied **after the CSV filtering options**. The sorting follow the *First In First Out* rule.

<p class="message-warning">To sort the data <code>iterator_to_array</code> is used which could lead to performance penalty if you have a heavy CSV file to sort
</p>

### addSortBy(callable $callable)

`addSortBy` method adds a sorting function each time it is called. The function takes exactly two parameters which will be filled by pairs of rows.

### removeSortBy(callable $callable)

`removeSortBy` method removes an already registered sorting function. If the function was registered multiple times, you will have to call `removeSortBy` as often as the function was registered. **The first registered copy will be the first to be removed.**

### hasSortBy(callable $callable)

`hasSortBy` method checks if the sorting function is already registered

### clearSortBy()

`clearSortBy` method removes all registered sorting functions.

## Interval methods

The methods enable returning a specific interval of CSV rows. When called more than once, only the last filtering settings is taken into account. The interval is calculated **after filtering and/or sorting but before extracting the data**.

### setOffset($offset = 0)

`setOffset` method specifies an optional offset for the return data. By default the offset equals `0`.

### setLimit($limit = -1)

`setLimit` method specifies an optional maximum rows count for the return data. By default the offset equals `-1`, which translate to all rows.

<p class="message-warning">Both methods have no effect on the <code>fetchOne</code> method output.</p>

## Examples

### Modifying extract methods output

Here's an example on how to use the query features of the `Reader` class to restrict the `fetchAssoc` result:

~~~php
function filterByEmail($row)
{
    return filter_var($row[2], FILTER_VALIDATE_EMAIL);
}

function sortByLastName($rowA, $rowB)
{
    return strcmp($rowB[1], $rowA[1]);
}

$data = $reader
    ->stripBom(false)
    ->setOffset(3)
    ->setLimit(2)
    ->addFilter('filterByEmail')
    ->addSortBy('sortByLastName')
    ->fetchAssoc(['firstname', 'lastname', 'email'], function ($value) {
    return array_map('strtoupper', $value);
});
// data length will be equals or lesser that 2 starting from the row index 3.
// will return something like this :
//
// [
//   ['firstname' => 'JANE', 'lastname' => 'RAMANOV', 'email' => 'JANE.RAMANOV@EXAMPLE.COM'],
//   ['firstname' => 'JOHN', 'lastname' => 'DOE', 'email' => 'JOHN.DOE@EXAMPLE.COM'],
// ]
//
~~~

### Modifying conversion methods output

Starting with `version 7.0`, the query options can also modify the output from the conversion methods as shown below with the `toHTML` method.

~~~php
function filterByEmail($row)
{
    return filter_var($row[2], FILTER_VALIDATE_EMAIL);
}

function sortByLastName($rowA, $rowB)
{
    return strcmp($rowB[1], $rowA[1]);
}

$data = $reader
    ->stripBom(true)
    ->setOffset(3)
    ->setLimit(2)
    ->addFilter('filterByEmail')
    ->addSortBy('sortByLastName')
    ->toHTML("simple-table");
// $data contains the HTML table code equivalent to:
//
//<table class="simple-table">
//  <tr><td>JANE</td><td>RAMANOV</td><td>JANE.RAMANOV@EXAMPLE.COM</td></tr>
//  <tr><td>JOHN</td><td>DOE</td><td>JOHN.DOE@EXAMPLE.COM</td></tr>
//</table>
//
~~~