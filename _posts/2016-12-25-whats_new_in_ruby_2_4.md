---
layout: post
title:  "Whats new in Ruby 2.4?"
date:   2016-12-25 08:59:26 +0530
categories: release, ruby
---

It’s Christmas day, and following the tradition of the last few years, the Ruby
core team have released a new Ruby version today. I’ll summarize some of the
interesting new features in Ruby 2.4 here.


## Numbers

`Fixnum` and `Bignum `have been unified into `Integer` class.

So far, we’ve had two classes for storing integers - `Fixnum` for small integers, and `Bignum` for numbers outside this range. However, these are implementation details that programmers don’t need to worry about while writing code.

These two classes have been replaced by a single `Integer` class. Previously, `Integer` was a superclass of these two classes, but now both `Fixnum` and `Bignum `are aliases to `Integer`.

{% highlight ruby %}
# 2.3
42.class      #=> Fixnum
(2**62).class #=> Bignum

# 2.4
42.class      #=> Integer
(2**62).class #=> Integer

Fixnum == Integer #=> true
Bignum == Integer #=> true
{% endhighlight %}

### New Integer#digits method

{% highlight ruby %}
42.digits  #=> [2, 4]
Precision for float modifiers
{% endhighlight %}

Float methods like `#ceil`, `#floor`, `#truncate` and `#round` take an optional argument to set precision.

{% highlight ruby %}
1.567.round       #=> 2
1.567.round(2)    #=> 1.57
123.456.round(-1) #=> 120
{% endhighlight %}

### `Float#round` default behavior remains unchanged

This one isn’t really a change, but this change in default behavior initially made it to one of the preview releases, and was reverted later on.

By default, `#round` uses round-half-up behavior, ie. 1.5 would be rounded to 2. The new behavior was to use banker’s rounding, which rounds half to nearest even number. This might cause bugs in many existing applications which rely on half-up rounding, so the original default has been retained.


{% highlight ruby %}
# suggested behavior
1.5.round  #=> 2
2.5.round  #=> 2

# actual behavior
1.5.round #=> 2
2.5.round #=> 3
{% endhighlight %}

### Float#round options

Even though the round-to-nearest-even change was reverted, new options were introduced in `Float#round` that allow you to explicitly set what kind of rounding to use.

{% highlight ruby %}
2.5.round               #=> 3
2.5.round(half: :even)  #=> 2
2.5.round(half: :down)  #=> 2
2.5.round(half: :up)    #=> 3
{% endhighlight %}

## binding.irb

I’m a big fan of the pry gem for the `binding.pry` method that opens a REPL while running your code. IRB has now introduced this feature, and ruby now opens a REPL when it encounters the `binding.irb` method.

## Hash

### Hash#compact

This method, and the bang version, `#compact!`, remove keys with nil values from the hash.

{% highlight ruby %}
{ a: "foo", b: false, c: nil }.compact
#=> { a: "foo", b: false }
{% endhighlight %}

### Hash#transform_values

Applies the block for each value in the hash. Also provides a `#transform_values!` method that modifies the existing hash. Examples from the docs:

{% highlight ruby %}
h = { a: 1, b: 2, c: 3 }
h.transform_values {|v| v * v + 1 }  #=> { a: 2, b: 5, c: 10 }
h.transform_values(&:to_s)           #=> { a: "1", b: "2", c: "3" }
{% endhighlight %}

## Strings, Symbols and IO

### String supports unicode case mappings

Until now, Ruby only performed case conversion on ASCII characters. The `upcase`, `downcase`, `swapcase`, `capitalize` methods on `String` and `Symbol` have now been extended to work with unicode characters.

{% highlight ruby %}
# 2.3
"Türkiye".upcase   #=> "TüRKIYE"
"TÜRKİYE".downcase #=> "tÜrkİye"

# 2.4
"Türkiye".upcase   #=> "TÜRKIYE"
"TÜRKİYE".downcase #=> "türki̇ye"
{% endhighlight %}

### Specify string buffer size

`String.new` now allows a capacity argument to specify the size of the buffer. This will have performance benefits when the string will be concatenated many times.

{% highlight ruby %}
String.new('foo', capacity: 1_000)
Symbol#match now works like String#match
{% endhighlight %}

`Symbol#match` used to return the match position, while `String#match` returned a `MatchData` object. This has been fixed in 2.4 and now both return a `MatchData`.

{% highlight ruby %}
# 2.3
:hello_ruby.match(/ruby/) #=> 6

# 2.4
:hello_ruby.match(/ruby/) #=> #<MatchData "ruby">
IO#gets and other methods get a chomp flag
{% endhighlight %}

You can now add an optional `chomp: true` flag to `#gets`, `#readline`, `#each_line`, `#readlines` and `IO.foreach`.

{% highlight ruby %}
# In 2.3, you did this
foo = gets.chomp

# 2.4
foo = gets(chomp: true)
{% endhighlight %}

## Regexp

### Regexp#match?

This new method returns true or false without updating the `$~` global variable. Since it doesn’t create a `MatchData` object or update `$~`, it performs better than `#match`.

{% highlight ruby %}
/foo/.match?('foo')  #=> true
$~                   #=> nil
{% endhighlight %}

### Regexp#named_captures

Returns a hash representing information about the named captures.

{% highlight ruby %}
/(?<fname>.+) (?<lname>.+)/.match('Ned Stark').named_captures
#=> {"fname"=>"Ned", "lname"=>"Stark"}
{% endhighlight %}

## Enumerable

### Enumerable#sum

{% highlight ruby %}http://nithinbekal.com/posts/ruby-2-4-features/
(1..5).sum         #=> 15
%w(a b c).sum('')  #=> "abc"
{% endhighlight %}

## Files and directories

The `#empty?` method was added to `Dir`, `File` and `Pathname`.

{% highlight ruby %}
Dir.empty?('path/to/some/dir')     #=> true
File.empty?('path/to/some/file')   #=> true
Pathname.new('file-or-dir').empty? #=> true
{% endhighlight %}

## Language features
### Multiple assignment in conditionals

In Ruby 2.3, you would get a syntax error if you tried multiple assignment in a conditional. This has been changed to a warning instead.

{% highlight ruby %}
# 2.3
if (a,b = [1,2]) then 'yes' else 'no' end
#=> SyntaxError: (irb):9: multiple assignment in conditional

# 2.4

if (a,b = [1,2]) then 'yes' else 'no' end
#=> warning: found = in conditional, should be ==
#=> 'yes'

if (a,b = nil) then 'yes' else 'no' end
#=> warning: found = in conditional, should be ==
#=> 'no'
{% endhighlight %}

## Wrapping up

Not all the new features in Ruby 2.4 are mentioned here, but if you’re interested in the complete list of changes, take a look at the [Ruby 2.4.0 NEWS file](https://github.com/ruby/ruby/blob/v2_4_0/NEWS).

**Source:** http://nithinbekal.com/posts/ruby-2-4-features/
