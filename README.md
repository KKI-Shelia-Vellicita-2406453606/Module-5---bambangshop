# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1
1. In the classic Observer Pattern described in the Head First Design Patterns book, an interface is used to ensure the "Subject" can notify diverse objects without knowing their specific types. In this bambangShop implementation, a single Subscriber model struct is sufficient for now because the notification logic is uniform, every subscriber is notified by sending an HTTP POST request to a specific URL. However, we would need to implement a Trait (the Rust equivalent of an interface) if we wanted to support different types of notification logic in the future. For instance, if some subscribers required an email while others required a local system log, a trait would allow the Publisher to call a common .update() method on any object that implements the notification behavior, regardless of its internal implementation.

2. While a Vec (list) is a standard way to store collections, it is not the most efficient choice for handling unique identifiers like a url or a product id. If we were to use a Vec, searching for a specific subscriber to delete or verifying uniqueness before adding a new one would require the program to iterate through every element, resulting in a time complexity of O(n). By using a DashMap (a concurrent hash map), we treat the unique url or id as a key, allowing the program to locate, add, or remove entries in nearly instantaneous O(1) time. This structure is necessary for a web application where performance and the prevention of duplicate subscriptions are critical.

3. The implementation already uses a variation of the Singleton Pattern by employing lazy_static to create a single, global instance of the SUBSCRIBERS database that exists for the lifetime of the program. However, simply having a Singleton is not enough in Rust because the compiler enforces rigorous constraints to ensure thread safety. Because multiple web requests (threads) might try to access or modify the subscriber list simultaneously, we need a data structure that can handle concurrent access without causing data races. DashMap provides this necessary synchronization, allowing us to safely mutate our global Singleton state across multiple threads without the program crashing or corrupting data.

#### Reflection Publisher-2
1. The separation of Service and Repository from the Model is a manifestation of the Single Responsibility Principle. While a traditional MVC pattern defines the "Model" broadly, in modern software architecture, a "Model" struct should only serve as a simple data container (DTO) to ensure it is lightweight and portable. The Repository layer is introduced to isolate the complexities of data persistence, whether it's an in memory map like DashMap or an external SQL database ensuring that changes to how data is stored do not affect the rest of the application. Meanwhile, the Service layer acts as the orchestrator of business logic, handling the coordination between different models and repositories. This separation makes the codebase much easier to maintain, test, and scale, as each component has a clear, singular purpose.

2. If we were to combine all logic into the Model, the code would quickly suffer from high coupling and low cohesion. For example, the Product model would not only store product details but would also need to contain logic for database operations and notification triggering. If the Product model needs to interact with Subscriber and Notification directly, it would create a "God Object" that is incredibly complex to debug. As the number of interactions grows such as a product deletion needing to trigger a specific type of notification to a filtered list of subscribers, the logic within the single model would become a tangled mess of dependencies. Testing would also become difficult because we couldn't test the business logic without also triggering the data storage and networking code.

3.Postman is an invaluable tool for testing REST APIs because it allows for the simulation of complex client-server interactions without needing to build a front end interface first. In this tutorial, Postman helps verify that the /subscribe and /unsubscribe endpoints are correctly processing JSON payloads and returning the expected HTTP status codes, such as 201 Created or 404 Not Found. Beyond basic requests, Postman’s features like Environments are particularly helpful for group projects, as they allow developers to switch between localhost and a deployed production URL with a single click. Furthermore, the ability to write Test Scripts in JavaScript within Postman allows for automated verification of response data, which is crucial for maintaining quality in larger software engineering projects.

#### Reflection Publisher-3
