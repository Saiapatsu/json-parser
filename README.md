# jsons

## Usage
Get an iterator with `jsons.parser(str)`,  
where str is a string containing one json value.

Get a pretty json string with `jsons.pretty(iter)`  
or a minified json string with `jsons.minify(iter)`,

The iterator will return 0, 2 or 4 values:
* type: one of: nil, "object", "array", "string", "number", "boolean", "nil";
* where: current position in str;
* if type is not "object" or "array":
* * value: value of the above Lua type;
* * valuestr: string, the value copied verbatim from str.

An "object" or "array" type opens one, and a nil closes either.  
Use a for loop to iterate over their contents.

Example input:
```json
[123, 456, ["foo", {"qux": "baz", "test": 789}, "bar"], null, false]
```

Example output (empty line represents no return value at all):
```lua
"array"  , 1
"number" , 2 , 123   , "123"
"number" , 7 , 456   , "456"
"array"  , 12
"string" , 13, "foo" , '"foo"'
"object" , 20
"string" , 21, "qux" , '"qux"'
"string" , 28, "baz" , '"baz"'
"string" , 35, "test", '"test"'
"number" , 43, 789   , "789"

"string" , 49, "bar" , '"bar"'

"nil"    , 57, nil   , "null"
"boolean", 63, false , "false"

68
cannot resume a dead coroutine
```

Example function that validates and returns an array of numbers:
```lua
local function arr(str)
	local parser = require("jsons").parser(str)
	assert(parser() == "array", "Not an array")
	local out = {}
	for type, value, value in parser do
		assert(type == "number", "Array contains a non-number")
		table.insert(out, value)
	end
	return out
end

for k,v in ipairs(arr("[1, 2, 3, 4, 5]")) do print(k, v) end
for k,v in ipairs(arr("[1, 2, [], 4, 5]")) do print(k, v) end
```

## Afterword
The iterator throws syntax errors when it encounters them.  
Thus, avoid causing side effects until the entire document has been parsed.

If you don't like any of the values that the iterator returns, then
just make a function that calls the iterator and returns only the ones you want:
```lua
local parser = jsons.parser(str)
local function parser2()
	local type, value, value = parser()
	return type, value
end
```
This will return only the type and the lua value, but not the location or
the verbatim value.

This parser was created primarily with pretty-printing/beautification
and linearization/minification in mind, hence the "verbatim" return value.  
In addition, Lua's nil is not JSON's null.

Use for anything else but the above at your own risk, because it has
never been used for anything else.

This might be better called a JSON tokenizer, although unlike another similar
streaming parser I just looked at, it does not emit commas, colons, separate
end tokens for arrays and objects etc.  
OTOH, the parser alone is a solid validator.

Speaking of validation, it does not care whether the contents of
strings are valid Unicode.  

(not to be confused with jsons for Python)
