#### 关于service和component的区别
Service-oriented design and component-oriented design are extremely complementary. The key semantic difference between these two approaches is as follows:
■ In a component-oriented approach, the architect focuses on the provider’s view.
■ In a service-oriented approach, the architect focuses on the consumer’s view
Typically, in a component-oriented approach, the architect is focused on ensuring that
the component they provide is packaged in such a way that it makes their life easier.
You know that when it comes to packaging and deploying Java code, the code will
often be used in a range of different scenarios. For example, a stock-quote program
can be deployed as a console, GUI, or web application by combining different components. A component design approach tries to make it as easy as possible for the architect to select what functionality they want to deploy without hardcoding this into
their application.
This contrasts with a service-oriented approach, where the architect is focused on
supplying a function or set of functions to consumers who typically have little interest
in how the internals of the individual component are constructed, but have specific
requirements for how they want the function to behave. Examples include acid transactions, low latency, and encrypted data.
