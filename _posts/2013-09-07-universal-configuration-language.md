---
layout: post
title:  "New configuration format for rspamd"
categories: rspamd update
---
After long live with XML format I've finally decided to improve the configuration
system to avoid various issues related to the configuration extending and readability.
In this post I try to describe the main features and principles of the configuration
language which I've called `RCL` - rspamd configuration language.

## Basic structure

RCL is heavily infused by `nginx` configuration as the example of a convenient configuration
system. However, RCL is fully compatible with `JSON` format and is able to parse json files.
For example, you can write the same configuration in the following ways:

* in nginx like:

~~~nginx
param = value;
section {
	param = value;
	param1 = value1;
	flag = true;
	number = 10k;
	time = 0.2s;
	string = "something";
	subsection {
		host = {
			host = "hostname"; 
			port = 900;
		}	
		host = {
			host = "hostname";
			port = 901;
		}	
	}
}
~~~

* or in JSON:

~~~json
{
	"param": "value",
	"param1": "value1",
	"flag": true,
	"subsection": {
		"host": [
			{	
				"host": "hostname",
				"port": 900
			},
			{
				"host": "hostname",
				"port": 901
			}
		]
	}
}
~~~

## Improvements to the json notation.

There are various things that makes json parsing more convenient for editing:

* Braces are not necessary to enclose the top object: it is automatically treated as object:

~~~json
"key": "value"
~~~

is the equivalent to:

~~~json
{"key": "value"}
~~~

* There is no requirement of quotes for strings and keys, moreover, `:` sign may be replaced with `=` sign or even skipped for objects:

~~~nginx
key = value;
section {
	key = value;
}
~~~

is the equivalent to:

~~~json
{
	"key": "value",
	"section": {
		"key": "value"
	}
}
~~~

* No commas mess: you can safely place a comma or semicolon for the last element in array or object:

~~~json
{
	"key1": "value",
	"key2": "value",
}
~~~

* Non-unique keys in an object are allowed and automatically converted to the arrays internally:

~~~json
{
	"key": "value1",
	"key": "value2"
}
~~~

is converted to:

~~~json
{
		"key": ["value1", "value2"]
}
~~~

* Numbers can have suffixes to specify standard multipliers:
	* `[kKmMgG]` - standard 10 base multipliers (so `1k` is translated to 1000)
	* `[kKmMgG]b` - 2 power multipliers (so `1kb` is translated to 1024)
	* `[s|min|d|w|y]` - time multipliers, all time values are translated to float number of seconds, for example `10min` is translated to 3600.0 and `10ms` is translated to 0.01
* Booleans can be specified as `true` or `yes` or `on` and `false` or `no` or `off`.
* It is still possible to treat numbers and booleans as strings by enclosing them in double quotes.

## General improvements

RCL supports different style of comments:

* single line: `#` or `//`
* multiline: `/* ... */`

Multiline comments may be nested:

~~~c
# Sample single line comment
/* 
 some comment
 /* nested comment */
 end of comment
*/
~~~

RCL supports external macroes both multiline and single line ones:

~~~nginx
.macro "sometext";
.macro {
	Some long text
	....
};
~~~

There are two internal macroes provided by RCL:

* `include` - read a file `/path/to/file` or an url `http://example.com/file` and include it to the current place of
RCL configuration;
* `includes` - read a file or an url like the previous macro, but fetch and check the signature file (which is obtained
by `.sig` suffix appending).

Public key (or keys) used for the last command are specified by the concrete RCL user (by rspamd for example).

## Emitter

Each RCL object can be serialized to one of the three supported formats:

* `JSON` - canonic json notation (with spaces indented structure);
* `Compacted JSON` - compact json notation (without spaces or newlines);
* `Configuration` - nginx like notation.

## Conclusion

RCL has clear design that should be very convenient for reading and writing. At the same time it is compatible with
JSON language and therefore can be used as a simple JSON parser. Macroes logic provides an ability to extend configuration
language (for example by including some lua code) and comments allows to disable or enable the parts of a configuration
quickly. Rspamd 0.6.0 will be the first version with RCL configuration. It will be possible to convert the existing XML configuration
to RCL one by rspamd itself.
