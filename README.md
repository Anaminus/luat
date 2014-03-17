# Luat

**Luat** is yet another Lua documentation system. Unlike others, luat is
focused on describing the types of values, to a point where they could be
theoretically enforceable (if such a parser were written).

For now though, it only serves to reduce the redundancies involved in
describing value types and how they should be formatted. There's no parser, or
documentation generator (yet). Any actual type enforcement will have to be
done by the careful programmer.

## About examples within this file

Examples of syntax within this file are fairly intuitive. Text enclosed in `<`
and `>` indicates a placeholder. The text between them suggests what it
placeholds. For example, `<name>` might suggest a value containing letters and
digits.

For a complete, unambiguous syntax, see the [EBNF](#ebnf) section.

## Luat Location

Like most other Lua documentation systems, luat syntax is usually contained
inside comments in a Lua source file. However, it is designed to be
independent of the actual source code. That is, you can put bits of luat
anywhere you want; dispersed throughout the file, smashed together at the top,
in a completely different file, etc.

## Tags

Tags are started with a `@` symbol, beginning on a new line. Text after a tag
is a description of the tag. Optionally, a `;` symbol may be used to indicate
the end of the tag, if necessary.

Some tags are only correct if they're defined after some other specific tag.
This relation occurs across an entire file. That is, it doesn't make a
difference if tags are defined in separate comment blocks within a Lua file;
they are still evaluated top to bottom.

The following tags are defined:

### module

	@module <name> <type>
	@module <name>

Defines a new module with the given name. The type indicates the type of value
the module returns.

All tags afterwards are assumed to be apart of the module. Until another
module tag is defined, at least.

The module tag may define the same name multiple times, which is useful for
defining parts of the module in multiple places. In this case, the type may be
omitted, but doesn't have to be. However, the type should be defined somewhere
at least once for a single module. Defining multiple different types for a
single module is invalid.

### type

	@type <name> <type>

Defines a new value type under a given name. This name may be used in place of
a literal type definition. A named type does not have to be defined before it
is used.

	-- as literal type
	@type Thing struct {
		foobar struct {foo bool, bar number}
	}

	-- as named type
	@type Foobar struct {foo bool, bar number}
	@type Thing struct {
		foobar Foobar
	}

Semantically, this enables values to have the same type.

### field

	@field <name>
	@field

Used after a struct or tuple @type to define a single field. The named form
must be used for struct, while the unnamed form is used for tuples.

### method

	@method <name> ( <arguments> ) <returns>
	@method <name> ( <arguments> ) ( <returns> )

Used after a @type to indicate a class-like method. Only applicable to table-
like types. The argument and return definitions are the same as that of
[function](#function) type definitions.

It would be possible to define a method-like struct field using types alone:

	@type Thing struct {
		~Method function(self Thing)
	}

However, this can also be interpreted simply as a readonly function. The
method tag can be used to indicate that a field is, semantically, a method. It
also works on other table-like types, which may not have ways to define
method-like fields.

### event

	@event <name> ( <arguments> )

Used after a @type to indicate a class-like event. Only applicable to table-
like types. The argument definitions are the same as that of
[function](#function) type definitions.

"Event" is a generic term for a system that follows an event-like pattern.
That is, a "listener" function is passed to the event in some way, and the
event "fires" at some point, calling the function with the defined arguments.

### arg

	@arg <name>

Used after a @method, @event, or function type, to define a single argument of
the function. The name corresponds to the name defined in the argument list.

### return

	@return <name>

Used after a @method or function type to define a return value of the
function. The name corresponds to the name defined in the return list.

### using

	@using <module>
	@using <name> <module>

Indicates that the types in a specified module may be referenced directly,
rather than through the module name.

An alternative name may also be given, which can be used in place of the
module name. This is necessary for modules with complex names.

Normally, a type in another module may be indicated in the following way:

	@module foo
	@type Thing bool

	@module bar
	@type GetThing function() foo::Thing

With @using, this may be simplified:

	@module bar
	@using foo
	@type GetThing function() Thing

If types from two modules share the same name, then the type from latest
module indicated with @using will be used. The two may still be distinguished
by using their module names.

## Types

Basic named types:

- `bool`:   a Lua boolean
- `number`: a Lua number
- `string`: a Lua string
- `nil`:    a Lua nil value
- `int`:    a signed integer

The following special types are defined:

### Any

Represented by a `*` symbol, indicating a value of any type.

	*

### function

A first-class function.

	function ( <arguments> ) <returns>
	function ( <arguments> ) ( <returns> )

`<arguments>` and `<returns>` are both comma-separated lists of zero or more
value pairs, which take the form of:

	<name> <type>

A default value for the argument may be given, which is indicated by `=`
followed by a Lua expression.

	<name> <type> = <exp>

This implies that the type is nullable. For complex expressions, it may be
easier to use some placeholder variable, then describe it in the argument's
definition.

The last item in either list may indicate a varying amount of values. This is
indicated by a `...` followed by a type:

	...<type>

This indicates the value types of the remaining arguments. The type may also
be omitted, which indicates any type.

### struct

A Lua table that contains a specific number of named fields of specific types.

	{<fields>}           Single line
	struct {<fields>}    Single line with clarity
	struct {             Multi-line
		<fields>
	}

`<fields>` is a comma-separated list of zero or more value pairs, which take
the form of:

	<name> <type>

This is distinct from tuple fields, which require only a type. If a field name
is prefixed with `~`, then the field is readonly. This suggests that a field
is to be set only by internal means.

If the field list is defined on multiple lines, then the commas should be
omitted. For structs defined on a single line, the `struct` is optional.

Structs may be extended by defining a struct on an existing struct type. This
is done by replacing `struct` with the type name:

	@type Shape struct {
		Visible bool
		Position {X number, Y number}
	}

	@type Circle Shape {
		Radius number
	}

### map

A Lua table that contains any number of fields of a single key type and single
value type. Prefixing with `~` indicates that all fields are readonly.

	[<keyType>]<valueType>
	~[<keyType>]<valueType>

### tuple

A Lua table that contains a specific number of unnamed fields of specific
types.

	{<fields>}
	tuple {<fields>}
	tuple {
		<fields>
	}

`<fields>` is a comma-separated list of zero or more types. This is distinct
from struct fields, which require a name and a type. If a field is prefixed
with `~`, then the field is readonly. This suggests that a field is to be set
only by internal means.

If the field list is defined on multiple lines, then the commas should be
omitted. For tuples defined on a single line, the `tuple` is optional.

Like structs, tuples may also be extended. However, tuples are ordered, so
extended fields are appended to existing ones.

	@type Shape tuple {bool, {number, number} }
	@type Circle Shape {number}

### array

A Lua table that contains any number of fields of a single type. Prefixing
with `~` indicates that all fields are readonly.

	[]<type>
	~[]<type>

### Multiple types

Indicates one type from a given list of specific types.

	[<types>]
	[
		<types>
	]

`<types>` is a comma-separated list of one or more types. If defined is
defined on multiple lines, then the commas should be omitted.

### Nullable types

Prefixing any type with `?` indicates that the value is optional, or rather,
may be nil. This is pretty much a shortcut for `[<type>,nil]`.

	?bool    bool or nil
	?int     integer or nil
	?nil     what are you doing

### External types

Types from other modules may be used as a type:

	<module>::<type>

See [using](#using) for referencing external types directly.

## EBNF

The following EBNF syntax describes the grammar of luat tags.

	tag = "@" ,
	      ( tag_module
	      | tag_using
	      | tag_type
	      | tag_field
	      | tag_event
	      | tag_method
	      | tag_arg
	      | tag_return
	      ) , [";"] ;

	tag_module = "module" , s, module_name , s , type ;
	tag_using  = "using" , [s , module_nick] , s , module_name ;
	tag_type   = "type" , s , value_pair ;
	tag_field  = "field" , [s , ident] ;
	tag_method = "method" , s , ident, s , func_args , s , func_returns ;
	tag_event  = "event" , s , ident, s , event_args ;
	tag_arg    = "arg" , [s , ident] ;
	tag_return = "return" , [s , ident] ;

	type = ["?"] ,
	       ( type_external
	       | type_name
	       | type_bool
	       | type_int
	       | type_number
	       | type_string
	       | type_value
	       | type_function
	       | type_struct
	       | type_map
	       | type_tuple
	       | type_array
	       | type_mult
	       ) ;

	type_external = (module_name | module_nick) , "::" , type_name ;
	type_bool     = "bool" ;
	type_int      = "int" ;
	type_number   = "number" ;
	type_string   = "string" ;
	type_value    = "*" ;
	type_function = ["function"] , s , func_args , s , func_returns ;
	type_array    = ["~"] , s , "[]" , s , type ;
	type_map      = ["~"] , s , "[" , s , type , s , "]" , s , type ;

	type_tuple      = tuple_line | tuple_multiline ;
	tuple_line      = (["tuple"] | type_name) , s , "{" , s , [tuple_field , {s , "," , s , tuple_field}] , s , "}" ;
	tuple_multiline = ("tuple" | type_name) , s , "{" , w , {tuple_field , w} , "}" ;
	tuple_field     = ["~"] , s , type ;

	type_struct      = struct_line | struct_multiline ;
	struct_line      = (["struct"] | type_name) , s , "{" , s , [struct_field , {s , "," , s , struct_field}] , s , "}" ;
	struct_multiline = ("struct" | type_name) , s , "{" , w , {struct_field , w} , "}" ;
	struct_field     = ["~"] , s , value_pair ;

	type_mult      = mult_line | mult_multiline ;
	mult_line      = "[" , s , [type , {s , "," , s , type}] , s , "]" ;
	mult_multiline = "[" , w , {type , w} , "]" ;

	event_args = "(" , s , [value_pair , {s , "," , s , value_pair}] , s , ")" ;
	func_args  = "(" , s , [func_arg , {s , "," , s , func_arg}] , [s , var_arg] , s , ")" ;
	func_arg   = value_pair , [s , "=" , s , ?lua:exp?] ;

	func_returns = "(" , s , returns , s , ")" | returns ;
	returns      = [value_pair , {s , "," , s , value_pair}] , [s , var_arg] ;

	var_arg = "..." , type;

	module_name = {ident | "."} ;
	module_nick = ident ;
	type_name   = ident ;
	value_pair  = ident , s , type ;
	ident       = (a | "_") , {a | d | "_"} ;

	a = ?regexp:[A-Za-z]? ;
	d = ?regexp:[0-9]? ;
	w = { s | "\n" | "\r\n" } ;
	s = { " " | "\t" } ;

Some notes:

- `?lua: ... ?` indicates a rule defined by the [Lua reference manual](http://www.lua.org/manual/5.1/manual.html#8).
- `?regexp: ... ?` indicates a simple regular expression.
