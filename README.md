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
digits. `<etc>` suggests that preceding text continues or may be repeatable in
some way.

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

### const

	@const <name> <type>
	@const <name> <type> = <value>

Defines a constant value. Optionally, the value of the constant may be
explicitly given as a Lua expression.

### var

	@var <name> <type>

Defines a variable with a name and a given type.

### type

	@type <name> <type>

Defines a new value type under a given name. This name may be used in place of
a literal type definition.

	-- as literal type
	@var One struct {foo bool, bar number}
	@var Two struct {foo bool, bar number}

	-- as named type
	@type Thing struct {foo bool, bar number}
	@var One Thing
	@var Two Thing

Semantically, this enables multiple values to have the same type.

A named type does not have to be defined before it is used.

### method

	@method <name> ( <arguments> ) <returns>
	@method <name> ( <arguments> ) ( <returns> )

Used after a @type or @var to indicate a class-like method. Alternatively, the
name may be dot-separated, which explicitly defines the type or variable it is
a member of. Only applicable to table-like types.

It would be possible to define a method-like struct field using types alone.

	@type Thing struct {
		~Method function(self Thing)
	}

However, this can also be interpreted simply as a readonly function. The
method tag is used to indicate that a field is, semantically, a method. It
also works on other table-like types, which may not have ways to define
method-like fields.

### event

	@event <name> ( <arguments> )

Used after a @type or @var to indicate a class-like event. Alternatively, the
name may be dot-separated, which explicitly defines the type or variable it is
a member of. Only applicable to table-like types.

"Event" is a generic term for a system that follows an event-like pattern.
That is, a "listener" function is passed to the event in some way, and the
event "fires" at some point, calling the function with the defined arguments.

### arg

	@arg <name>

Used after a @method, @event, or function type, to define a single argument.
The name corresponds to the name defined in the argument list.

### return

	@return <name>

Used after a @method or function type to define a function's return value. The
name corresponds to the name defined in the return list.

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

	function ( )                             No arguments, no returns
	function ( <name> <type>, <etc> )        Arguments, no returns
	function ( <name> <type>, ... )          Variable arguments, no returns
	function ( ) <name> <type>, <etc>        No arguments, returns
	function ( ) ( <name> <type>, <etc> )    No arguments, returns with parentheses
	function ( ) <name> <type>, ...          No arguments, variable returns

### struct

A Lua table that contains a specific number of named fields of specific types.
If a field name is prefixed with `~`, then the field is readonly. This
suggests that a field is to be set only by internal means.

	{<fieldName> <fieldType>, <etc>}                    Single line
	struct {<fieldName> <fieldType>, <etc>}             Single line with clarity
	struct {                                            Multi-line
		<fieldName> <fieldType>
		~<fieldName> <fieldType>                        Readonly field
		<fieldName> <fieldType>
		<etc>
	}

Structs may be extended by defining a struct on an existing struct type.

	@type Shape struct {
		Visible bool
		PosX    number
		PosY    number
	}

	@type Circle Shape { -- extends Shape
		Radius number
		-- has Visible, PosX, and PosY
	}

### map

A Lua table that contains any number of fields of a single key type and single
value type.

	[<keyType>]<valueType>
	~[<keyType>]<valueType>        All fields are readonly

### tuple

A Lua table that contains a specific number of unnamed fields of specific
types. If a field is prefixed with `~`, then the field is readonly.


	{<type>, <etc>}
	tuple {<type>, <etc>}
	tuple {
		<type>
		~<type>
		<type>
		<etc>
	}


Like structs, tuples may also be extended. However, tuples are ordered, so
extended fields are appended to existing ones.

	@type Shape tuple {bool, number, number}
	@type Circle Shape {number}

### array

A Lua table that contains any number of fields of a single type.

	[]<type>
	~[]<type>        All fields are readonly

### Other type syntax

A type may consist of multiple types, which can be indicated by a `|` symbol.

	bool | int             A bool or integer
	bool | int | string    A bool, int, or string

It is possible for ambiguity to occur.

	@var Thing function() name TypeA | TypeB

	-- would be interpreted as
	function() name (TypeA | TypeB)

	-- but could be
	(function() name TypeA) | TypeB

In cases like this, use @type to disambiguate.

	@type Func function() name TypeA
	@var Thing Func | TypeB

Prefixing any type with `?` indicates that the value is optional, or rather,
may be nil. This is pretty much a shortcut for `<type> | nil`.

	?bool    bool or nil
	?int     integer or nil
	?nil     what are you doing

## EBNF

The following EBNF syntax describes the grammar of luat tags.

	tag = "@" ,
	      ( tag_module
	      | tag_const
	      | tag_var
	      | tag_type
	      | tag_event
	      | tag_method
	      | tag_arg
	      | tag_return
	      ) , [";"] ;

	tag_module = "module" , s, {ident | "."} , s , type ;
	tag_const  = "const" , s , ident , s , type , [s , "=" , s , ?lua:exp?] ;
	tag_var    = "var" , s , ident , s , type ;
	tag_type   = "type" , s , value_pair ;
	tag_method = "method" , s , ident, {"." , ident} , s , func_args , s , func_returns ;
	tag_event  = "event" , s , ident, {"." , ident} , s , event_args ;
	tag_arg    = "arg" , [s , ident] ;
	tag_return = "return" [s , ident] ;

	type = ["?"] ,
	       ( ident
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
	       ) , [w , "|" , w , type] ;

	type_bool     = "bool" ;
	type_int      = "int" ;
	type_number   = "number" ;
	type_string   = "string" ;
	type_value    = "*" ;
	type_function = ["function"] , s , func_args , s , func_returns ;
	type_array    = ["~"] , s , "[]" , s , type ;
	type_map      = ["~"] , s , "[" , s , type , s , "]" , s , type ;

	type_tuple      = tuple_line | tuple_multiline ;
	tuple_line      = (["tuple"] | ident) , s , "{" , s , [tuple_field , {s , "," , s , tuple_field}] , s , "}" ;
	tuple_multiline = ("tuple" | ident) , s , "{" , w , {tuple_field , [s , ","] , w} , "}" ;
	tuple_field     = ["~"] , s , type ;

	type_struct      = struct_line | struct_multiline ;
	struct_line      = (["struct"] | ident) , s , "{" , s , [struct_field , {s , "," , s , struct_field}] , s , "}" ;
	struct_multiline = ("struct" | ident) , s , "{" , w , {struct_field , [s , ","] , w} , "}" ;
	struct_field     = ["~"] , s , value_pair ;

	event_args   = "(" , s , [value_pair , {s , "," , s , value_pair}] , s , ")" ;
	func_args    = "(" , s , [func_arg , {s , "," , s , func_arg}] , [s , "..."] , s , ")" ;
	func_arg     = value_pair , [s , "=" , s , ?lua:exp?] ;

	func_returns = "(" , s , returns , s , ")" | returns ;
	returns       = [value_pair , {s , "," , s , value_pair}] , [s , "..."] ;

	value_pair = ident , s , type ;
	ident      = (a | "_") , {a | d | "_"} ;

	a = ?regexp:[A-Za-z]? ;
	d = ?regexp:[0-9]? ;
	w = { s | "\n" | "\r\n" } ;
	s = { " " | "\t" } ;

Some notes:

- `?lua: ... ?` indicates a rule defined by the [Lua reference manual](http://www.lua.org/manual/5.1/manual.html#8).
- `?regexp: ... ?` indicates a simple regular expression.
