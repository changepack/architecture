# Type checking

| Status | Deciders | Date |
|--------|----------|------|
| Accepted | @nicieja | 2023-03-19 |

## Context and Problem Statement

Static typing in Ruby, like any other programming language, can be helpful for various reasons. Ruby is a dynamically typed language by default, but the introduction of static typing through tools can provide several benefits such as improved code reliability, better readability, enhanced documentation, and improved tooling.

## Decision Drivers

* In an open-source project with multiple contributors, having a consistent and well-documented codebase is crucial. Type annotations can serve as self-documenting code, making it easier for team members to understand and collaborate on the codebase.
* Adopting a type-checking tool can lend concrete structure to some engineering norms that can be enforced by continous integration. By adopting a type-checking tool, a team agrees on certain coding practices and standards. This includes using explicit type annotations, adhering to type contracts between different parts of the code, and consistently following a set of rules defined by the tool.
* TypeScript has become popular for reasons like the benefits it provides to developers and the growing need for more robust and scalable JavaScript applications. It makes sense to consider a similar approach for Ruby so that we can apply the same principles consistenly across front end and back end if Changepack decides to adopt JavaScript.

## Considered Options

The adoption of type-checking in Changepack occurred gradually. This section provides a summary of the steps taken, as seen in retrospect.

1. We began by implementing `ActiveRecord::Attributes` in our models, aiming to achieve better control over data types, improved readability, and enhanced compatibility with non-database-backed attributes.
2. The adoption of Rails Event Store led to the introduction of `dry-struct` and `dry-types` in our codebase. RES treats events as statically-typed structs.
3. After adopting two `dry-rb` libraries, we incorporated `dry-initializer` to help us build service objects more easily. `dry-initializer` works well with dry-types, allowing developers to type params and options for services, components, or other POROs.
4. `dry-rb` has a functional gap, as it only allows static typing during class initialization. Method typing and returned value typing are not covered. This led us to explore Sorbet, a gradual type checker for Ruby that offers both static analysis and runtime checks to catch type-related errors and enforce type safety in your code.
5. Sorbet’s static analysis is performed during the development process and checks the codebase for type-related issues by analyzing type annotations and inferring types based on method signatures and variable assignments.
   1. Sorbet’s static analysis can be challenging to work with, as many third-party libraries may lack type annotations or have incomplete type information. In such cases, Sorbet cannot effectively perform static analysis on the code interacting with these libraries, leading to reduced type safety and increased potential for type-related errors.
   2. Adding type annotations for external dependencies can increase the overall complexity of the codebase. Developers need to understand both the semantics of the third-party libraries and Sorbet’s type system to effectively work with statically typed dependencies.
6. Sorbet’s runtime checks are performed while the code is being executed. Sorbet’s runtime checks involve adding runtime assertions to the code to ensure that the actual types of values at runtime match their expected types, as defined by the type annotations.
7. We rejected Steep early, because it declares types in `.rbs` files in `sig` directory which we wanted to avoid. We believe that by placing the type annotations directly in the Ruby code, developers can more easily understand the types of variables, method arguments, and return values without needing to consult a separate file.

## Decision Outcome

We decided to focus solely on runtime checks for now. This aligns with my natural inclination to extend strong typing from class initialization to method definitions and their returned values in the codebase. Static analysis proved to be a hindrance that slowed down development, as we spent too much time resolving errors without feeling any sense of accomplishment. The idea is that by concentrating on runtime checks, we can reap many benefits of static typing, and once we have fully embraced that style, transitioning to complete static typing in the future will be easier.

This approach aligns with how Stripe, the creators of Sorbet, [initially adopted type annotations.](https://stripe.com/blog/sorbet-stripes-type-checker-for-ruby) At the time, neither Sorbet nor any other static type checker existed to utilize these type annotations; they were only available at runtime. These annotations emerged from a desire to encourage engineers to write modular units with well-defined public interfaces.

### Positive Consequences

* Improved code readability and maintainability. Type checking, especially when combined with type annotations, makes it easier for developers to understand the expected input and output types for functions and methods. This can make the code more self-explanatory, reducing the likelihood of misinterpretation and making it easier to maintain.
* Enforcing contracts. Type checking enforces a contract between different parts of the code, ensuring that functions and methods are used correctly with the expected types. This reduces the likelihood of bugs and increases the overall reliability of the software.
* Catching errors early. Type checking can make it easier for developers to understand each other’s code, reducing the likelihood of miscommunication and bugs when working in a team.
* Enhanced code documentation. Type annotations act as a form of documentation, providing insights into the data structures and types used in the code. This can be particularly useful for developers who are new to a project or reviewing code.

### Negative Consequences

* Learning curve. Developers familiar with Ruby’s dynamic typing may need to learn new concepts and techniques when adopting Sorbet. This learning curve may initially slow down development speed and require extra effort from team members.
* Increased code verbosity. Adding type annotations to Ruby code can make the code more verbose, which might reduce its readability for some developers. This verbosity might be seen as a trade-off between the benefits of explicit type information and the simplicity of Ruby’s original dynamic typing approach.
* Compatibility with third-party libraries. While Sorbet has a growing ecosystem of type definitions for popular Ruby libraries, not all third-party libraries have type annotations or fully support Sorbet. Integrating Sorbet with these libraries might require additional effort to create custom type definitions or workarounds.

## Adopted Conventions

### Focus on typing what matters

Adding type annotations to basic controller actions like `index` or `new` which are universally recognized, does not provide much benefit. Our current goal is not to achieve 100% statically typed code but to concentrate on impactful typing.

### Refrain from typing methods in Phlex components

Unless they have params or options and return something other than HTML tags.

### Use the `Event` struct for events and `T::Struct` for other structs

`Event` is a `T::InexactStruct` which prevents Sorbet from statically checking the types of any arguments passed to the initialize method on the subclass but allows defining `T::Struct` hierarchies. The Sorbet team recommends using this only as a last resort, and we had to resort to it to make the library work with Rails Event Store due to how RES expects events to behave.

To maintain syntax consistency with `ActiveRecord::Attributes` we've added an attribute class method to `T::Struct` which serves as an equivalent to the `const` prop provided by Sorbet natively.

### Refactoring complex type signatures for better readability

Sorbet's patterns may not always align smoothly with implicit Rails conventions, which can result in verbose or disjointed code. When a type signature becomes convoluted, consider splitting it into new type aliases. These aliases can be defined locally within the class or in the `T` module to indicate their common use throughout the system.

Consider the following example, which updates an address and returns it or a collection of all addresses if there are multiple:

```ruby
sig { params(address: { line1: String, line2: T.nilable(String), city: String, code: String, country: String }).returns(T.any({ line1: String, line2: T.nilable(String), city: String, code: String, country: String }, T::Array[{ line1: String, line2: T.nilable(String), city: String, code: String, country: String }])) }
def update(address)
   ...
end
```

This can be refactored into:

```ruby
module T
   Address = T.type_alias do
     { line1: String, line2: T.nilable(String), city: String, code: String, country: String }
   end
end

sig { params(address: T::Address).returns T.any(T::Address, T::Array[T::Address]) }
def update(address)
   ...
end
```

Some commonly used types have already been added:

* `T::Time`: Equivalent to Time, DateTime, or ActiveSupport::TimeWithZone
* `T::Key`: Equivalent to String or Symbol, makes working with hashes easier
* `T::Payload`: For attributes in structs like metadata
* Corresponding types for each model, like `User` and `T::User`, defined at the top of the model
* Collection or relation types for each model, such as `User` and `T::Users`
* `T::String`, `T::Symbol`, etc., for compatibility with `dry-initializer`, which expects the types of object's keywords to be blocks

However, avoid using Changepack's helpers if their behaviors are not needed. For instance, replace `sig { returns T::String }` with `sig { returns String }` as the former does not require the type to be callable.

### Integrate with Active Record’s Attributes API when needed

Use `T.instance(User)` to return an `ActiveRecord::Type::Value` class compatible with `ActiveRecord::Attributes` and suitable for typing attributes of PORO models and service objects that include `ActiveModel::Model`.

Technically, we could type models using `attribute :name, T.instance(String)` but we opt for `attribute :name, :string` to maintain simplicity and consistency with other Rails codebases. Reserve `T.instance` for Ruby objects that require custom casting. This approach aligns with the convention of avoiding Changepack helpers when the same result can be achieved natively, too.
