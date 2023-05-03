# Domain modelling at Changepack

| Status | Deciders | Date |
|--------|----------|------|
| Accepted | @nicieja | 2023-05-02 |

## Context and Problem Statement

Domain modelling plays a pivotal role in creating applications that are adaptable, efficient, and easy to maintain. In this post, we will be exploring our domain modelling strategy, which combines Rails best practices with non-standard approaches to create a robust, future-proof architecture that can adapt to changing requirements and withstand the test of time.

## Decision Drivers

Recognizing the inherent flexibility of Rails and its potential for generating bad code if left unchecked, it is crucial to define the architectural approach explicitly and up-front. Several key decision drivers can help guide this process, ensuring a clean, maintainable, and scalable codebase:

* Rails conventions vs. alternative patterns: Strive for a pragmatic balance between adhering to Rails conventions and introducing non-standard patterns, focusing on the application's long-term success, scalability, and maintainability.
* Modularity and separation of concerns: Ensure that each component of the application is responsible for a single aspect of functionality. This modular design makes it easier to identify, manage, and update code as needed without impacting other unrelated parts of the application.
* Code organization and structure: Adopting a well-defined directory structure and naming conventions for files and classes will promote readability and ease of navigation, making it simpler for developers to understand the codebase and avoid inadvertently introducing spaghetti code.
* Code reviews and pair programming: Establish a unified approach to consistently evaluate new code, minimizing debates over foundational design patterns and streamlining code reviews and pair programming sessions.

## Considered Options

* As we explored various architectural options for Changepack, we initially considered two primary approaches: adhering strictly to The Rails Way, embracing models and concerns, or adopting a non-standard approach like Domain-Driven Design and event sourcing.
* The second architectural consideration was whether to keep the app monolithic or adopt a microservices approach. Both options have their pros and cons, which needed to be carefully weighed against the specific needs of our application.
* The third consideration in our architectural decision-making process was determining whether to focus on rich models or build a separate service layer using Pure Ruby Objects to encapsulate domain logic. A primary concern was avoiding the pitfall of creating fat models, which can become unwieldy and difficult to maintain. In terms of implementing service objects, we considered using libraries such as `dry-initializer` or `trailblazer` to facilitate their creation and management.

## Decision Outcome

Monolithic architectures are characterized by their tightly coupled components, which can simplify prototyping and speed up the development process. However, as the application grows, monoliths can suffer from a lack of perceivable structure, leading to the dreaded "Big Ball of Mud"—a software design anti-pattern that results in a tangled and unmaintainable codebase.

On the other hand, microservices offer a loosely coupled architecture, allowing for better code organization and separation of concerns. This modularity can lead to a more robust and maintainable system in the long run. However, adopting microservices can also result in a more complex development process, increased operational overhead, and potential challenges in coordinating and communicating between services. Such an architecture may not be the best fit for most apps, especially given Rails’ love for monoliths.

Changepack's architecture can be described as a blend of simplicity and complexity, which is reflected in its various components. On one hand, the blog (changelog) aspect of the application is relatively simple and follows a standard structure, encompassing elements such as accounts, blogs, and posts. On the other hand, the application becomes more advanced and intricate when it comes to other parts of the system, particularly those that involve integrations with external applications like GitHub, Linear, or ChatGPT. These components demand a more sophisticated architectural approach, as they require seamless communication, data handling, and coordination between different services.

This is why we opted for a hybrid approach that combines the best of both worlds. We decided to primarily adhere to The Rails Way for the majority of our architecture while incorporating a lightweight, event-based design for specific components where it adds value. This decision was driven by Changepack's focus on integrations, which necessitates a significant amount of behind-the-scenes back-end communication. We don’t use event sourcing, though.

Why use events?

* Events are commonly asynchronous, eliminating temporal coupling.
* Events reduce coupling between components. By adhering to a defined schema, the event publisher only needs to know what to publish, without any knowledge of the subscribers. This approach allows for the seamless addition of new subscribers without altering the event publishing code.
* Buffers. If a subscriber is temporarily offline due to deployment, maintenance, or failure, events will accumulate in the queue. Once the subscriber is back online, it will process the backlog of events, eventually achieving consistency with the rest of the system. This level of resilience would be difficult to achieve with synchronous communication methods.

You might have noticed a focus on reducing coupling, which might seem contradictory when adopting The Rails Way, as it often involves embracing coupling to streamline development. To strike the right balance, we incorporated a concept from Domain-Driven Design called Bounded Context.

Bounded Context is a principle where a large system is divided into smaller, more manageable contexts, each with a clearly defined boundary and responsibility. These contexts encapsulate specific areas of the domain and help maintain a clear separation of concerns.

In the case of Changepack, we divided the app into several contexts such as platform, connect, write, and so on. Within each context, we allow for tight coupling, where models can reference each other through relationships, and classes can be used directly. This approach simplifies development and adheres to The Rails Way within individual contexts.

However, to ensure maintainability and flexibility, we embrace loose coupling between contexts, minimizing direct interactions between components across different contexts. This is where the event-based architecture becomes valuable. By utilizing events as the primary communication mechanism between different contexts, we effectively reduce coupling and maintain a clean, modular architecture that can adapt to changing requirements and scale effectively.

To provide a clear example of how the principle is applied in Changepack, let's examine the interaction between the `connect` and `write` contexts. In the connect context, we have several models like `Repository` and `Commit` (for Git) as well as `Team` and `Issue` (for project management tools.) Commits and Issues are used to feed OpenAI's ChatGPT, which then generates posts automatically. However, this falls under the responsibility of the write context.

To facilitate the interaction between these two contexts while maintaining the separation of concerns, we introduced two additional models called `Source` and `Update`. Sources represent Repositories and Teams, while Updates represent Commits and Issues. These new models are created based on events sent from the connect context to the write context.

This approach allows for clean communication between the two contexts without direct dependencies or tight coupling. By utilizing events and the additional models, the connect context can effectively trigger the necessary actions in the write context without compromising the overall architectural integrity and separation of concerns. We facilitate clean communication between the two contexts without direct dependencies or tight coupling. Bounded contexts function similarly to modules or microservices, but without the additional overhead that might come with full-fledged microservices. Within each context, we continue to utilize the tried-and-tested Rails Way, ensuring a rapid development flow.

As the project grows in the future, different individuals or teams could be assigned to maintain each context. Keeping these contexts relatively independent allows for a more efficient development process, as teams can focus primarily on their respective bounded context without being bogged down by external dependencies. This separation of concerns not only enhances code maintainability but also promotes fast flow and productivity within the development teams, enabling them to deliver high-quality features and improvements more quickly.

Now let’s talk about models.

After evaluating the trade-offs, we decided to adopt rich models and forgo the use of service objects. The domain logic is maintained within the models, and we view these domain models as our application API. As a guiding design principle, we aim for this API to feel as natural and intuitive as possible.

To access business logic via domain models, some core domain entities inevitably end up offering a wide range of functionality. To avoid issues related to the dreaded fat model problem, we utilize concerns to organize our model's code. This approach is well-explained in posts like ["Vanilla Rails is plenty"](https://dev.37signals.com/vanilla-rails-is-plenty/) and ["Active Record, nice and blended"](https://dev.37signals.com/active-record-nice-and-blended/) by 37signals. The reason we decided against adopting service objects is that we didn't want to end up with anemic domain models, as described by Martin Fowler in his article on ["Anemic domain models."](https://martinfowler.com/bliki/AnemicDomainModel.html)

Our data models strive for self-documentation within their Ruby files:

* We utilize [ActiveRecord::Attributes](https://api.rubyonrails.org/classes/ActiveRecord/Attributes/ClassMethods.html) to document attributes and their respective types.
* Non-obvious behavior is clarified through comments.
* IDs are defined using `key` with a prefix like `src`, resulting in IDs formatted as `src_xxx`. We employ these prefixes because they provide immediate recognition of the ID type when examining logs or stack traces, significantly improving the debugging process.

For value objects on models, we employ the `store_model` gem. The StoreModel gem enables wrapping JSON-backed database columns with ActiveModel-like classes, allowing us to treat them as mini-models capable of encapsulating domain logic even though they are JSONs behind the scenes. For instance, each commit has an author, and `Commit#author` is a `jsonb` column in the commits table. This column would be too small to warrant its own ActiveRecord model, but it can still contain logic, such as mapping a commit author to their account on Changepack.

In Rails, a similar approach can be achieved using `composed_of`. However, this method is less explicit, as the object is constructed by Ruby at runtime based on selected attributes rather than being stored in an easily accessible form within the database. By utilizing the `store_model` gem, we can achieve a more transparent and structured implementation of value objects on our models.

### Positive Consequences

1. Improved maintainability: By adopting Bounded Contexts and organizing code using concerns, the overall code structure becomes more modular and easier to maintain over time.
2. Faster development flow: Embracing The Rails Way within individual contexts helps streamline the development process, leading to quicker iterations and feature implementations.
3. Clear communication between contexts: Using events as a communication mechanism between Bounded Contexts ensures clean, decoupled interactions that contribute to the overall stability and reliability of the application.
4. Future-proof design: The architectural choices that promote modularity, separation of concerns, and event-based communication help to ensure that the application can more easily adapt to changing requirements and new features over time.

### Negative Consequences

1. Complexity in managing boundaries: While Bounded Contexts provide many benefits, they can also introduce complexity in defining and managing the boundaries between contexts, which could require more effort during the development process.
2. Potential for event handling issues: An event-based architecture might introduce new challenges related to event handling, such as ensuring correct event sequencing, handling event failures, and dealing with duplicate events.
3. Learning curve for developers: The combination of various architectural patterns and techniques, such as Bounded Contexts, event-based communication, and rich models, may require developers to invest time in learning and adapting to these concepts, especially if they are not familiar with them.
4. Risk of over-engineering: Balancing the use of different architectural patterns could lead to over-engineering in some parts of the system, which may result in additional complexity and slower development.
5. Performance considerations: While event-based communication offers many advantages, it may also introduce performance implications, such as latency in processing events or the need to manage and monitor event queues. These aspects may require additional resources and attention during development and maintenance.
