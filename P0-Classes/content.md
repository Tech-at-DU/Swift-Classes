---
title: "Learn Swift by example - Part 2: Enums"
slug: classes-in-swift
---     

In [part 1](https://www.makeschool.com/tutorials/learn-swift-by-example-part-1-structs/structs-in-swift) and [part 2](https://www.makeschool.com/tutorials/learn-swift-by-example-part-2-enums) of this tutorial series we have discussed how the value types *enum* and *struct* can be used in Swift. We have learned that these value types provide almost the same functionality as classes. 

But don't worry, classes still have their place in Swift. Unlike value types, classes support inheritance, muliple ownership and deinitializers. When writing Swift code for iOS we often need to subclass Apple's UIKit classes, so dealing with classes remains important.

Today we will discuss classes in detail. We will start with basic class definitions, then we'll look into subclassing and initialization. 

##Basics

Class definitions are way simpler than they used to be in Objective-C. We no longer have separate header and implementation files. Let's start by defining a simple `User` class in playground:

    class User {
      var name: String
      var age: Int
    }
    
With this class in place the Swift compiler will ommit an error message: *Class 'User' has no initializers*.

![](no_init_error.png)

This is one of the first differences between structs and classes. Structs provide a memberwise initializer by default. **Classes in Swift don't have default initializers (by default)**. Swift takes initialization security very seriously and doesn't allow non-optional properties to be uninitialized when an instance of our class is created. This means we will need to provide an initializer that sets all of our properties to non-nil values. 

One way of solving this problem is to make the default initializer available by setting default values for all of our properties:

    class User {
      var name: String = ""
      var age: Int = 0
    }
    
Now the compiler error disappears and Swift generates an initializer that takes no parameters. We can create a `User` instance like this:

    let user1 = User()

The new user will have an empty name and an age of *0*. In many cases this default initializer is not very helpful, instead we should provide an initializer that takes a name and an age:

    class User {
      var name: String = ""
      var age: Int = 0
      
      init (name: String, age: Int) {
        self.name = name
        self.age = age
      }
    }

If you are familiar if Objective-C you should be delighted by how simple initializers in Swift are. Since `User` is a root class (it is not subclassing from any other class) we don't need to call a `super` initializer. All we're doing is taking the two parameters and assigning them to our properties. Now that we have provided a custom initializer you will realize that our default initializer is no longer available.

You should see the following compiler error: *Missing argument for parameter 'name' in call*.
![](missing_parameter.png)

Swift only provides the default initializer when we don't implement a custom one. You are now required to initialize the user with a name and age:

    let user1 = User(name: "User", age: 99)
    
We have defined and initialized a basic class. Next, let's take a look at how classes deal with mutablity.
    
##Mutablity

As mentioned throughout the first two parts of this tutorial series, one of the important differences between value types and reference types is how they work in combination with `let` and `var`. 

When you use the `let` keyword to declare an immutable variable of a reference type (a class) then only the reference itself becomes immutable. This means that the variable will not be able to reference a different object, but it is possible to change values of the object that's currently referenced.

Let's take a look at this in practice:
    
    let user1 = User(name: "User", age: 99)
    user1.age = 20
    
You can create a user, assign the instance to a `let` variable and later change properties of that user. This wouldn't be possible if the user was declared as a `struct`. You cannot however, assign a new value to this variable:

    let user1 = User(name: "User", age: 99)
    user1.age = 20

    user1 = User(name: "User2", age: 10)
    
You will see this compiler error: *Cannot assign to 'let' value 'user1'*:
![](cannot_assign_let.png)

It's important to keep the different semantics for `let` in mind when dealing with classes and structs in Swift.

##Inheritance
Class inheritance in Swift works mostly as in Objective-C and other popular languages. There are two interesting aspects which I want to discuss in more detail: initializor inheritance and overriding.

Let's add a method to our `User` class so that we can demonstrate method overriding:

    class User {
      var name: String = ""
      var age: Int = 0
      
      init (name: String, age: Int) {
        self.name = name
        self.age = age
      }
      
      func sayHi() -> String {
        return "Hi \(name)"
      }
    }
    
And now let's create a subclass that adds one property and overrides the `sayHi` function:

    class SpecialUser : User {
      var greetingMessage: String
      
      func sayHi() -> String {
        return "\(greetingMessage) \(name)"
      }
    }

The syntax for inheritance is the same as for conforming to a protocol, add the type after a colon behind the class name. With this approach you will trigger two compiler errors: *Class 'SpecialUser' has no initializers* and *Overriding delcaration requires an 'override' keyword*. 

Let's tackle them one by one. The first error occurs because `greetingMessage` is not initialized by any initializer and doesn't have a default value. We can either add an initializer or assign a default value to `greetingMessage`. 

The second error occurs because Swift explicitly requires us to use the `override` keyword whenever we override an implementation of a superclass.

Here's how we can mute the compiler:

    class SpecialUser : User {
      var greetingMessage: String
      
      init(name: String, age: Int, greetingMessage: String) {
        self.greetingMessage = greetingMessage
        super.init(name: name, age: age)
      }
      
      override func sayHi() -> String {
        return "\(greetingMessage) \(name)"
      }
    }

We add an initializer that takes a `greetingMessage`. Additionally we add parameters for `name` and `age`. Swift requires us to call the initializer of the superclass, so we need to provide useful values for `name` and `age` as well.

One significant difference to Objective-C is that **Swift subclasses don't inherit their superclass initializers when a custom initializer is provided**.  This means the only way to instantiate a `SpecialUser` is calling the `init(name:, age:, greetingMessage:)` initializer. This is a great improvement compared to Objective-C. If all initializers were inherited we could instantiate a `SpecialUser` with `init(name:, age:)` which would leave the `greetingMessage` uninitialized.

The last aspect of initializer inheritance that we will discuss is the difference between `required`, `convenience` and designated initializers.

##Classes in Swift: Multiple References

##Classes in Swift: Deinitialization


##Conclusion

Enums are very powerful in Swift! We can associate values with enum members and use them to model functions with multiple return results. As shown in the bank account example, this can be very useful for error handling. 

We can also use enums to build state machines. Enums are guaranteed to only store one of their possible values at any point in time, this guarantee can be used to write more predictable code.

The possibilities don't end here, I'm very excited to see how enums will be used to write better Swift code.

If you want to learn more about Swift and ship your own original iPhone App or iPhone Game you should attend our [Summer Academy](https://makeschool.com/apply?referrer=54750)!

Stay tuned for more tutorials in this series!

##Related Resources

- A great talk by [Austin Zheng](https://twitter.com/austinzheng) about [enums and pattern matching in Swift](http://realm.io/news/swift-enums-pattern-matching-generics/)
