ruby-numeric-with-unit
======================
This library provide NumerichWithUnit class to calculate numeric easily with unit of measurement.

Install
======================

    $ gem install numeric_with_unit

Usage
======================

Examples
----------------------

    require 'numeric_with_unit'

    #example 1
    length = NumericWithUnit.new(10, 'm') # An object representing 10[m]
    puts length                           #=> 10 m
    puts length['cm']                     #=> 1000/1 cm
                                          #   'centi' is processed by Rational class in internal, the result also become Rational class.

    time = 10.to_nwu('s')                 # Another way, use Fixnum#to_nwu. An object representing 10[s]
    puts time                             #=> 10 s
    puts time['min']                      #=> 1/6 min

    speed = length / time
    puts speed                            #=> 6000/1 cm/min
    puts speed['km/hr']                   #=> 18/5 km/hr

    require 'numeric_with_unit/util'      # load utility script to describe with natural notation.
    puts (10['m'] / 10['s'] )['km/hr']    #=> 18/5 km/hr

    # example 2
    puts (50['L/min'] + 3['m3/hr'] ) * 30['min'] #=> 3000/1 L

* Caution: `Numeric#[]`, `Fixnum#[]` and `Bignum#[]` are overrided when `require 'numeric_with_unit/util'`



How to define units
----------------------
Unit of measurement is expressed in NumericWithUnit::Unit class.
The basic unit (SI) is defined in 'numeric_with_unit/unit_definition/base.rb', and units derived from SI unit can be defined as follows.

    # [kg], [m], [s] are predefined here.
    NumericWithUnit::Unit['N'] = 'kg.m/s2'    # definition of newton
    NumericWithUnit::Unit['Pa'] = 'N/m2'      # definition of pascal
    NumericWithUnit::Unit['ton'] = 1000, 'kg' # definition of ton

If some unit is defined as a basic unit, it is not necessary to define units with prefixes (such as [cm]), units with exponents (such as [m2]) and assembly units (such as [m/s]).

In the above example, it is not necessary to define [kPa] or [Pa.s] (unit of viscosity) because [Pa] is defined.

The basic and common unit is predefined in 'numericwithunit/unit_definition/base.rb' and 'numericwithunit/unit_definition/common.rb'
(These files are loaded by default)

`NumericWithUnit::Unit.list.map(&:symbol)` gives a list of defined units.



Format of string representing unit
---------------------
For Unit#[] and NumericWithUnit#[], you can pass a string representing the unit.
For example.

- m
- cm
- m2
- kW.hr
- kg/cm2
- m.s-1
- J/kg/K
- kcal/(hr.m2.℃)

The basic unit with the prefix at the beginning and the exponent at the end is expressed in a form concatenated with "." or "/".
You can also enclose it as a basic unit with "()".




class NumericWithUnit::Unit
======================
The class representing "Unit of Measurement".

Unit.new
----------------------

    km = NumericWithUnit::Unit.new do |conf|
      conf.symbol = 'km'
      conf.dimension[:L] = 1
      conf.from_si{|x| x/1000}
      conf.to_si{|x| x*1000}
    end

`conf.from_si` and `conf.to_si` configure the conversion formula expressing SI basic unit.

Unit#cast
----------------------
1 mi = 1.609344 km  
Unit of relationship like above is generated by

    mi = km.cast('mi', 1.609344)

It is only a proportional relationship.
Use `Unit.new`, to convert unit of relationship like ℃ and ℉.

Unit<<, Unit[], Unit[]=
----------------------
Register `unit` as a basic unit by `Unit << unit`.
By registering as a basic unit, an assembly unit is automatically derived with `Unit[]`.

    NumericWithUnit::Unit << km
    NumericWithUnit::Unit << NumericWithUnit::Unit.new do |conf|
      conf.symbol = 'hr'
      conf.dimension[:T] = 1
      conf.from_si{|x| x/60/60}
      conf.to_si{|x| x*60*60}
    end
     
    p NumericWithUnit::Unit['km2']    #=> #<NumericWithUnit::Unit:[km2] {:L=>2}>
    p NumericWithUnit::Unit['km/hr']  #=> #<NumericWithUnit::Unit:[km/hr] {:L=>1, :T=>-1}>

Units registered as basic units are obtained with `Unit.list`.

Also, `Unit[]=` convert units and register them as basic units at the same time.

    NumericWithUnit::Unit['kph'] = 'km/hr'
    NumericWithUnit::Unit['ua'] = 1.495978706916e8, 'm'

----
The basic unit is defined in the following file.
Please `require` accordingly.

* 'numeric_with_unit/unit_definition/base' (SI unit, SI assembly unit and SI combination unit. It is `require` by default.)
* 'numeric_with_unit/unit_definition/common'  (Units seemed as common. It is `require` by default.)
* 'numeric_with_unit/unit_definition/cgs' (incomplete)
* 'numeric_with_unit/unit_definition/imperial' (incomplete)
* 'numeric_with_unit/unit_definition/natural' (incomplete)


class NumericWithUnit
======================
The class representing numeric with unit of measurement information.

It is able to unit convert, add, subtract, multiply, divide and raise a power.


NumericWithUnit.new(value, unit)
----------------------
A NumerichWithUnit object that have numeric of `value` and unit of measurement of `unit` is returned.
For `unit`, pass a string representing a unit or an object of Unit class.


Numeric#to_nwu(unit), Fixnum#to_nwu(unit), Bignum#to_nwu(unit)
----------------------
NumericWithUnit.new(self, unit) is returned.


NumericWithUnit#value
----------------------
Numeric returned.


NumericWithUnit#unit
----------------------
Unit object returned.


NumericWithUnit#convert(new_unit), NumericWithUnit#to_nwu(new_unit) 
----------------------
NumericWithUnit object converted to `new_unit` is returned.

NumericWithUnit::DimensionError will be raised if new_unit with different dimensions from self is specified.


NumericWithUnit#convert!(new_unit), NumericWithUnit#\[\](new_unit)
----------------------
self converted to `new_unit` and self is returned.

NumericWithUnit::DimensionError will be raised if new_unit with different dimensions from self is specified.


NumericWithUnit#+(other), NumericWithUnit#-(other)
----------------------
NumerichWithUnit object that have added or subtracted numeric and converted unit returned.
If `other` is not a NumericWithUnit class, `other` converted to dimensionless NumerichWithUnit object.

NumericWithUnit::DimensionError will be raised if other with different dimensions from self is specified.


NumericWithUnit#*(ohter), NumerichWithUnit#/(other)
----------------------
NumerichWithUnit object that have multiplied or divided numeric and derived unit returned.
If `other` is not a NumericWithUnit class, `other` converted to dimensionless NumerichWithUnit object.


NumericWithUnit#**(num)
----------------------
NumerichWithUnit object that have numeric raised to `num` and derived unit raised to `num` returned.



Utility script
=====================
For user who hope reduce number of key press.

    require 'numeric_with_unit/util2'
    
    x = 1000.0.m    # 1000.0[m]
    t = 1800.0.s    # 1800.0[s]
    v = (x/t).km_hr # 2.0 [km/hr]
    
    vis = 1.0.Pa.s  # 1.0 [s.Pa]
    puts vis.cP     #=> 1000.0 cP

