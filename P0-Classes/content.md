---
title: "Learn Swift by example - Part 3: Classes and Initialization"
slug: classes-in-swift
---     

In [part 1](https://www.makeschool.com/tutorials/learn-swift-by-example-part-1-structs/structs-in-swift) and [part 2](https://www.makeschool.com/tutorials/learn-swift-by-example-part-2-enums) of this tutorial series we have discussed how the value types *enum* and *struct* can be used in Swift. We have learned that these value types provide almost the same functionality as classes. 

But don't worry, classes still have their place in Swift. Unlike value types, classes support inheritance, multiple ownership and deinitializers. When writing Swift code for iOS we often need to subclass Apple's UIKit classes, so dealing with classes remains important.

Today we will discuss classes in detail. We will start with basic class definitions, then we'll look into subclassing and the many details of initialization in Swift.

##Basics

Class definitions are way simpler than they used to be in Objective-C. We no longer have separate header and implementation files. Let's start by defining a simple `User` class in playground:

    class User {
      var name: String
      var age: Int
    }
    
With this class in place the Swift compiler will omit an error message: *Class 'User' has no initializers*.

![](no_init_error.png)

This is one of the first differences between structs and classes. Structs provide a memberwise initializer by default. **Classes in Swift don't have default initializers (by default)**. Swift takes initialization security very seriously and doesn't allow non-optional properties to be uninitialized when an instance of our class is created. This means we will need to provide an initializer that sets both of our properties to non-nil values. 

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

If you are familiar with Objective-C you should be delighted by how simple initializers in Swift are. Since `User` is a root class (it is not subclassing from any other class) we don't need to call a `super` initializer. All we're doing is taking the two parameters and assigning them to our properties. 

Now that we have provided a custom initializer you will realize that our default initializer is no longer available.

You should see the following compiler error: *Missing argument for parameter 'name' in call*.
![](missing_parameter.png)

Swift only provides the default initializer when we don't implement a custom one. You are now required to initialize the user with a name and age:

    let user1 = User(name: "User", age: 99)
    
We have defined and initialized a basic class. Next, let's take a look at how classes deal with mutability.
    
##Mutability

As mentioned throughout the first two parts of this tutorial series, one of the important differences between value types and reference types is how they work in combination with `let` and `var`. 

When you use the `let` keyword to declare an immutable variable of a reference type (a class) then only the reference itself becomes immutable. This means that the variable will not be able to reference a different object, but it is possible to change values of the object that's currently referenced.

Let's take a look at this in practice:
    
    let user1 = User(name: "User", age: 99)
    user1.age = 20
    
You can create a user, assign the instance to a `let` variable and later change properties of that user. This wouldn't be possible if the user was declared as a `struct`. You cannot however assign a new object to this variable:

    let user1 = User(name: "User", age: 99)
    user1.age = 20

    user1 = User(name: "User2", age: 10)
    
You will see this compiler error: *Cannot assign to 'let' value 'user1'*:
![](cannot_assign_let.png)

It's important to keep the different semantics for `let` in mind when dealing with classes and structs in Swift.

##Inheritance & Initializers
Inheritance in Swift works similarly to Objective-C and other popular languages. There are two interesting aspects which I want to discuss in more detail: initializer inheritance and overriding.

Let's add a `sayHi` method to our `User` class so that we can demonstrate method overriding:

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

The syntax for inheritance is the same as for conforming to a protocol, add the type after a colon behind the class name. With this approach you will trigger two compiler errors: *Class 'SpecialUser' has no initializers* and *Overriding declaration requires an 'override' keyword*. 

Let's tackle them one by one. The first error occurs because `greetingMessage` is not initialized by any initializer and doesn't have a default value. We can either add an initializer or assign a default value to `greetingMessage`. 

The second error occurs because Swift explicitly requires us to use the `override` keyword whenever we override an implementation of a superclass.

Here's one way to mute the compiler:

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

We add an initializer that takes a `greetingMessage`. Additionally we add parameters for `name` and `age`. Swift requires us to call the initializer of the superclass, so we need to provide useful values for `name` and `age` as well. An interesting detail is that the call to the `super` initializer is the last statement. In Swift a subclass needs to set up all of its properties before calling the `super` initializer.

One significant difference to Objective-C is that **Swift subclasses don't inherit their superclass initializers when a custom (designated) initializer is provided**. You will learn about designated initializers later in this article. This means the only way to instantiate a `SpecialUser` is calling the `init(name:, age:, greetingMessage:)` initializer. This is a great improvement compared to Objective-C. If all initializers were inherited we could instantiate a `SpecialUser` with `init(name:, age:)` which would leave the `greetingMessage` uninitialized.

The last aspect of initializer inheritance that we will discuss is the difference between `required`, `convenience` and designated initializers.

Because I want to keep the code used in this article short the use cases for these different types of initializers will be somewhat fictional, but they should do a good job at explaining the concepts.

###Required initializers

Using the `required` keyword you can force subclasses to implement an initializer of their superclass:

    class User {
      var name: String = ""
      var age: Int = 0
      
      required init (name: String, age: Int) {
        self.name = name
        self.age = age
      }
      
      func sayHi() -> String {
        return "Hi \(name)"
      }
    }
    
Now you will receive a compiler error in the `SpecialUser` class, because you aren't implementing the required initializer. You can mute the error by adding the initializer to the `SpecialUser` class:

    class SpecialUser : User {
      var greetingMessage: String
      
      init(name: String, age: Int, greetingMessage: String) {
        self.greetingMessage = greetingMessage
        super.init(name: name, age: age)
      }
      
      required init (name: String, age: Int) {
        self.greetingMessage = ""
        super.init(name: name, age: age)
      }
      
      override func sayHi() -> String {
        return "\(greetingMessage) \(name)"
      }
    }
    
Note that you need to use the `required` keyword again to enforce that any subclass down the inheritance hierarchy implements this initializer. If you want subclasses to provide initializers that are consistent with their superclasses, use the `require` keyword.

###Convenience initializers

Convenience initializers are not required to instantiate all properties of a class, instead they are allowed to rely on other initializers. In Swift only convenience initializers are allowed to delegate initialization to other initializers. Let's use a practical example to explore this:

    class SpecialUser : User {
      var greetingMessage: String
      
      init(name: String, age: Int, greetingMessage: String) {
        self.greetingMessage = greetingMessage
        super.init(name: name, age: age)
      }
      
      required init (name: String, age: Int) {
        self.greetingMessage = ""
        super.init(name: name, age: age)
      }
      
      convenience init(name:String) {
        self.init(name: name, age: 0)
      }
      
      convenience init() {
        self.init(name: "Default")
      }
      
      override func sayHi() -> String {
        return "\(greetingMessage) \(name)"
      }
    }

In this example we use convenience initializers to allow the `SpecialUser` to be initialized with a subset of the required parameters. We do this by using *constructor chaining*. The `init()` constructor provides a default name and calls the `init(name:)` constructor. That constructor in turn provides a default age and calls the **designated** initializer `init(name:, age:)`.

Use `convenience` to provide convenient options for initializing your class. Note that it's unfortunately not possible to call `convenience` initializers of a superclass, in my opinion that's a shortcoming of Swift.

###Designated initializers

All Swift initializers are designated initializers by default. Designated initializers are required to fully initialize an instance by assigning values to all non-optional properties. If a class isn't a root class, the designated initializer is also responsible for calling a designated initializer of the superclass. 

All initializers that don't have the `convenience` keyword are designated initializers. Required initializers are just a special form of designated initializer that need to be implemented by every subclass.

##Multiple ownership

As mentioned at the beginning of this article one of the special features of classes is that they support multiple ownership. They can be referenced by multiple variables and properties at the same time. This is not true for value types, they get copied upon every assignment.

When is this feature useful? Two very practical examples are signing up for notifications or being the delegate for another class.  

If you for example want to create a class `UserDataSource` that implements the  `UITableViewDataSource` protocol and becomes the delegate of a `UITableView` you need to pass a *reference* to a `UserDataSource` instance to the `UITableView`. Passing a reference is only possible using classes. So in such cases, where you need multiple parts of your code to work on the same instance, you'll need to resort to classes.

##Deinitialization

Luckily memory management in Swift works vastly automatically. Use cases for deinitialization are rare. A typical one for iOS development is unsubscribing from notifications. In Swift the deinitializer is called directly before the instance is destroyed. Here's an example from a class that subscribes to keyboard notifications and unsubscribes as part of the deinitializer:

      required init() {
        NSNotificationCenter.defaultCenter().addObserver(self,
          selector: "keyboardWillBeShown:",
          name: "UIKeyboardWillShowNotification",
          object: nil
        )
        
        NSNotificationCenter.defaultCenter().addObserver(self,
          selector: "keyboardWillBeHidden:",
          name: "UIKeyboardWillHideNotification",
          object: nil
        )
      }
      
      deinit {
        NSNotificationCenter.defaultCenter().removeObserver(self)
      }
      
You won't need deinitializers frequently, but in some cases, as shown above, they are essential.

##Conclusion

Classes are more complex than structs and enums because they provide additional features such as subclassing and multiple ownership. In some cases these features are necessary, for example when implementing the delegate pattern or registering for notifications. In other cases we are forced to use classes because we need to subclass from common UIKit classes. Classes are an essential part of every iOS app and I hope this article provided a good introduction to some details about their behavior in Swift.

If you want to learn more about Swift and ship your own original iPhone App or iPhone Game you should attend our [Summer Academy](https://makeschool.com/apply?referrer=54750)!

Stay tuned for more tutorials in this series!
