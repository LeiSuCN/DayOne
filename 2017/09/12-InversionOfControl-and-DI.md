### [InversionOfControl](https://martinfowler.com/bliki/InversionOfControl.html)
- 'me': the code written by me.
- One important characteristic of a framework is that the methods defined by the user to tailor the framework will often be called from within the framework itself, rather than from the user's application code. 
- Inversion of Control is a key part of what makes a framework different to a library. A library is essentially a set of functions that you can call, these days usually organized into classes. Each call does some work and returns control to the client.
- UI框架的 Event Listener, DI, ... 都是IoC

### [Inversion of Control Containers and the Dependency Injection pattern](https://martinfowler.com/articles/injection.html)
- *As a result I think we need a more specific name for this pattern. Inversion of Control is too generic a term, and thus people find it confusing. As a result with a lot of discussion with various IoC advocates we settled on the name Dependency Injection.*

