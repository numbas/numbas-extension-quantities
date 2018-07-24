# Quantities extension for Numbas

This extension wraps thr [js-quantities](https://github.com/gentooboontoo/js-quantities) library to provide a "quantity with units" data type to Numbas.

It provides a `quantity` data type, which represents a scalar *amount* and a list of *units*.

**A note about precision:** Amounts are represented with JavaScript floating-point numbers, which are only precise to around 30 decimal places. It's our intention to add support for more precise number representations to Numbas; when that happens, this extension could be updated to use that.

## JME functions

### `quantity([number],units)` or `qty([number],units)`

Create a quantity with the given units. If the number is not given, the returned quantity represents 1 of the given units.
Use `*` and `/` to combine units, and `^` for powers of units.
An empty `units` string will produce a unitless quantity.

**Examples:**

* `quantity("kg")`
* `qty("kg")`
* `quantity(1.2,"kg")`
* `quantity("kg*m/s^2")`
* `quantity("1/s")`
* `quantity("kg*m^-2")`

### `units_of_kind(kind)`

Returns a list of [recognised units](#recognised-units) of the given kind.

**Example:**

* `units_of_kind("length")` → `[ "angstrom", "AU", "datamile", "fathom", "foot", "furlong", "inch", "league", "light-minute", "light-second", "light-year", "meter", "mil", "mile", "naut-mile", "parsec", "pica", "point", "redshift", "rod", "yard", "m" ]`

### `aliases(unit)`

Returns a list of accepted names for the given unit.

**Example:**

* `aliases("meter")` → `[ "m", "meter", "meters", "metres", "metres" ]`

### `compatible(q1,q2)`

Are the two given quantities compatible? That is, are they of the same kindm, so one can be converted to the other?

**Examples:**

* `compatible(qty("m"),qty("ft"))` → `true`
* `compatible(qty("m"),qty("kg"))` → `false`

### `kind(quantity)`

What [kind of unit](#recognised-units) is `quantity` measured in? 
Returns a string code corresponding to the kind.
For combinations of units that don't correspond to a built-in kind, an empty string is returned.

**Examples:**

* `kind(qty("m"))` → `"length"`
* `kind(qty("N*s"))` → `"momentum"`
* `kind(qty("W/s"))` → `""`

### `unitless(quantity)`

Is the given quantity unitless?
Equivalent to `kind(quantity)="unitless"`

**Example:**

* `unitless(qty("m/ft"))` → `true`
* `unitless(qty("m/s"))` → `false`

### `isbase(quantity)`

Is the given quantity in base units, as defined in the International System of Units (SI)?

**Examples:**

* `isbase(qty("kg"))` → `true`
* `isbase(qty("lb"))` → `false`

### `tobase(quantity)`

Convert the given quantity to base SI units.

**Example**:

* `tobase(qty(1,"inch"))` → `quantity(0.0254, "m")`

### `as(units,q_from)` or `as(q_to,q_from)`

Convert quantity `q_from` to the units specified by the given string, or the same units as `q_to`.
If the desired units are not compatible with `q_from`, an error is thrown.

**Examples**:

* `as("cm",qty(1.5,"m"))` → `quantity(150,"cm")`
* `as(qty("kg"),qty(100,"g"))` → `quantity(0.1,"kg")`

### `as_si(quantity)`

Convert quantity to SI units.

**Example:**

* `as_si(quantity(1,"lb*ft"))` → `quantity(0.1382549544, "m*kg")`

### `inverse(quantity)`

Return the reciprocal of the given quantity.

**Example:**

* `inverse(qty(2,"m/s"))` → `quantity(0.5,"s/m")`

### `same(a,b)`

Are `a` and `b` both exactly the same quantity, measured in the same units?

**Examples:**

* `same(qty(1,"m"),qty(100,"cm"))` → `false`
* `same(qty(1,"m"),qty(2,"m"))` → `false`
* `same(qty(1,"m"),qty(1,"m"))` → `true`

### `a < b`, `a <= b`, `a > b`, `a >= b`

Compare quantities `a` and `b`. If their units are not compatible, an error is thrown.

**Examples:**

* `qty(1,"cm") < qty(1,"m")` → `true`
* `qty(4,"feet") > qty(1,"m")` → `true`
* `qty(4,"feet") > qty(1,"m")` → `true`
* `qty(1,"cm") > qty(1,"cm")` → `false`
* `qty(1,"cm") >= qty(1,"cm")` → `true`

### `a = b`

Quantities `a` and `b` are equal if their units are compatible and they represent exactly the same amount.

**Examples:**

* `qty(2.54,"cm") = qty(1,"inch")` → `true`
* `qty(1,"cm") = qty(1,"second")` → `false`

### `a + b`, `a - b`, `a * b`, `a / b`

Arithmetic on units is supported. When adding or subtracting units, the result is given in the same units as the left-hand argument, and if the two quantities being combined are in incompatible units an error is thrown.
When multiplying or dividing, units are not automatically converted to their common names. For example, The division of a quantity in `Newtons` by a quantity in `m^2` returns a quantity in `N/m^2`, not in `Pa`.

**Examples:**

* `qty(1,"cm") + qty(1,"m")` → `quantity(101, "cm")`
* `qty(100,"cm") - qty(1,"m")` → `quantity(0, "cm")`
* `qty(100,"cm") * qty(1,"s")` → `quantity(100, "cm*s")`
* `qty(100,"N") / qty(4,"m^2")` → `quantity(25, "N/m^2")`

### `round(quantity,precision)`

Round `quantity` to the given precision, either a string in the form `"amount units"`, or another quantity.
If precision is not given, the quantity is rounded to the nearest whole unit.

**Examples:**

* `round(qty("123","cm"),"1 m")` → `quantity(100,"cm")`
* `round(qty(0.1697,"m"),qty(5,"cm"))` → `quantity(0.15,"m")`
* `round(qty(6.32,"kg"))` → `quantity(6,"kg")`

### `abs(quantity)`

The scalar amount of the quantity, as a number - in other words, strip off the units information.

**Example:**

* `abs(qty(53,"s"))` → `53`

### `string(quantity,[notation style])`

A string representing the given quantity, in the given [notational style](https://docs.numbas.org.uk/en/latest/number-notation.html#styles-of-notation) (plain English is the default)

**Examples:**

* `string(qty(23,"kg*s^-1"))` → `"23 kg/s"`
* `string(qty(1000.235,"kg"), "si-fr")` → `"1 000,235 kg"`

### `units_numerator(quantity)` and `units_denominator(quantity)`

The units of a quantity can be written as a fraction whose numerator and denominator both consist of a list of units.
When a squared or cubed unit is used, the base unit is repeated.
These functions return the lists of units in the numerator and denominator, respectively, for a given quantity.
Any order-of-magnitude prefixes are included as separate items in the list

**Examples:**

* `units_numerator(qty(1,"m"))` → `[ "meter" ]`
* `units_denominator(qty(1,"m"))` → `[ "1" ]`
* `units_numerator(qty(1,"kg*m^2/s^2"))` → `[ "kilogram", "centi", "meter", "centi", "meter" ]`
* `units_denominator(qty(1,"m^2/s^2"))` → `[ "second", "second" ]`

### `units(quantity)`

Return a quantity representing one unit of the same kind as the given quantity.

**Example:**

* `units(qty(23,"m^2/s"))` → `quantity(1, "m^2/s")`

### `units_string(quantity)`

Return a string describing the units of the given quantity, suitable for display. Powers are displayed in superscript, and **⋅** is used for multiplication.

**Example:**

* `units_string(qty("kg*cm^2/s^2"))` → `kg⋅cm²/s²`



## <a name="recognised-units">Recognised units</a>


