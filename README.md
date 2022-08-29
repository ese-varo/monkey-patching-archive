# Monkey-patching: Messing with your Ruby methods

## Introduction
In software development, as in any other profession, knowing your tools is a must if you really want to be proficient and take the most out of them. In ruby, as in some other programming languages, you can take advantage of monkey patching and use it either to add extra functionality to existing code or to create a custom implementation as a workaround that fits with the problem you are trying to solve.

## What is Monkey patch?
One of the etymological definitions of [monkey patch](https://en.wikipedia.org/wiki/Monkey_patch) is referred to as “monkeying about” with the code, or in other words “messing with it”.

Monkey patch is a dynamic feature that basically refers to the ability to have Classes open all the time, which enables you to modify or extend them at runtime either by adding new methods, overriding them, or even deleting them.

We can implement it in some dynamic programming languages. Python is the language that first adopted the term "monkey patch", they used to use it with a negative connotation as a way to derogate this technique, since it wasn’t accepted that well at the moment, considering that it wasn’t a good practice.
On the other hand, the Ruby community embraced the term. In some cases, it is also replaced by "duck punching" or "functional reloading", but being "monkey patching" the most used. Ruby adopted it by giving it a completely different meaning, without understating how dangerous it could be, but now putting a high emphasis on how useful and powerful tool it could be to help developers to create great solutions without compromising the integrity and maintainability of their code. The best example of this is the most used and known ruby framework for web development, Ruby on Rails.

As with some other controversial topics, there is a discussion out there about this technique and whether or not it is recommended to implement it, and as you may know, opinions are really divided. I’m not going to take a position about it, besides whether it is correct or not making use of it, what I would say is that it really depends on the problem you are trying to solve and its context, so taking the time to analyze it is more important.

As dangerous as it may sound it’s also a great tool if used properly, it is so, that you could end up having very elegant and readable solutions when using monkey patches, one of the most popular examples of it are the Rails time methods, e.g. `3.days.ago` and so.

The most common use case for it is to augment or decorate existing methods. If you find out that monkey patching a class would help you to solve a very particular problem on your project either because doing it might prevent you from completely change your current implementation, or just because the Class in question is a third-party or Core class, and after some intents like even trying to use other gems or other workarounds you still don’t have what you need, you can make use of it and hopefully end up with a great solution. Of course, there are some considerations and good practices that you can apply. If you want to know a little more about it I suggest reading [this post](https://www.justinweiss.com/articles/3-ways-to-monkey-patch-without-making-a-mess/) by Justin Weiss, that would be really helpful on which techniques you can follow in order to monkey patching in a secure way.

## Examples
In order to understand this technique better, the following are some examples about how we can make use of it, in different situations depending on our needs.

### Adding a new method
Now, let’st try to monkey patch the `String` Class by adding a new method. Let’s say we need to find out if a string has any number in it, so we’re going to create a new method named `has_numbers?` to help us solve this problem. The code needed is the following:

```ruby
class String
  def has_numbers?
    !!(self =~ /\d/)
  end
end

puts "does 'abc' have numbers? #{'abc'.has_numbers?}"
puts "does 'abc0123' have numbers? #{'abc0123'.has_numbers?}"
```

After running this code the output should look like this:
```
=> does 'abc' have numbers? false
=> does 'abc0123' have numbers? true
```

### Overriding a method
This is one of the most common use cases for a monkey patch, overriding the current implementation of a method. For this example lets say that we are required to log and return the custom message `this code is under maintenance` each time the `length` method of the `String` core Class is called, instead of actually returning the length of the string.
The code would be like following:

```ruby
class String
  def length
    message = ‘this code is under maintenance’
    puts message
    message
  end
end

‘123’.length
```

After running the above, we should have the following output:
```
=> this code is under maintenance
```

### Extending an existing method using Alias
Some times we need to add some extra functionality to an existing method but we just need it in some places so keeping the original implementation would be required. In those cases we could extend an existing method by using the alias technique. For this example, lets say we need to add 2 to the result of calling the `size` method of the `String` core Class.
The code would be like following:

```ruby
class String
  def new_size
    self.old_size + 2
  end
  alias :old_size :size
  alias :size :new_size
end

puts "size of '1234'.size is #{'1234'.size}"
```

After running the above, we should have the following output:

```
=> size of '1234'.size is 6
```

You may be wondering that if instead of having two aliases we could just have something like the following:

```ruby
class String
  def new_size
    self.size + 2
  end
  alias :size :new_size
end

puts "size of '1234'.size is #{'1234'.size}"
```

After running the above, we’ll have an error similar to this:

```
=> in `new_size': stack level too deep (SystemStackError)
```

The reason for it is that the alias command is executed when the monkey patch gets loaded, then when we call the `size` method for the first time, it will execute the `new_size` method because of the alias, the inner call to ‘size’ inside the ‘new_method’ would cause the method to call itself falling into an infinite loop.
That’s why we need two aliases, as in the example above, one for redirecting the original name to the new method and another for keeping access to the original implementation.

### Deleting a method
Extending and overriding methods seems to be the most common way to go when doing a monkey patch, but if for any reason you end up trying to remove a method, there are some ways to do it. Lets try to remove the ‘odd?’ method from the core `Integer` Class using the following code:

```ruby
puts 3.odd?
Integer.class_eval("undef :odd?")
puts 3.odd?
```

After running this code the output should be like this:

```
=> true
=> undefined method `odd?' for 3:Integer (NoMethodError)
```

## Considerations / Pros and cons

One of the core principles of Ruby is being developer friendly, so there’s no coincidence that Rubyists are really into making their code as much readable as possible. With monkey patches, you have the chance to create very elegant interfaces. It doesn't matter if it is just for the sake of improving its readability. If you think it is worth it, why don’t give it a chance?

When monkey patching a core Class or a third party Class, especially when extending the current implementation of a method, would require you to keep an eye on them each time you update your gems or ruby version in order to be sure that your code continues working as expected since that could potentially be a source for bugs or even a cause for breaking changes if the original implementation has changed. A good test suit and documentation would help you to deal with it.

Monkey patching a core class just for having a custom method adapted to a particular need in your project is still a valid reason for doing it.

If you are thinking of overriding an existing method through monkey patching, consider making use of the alias mechanism in ruby, so you can still have the original implementation where it’s still needed.

Having documentation is always a good practice as it helps with the maintainability of your projects, and it’s even more important when you implement this kind of technique since your might be dealing with changes in core classes so having clear and well-documented those changes is a must.

As mentioned above, the wrong implementation of a monkey patch might be the root of wired bugs even if you think you’ve applied it in the right way. Future bugs are always a possibility, especially in this kind of solutions, so in order to handle them in a proper way, having a good error handling implementation would help you a lot. Including meaningful error logs would help you to faster recognize and identify bugs.

If you find yourself trying to monkey patch a Class of your own, think twice about it, there are big chances that you don’t really need to go that way.

## Conclusion

As always, with great power comes great responsibility. Don’t hesitate on implementing this technique if you find out that this is the best approach you can take. The only takeaway that I would mention is “do it responsibly by following best practices”, and always take your time to analyze the problems you are trying to solve, the more understanding you have about them, the better solutions you may end up with. Remember that is always better to think twice and code it once than the other way around.

If you find yourself monkey patching a functionality, either to add some improvements that might be useful for others or to fix a bug (we all welcome bug fixes 😃), it would be great to share it with the community of the respective gem you are working with.
Don’t be selfish 😜

> “Don’t let anyone tell you that a powerful technique is too scary or dangerous for you. Let it pique your curiosity instead.”
>
> <cite>DHH, Creator of Ruby on Rails. 10:06 a.m. 19 feb. 2018. [Tweet](https://twitter.com/dhh/status/965618592606638080).</cite>

## Sources:

https://www.justinweiss.com/articles/3-ways-to-monkey-patch-without-making-a-mess/

https://www.brice-sanchez.com/how-to-properly-monkey-patch-a-ruby-class-in-ruby-on-rails/

https://en.wikipedia.org/wiki/Monkey_patch
