# Backend best practices

In this readme are presented some of the best practices, tools and guidelines for backend applications gathered from different sources.

This Readme contains code examples mainly for TypeScript + NodeJS, but practices described here are language agnostic and can be used in any backend project.

**Check out my other repositories**:

- [Domain-Driven Hexagon](https://github.com/Sairyss/domain-driven-hexagon) - Guide on Domain-Driven Design, software architecture, design patterns, best practices etc.

---

- [Backend best practices](#backend-best-practices)
  - [Architecture](#architecture)
  - [Testing](#testing)
    - [White box vs Black box](#white-box-vs-black-box)
    - [Load Testing](#load-testing)
    - [Fuzz Testing](#fuzz-testing)
  - [API Security](#api-security)
    - [Rate Limiting](#rate-limiting)
  - [Documentation](#documentation)
    - [Document APIs](#document-apis)
    - [Add Readme](#add-readme)
    - [Write self-documenting code](#write-self-documenting-code)
    - [Prefer statically typed languages](#prefer-statically-typed-languages)
    - [Avoid useless comments](#avoid-useless-comments)
  - [Database Best Practices](#database-best-practices)
    - [Backups](#backups)
    - [Managing Schema Changes](#managing-schema-changes)
    - [Data Seeding](#data-seeding)
  - [Configuration](#configuration)
  - [Logging](#logging)
  - [Health monitoring](#health-monitoring)
  - [Static Code Analysis](#static-code-analysis)
  - [Code formatting](#code-formatting)
  - [Make application easy to setup](#make-application-easy-to-setup)
  - [Code Generation](#code-generation)
  - [Version Control](#version-control)
    - [Pre-push/pre-commit hooks](#pre-pushpre-commit-hooks)
    - [Conventional commits](#conventional-commits)
  - [API Versioning](#api-versioning)

## Architecture

Software architecture is about making fundamental choices of your application structure.

> Architecture serves as a blueprint for a system. It provides an abstraction to manage the system complexity and establish a communication and coordination mechanism among components.

Choosing the right architecture is crucial for your application.

We discussed architecture in details in this repository: [Domain-Driven Hexagon](https://github.com/Sairyss/domain-driven-hexagon).

Read more:

- [Software Architecture & Design Introduction](https://www.tutorialspoint.com/software_architecture_design/introduction.htm)

## Testing

Software Testing helps catching bugs early. Properly tested software product ensures reliability, security and high performance which further results in time saving, cost effectiveness and customer satisfaction.

### White box vs Black box

Lets review two types of software testing:

- [White Box](https://en.wikipedia.org/wiki/White-box_testing) testing.
- [Black Box](https://en.wikipedia.org/wiki/Black-box_testing) testing.

Testing module/use-case internal structures (creating a test for every file/class) is called _`White Box`_ testing (or unit testing). _White Box_ testing is widely used technique, but it has disadvantages. It creates coupling to implementation details, so every time you decide to refactor business logic code this may also cause a refactoring of corresponding tests.

Use case requirements may change mid work, your understanding of a problem may evolve or you may start noticing new patterns that emerge during development, in other words, you start noticing a "big picture", which may lead to refactoring. For example: imagine that you defined a unit test for a class, and while developing this class you start noticing that it does too much and should be separated into two classes. Now you'll also have to refactor your unit test. After some time, while implementing a new feature, you notice that this new feature uses some code from that class you defined before, so you decide to separate that code and make it reusable, creating a third class (which originally was one), which leads to changing your unit tests yet again, every time you refactor. Use case requirements, input, output or behavior never changed, but unit tests had to be changed multiple times. This is inefficient and time consuming.

When we have domain models that change often, unit tests end up having to change with them. Traditional unit tests tend to be very coupled to internals of our domain model structure.

To solve this and get the most out of your tests, prefer _`Black Box`_ testing ([Behavioral Testing](https://www.codekul.com/blog/what-is-behavioral-testing/)). This means that tests should focus on testing user-facing behavior users care about (your code's public API), not the implementation details of individual units it has inside. This avoids coupling, protects tests from changes that may happen while refactoring, makes tests easier to understand and maintain thus saving time.

> Tests that are independent of implementation details are easier to maintain since they don't need to be changed each time you make a change to the implementation.

Try to avoid _White Box_ (unit) testing when possible. However, it's worth mentioning that there are cases when _White Box_ testing may be useful. For instance, we need to go deeper into the implementation details when it is required to reduce combinations of testing conditions. For example, a class uses several plug-in [strategies](https://refactoring.guru/design-patterns/strategy), thus it is easier for us to test those strategies one at a time. Or you are developing a library that will be used by multiple modules or projects. In those cases _White Box_ tests may be appropriate.

Use _White Box_ testing only when it is really needed and as an addition to _Black Box_ testing, not the other way around.

It's all about investing only in the tests that yield the biggest return on your effort.

Black Box / Behavioral tests can be divided in two parts:

- Fast: Use cases tests in isolation which test only your business logic, with all I/O (external API or database calls, file reads etc.) mocked. This makes tests fast so they can be run all the time (after each change or before every commit). This will inform you when something fails as fast as possible. Finding bugs early is critical and saves a lot of time.
- Slow: Full [End to End](https://www.guru99.com/end-to-end-testing.html) (e2e) tests which test a use case from end-user standpoint. Instead of injecting I/O mocks those tests should have all infrastructure up and running: like database, API routes etc. Those tests check how everything works together and are slower so can be run only before pushing/deploying. Though e2e tests can live in the same project/repository, it is a good practice to have e2e tests independent from project's code. In bigger projects e2e tests are usually written by a separate QA team.

**Note**: some people try to make e2e tests faster by using in-memory or embedded databases (like [sqlite3](https://www.npmjs.com/package/sqlite3)). This makes tests faster, but reduces the reliability of those tests and should be avoided. Read more: [Don't use In-Memory Databases for Tests](https://phauer.com/2017/dont-use-in-memory-databases-tests-h2/).

For BDD tests [Cucumber](https://cucumber.io/) with [Gherkin](https://cucumber.io/docs/gherkin/reference/) syntax can give a structure and meaning to your tests. This way even people not involved in a development can define steps needed for testing. In node.js world [jest-cucumber](https://www.npmjs.com/package/jest-cucumber) is a nice package to achieve that.

Example files:

- [create-user.feature](https://github.com/Sairyss/domain-driven-hexagon/blob/master/tests/user/create-user/create-user.feature) - feature file that contains human readable Gherkin steps
- [create-user.e2e-spec.ts](https://github.com/Sairyss/domain-driven-hexagon/blob/master/tests/user/create-user/create-user.e2e-spec.ts) - e2e / behavioral test

Read more:

- [Pragmatic unit testing](https://enterprisecraftsmanship.com/posts/pragmatic-unit-testing/)
- [Google Blog: Test Behavior, Not Implementation](https://testing.googleblog.com/2013/08/testing-on-toilet-test-behavior-not.html)
- [Writing BDD Test Scenarios](https://www.departmentofproduct.com/blog/writing-bdd-test-scenarios/)
- Book: [Unit Testing Principles, Practices, and Patterns](https://www.amazon.com/gp/product/1617296279/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=1617296279&linkCode=as2&tag=vkhorikov-20&linkId=2081de4b1cb7564cb9f95526533c3dae)

### Load Testing

For projects with a bigger user base you might want to implement some kind of [load testing](https://en.wikipedia.org/wiki/Load_testing) to see how program behaves with a lot of concurrent users.

Load testing is a great way to minimize performance risks, because it ensures an API can handle an expected load. By simulating traffic to an API in development, businesses can identify bottlenecks before they reach production environments. These bottlenecks can be difficult to find in development environments in the absence of a production load.

Automatic load testing tools can simulate that load by making a lot of concurrent requests to an API and measure response times and error rates.

Example tools:

- [k6](https://github.com/grafana/k6)
- [Artillery](https://www.npmjs.com/package/artillery) is a load testing tool based on NodeJS.

Example files:

- [create-user.artillery.yaml](https://github.com/Sairyss/domain-driven-hexagon/blob/master/tests/user/create-user/create-user.artillery.yaml) - Artillery load testing config file. Also can be useful for seeding database with dummy data.

More info:

- [Top 6 Tools for API & Load Testing](https://medium.com/@Dickson_Mwendia/top-6-tools-for-api-load-testing-7ff51d1ac1e8).
- [Getting started with API Load Testing (Stress, Spike, Load, Soak)](https://www.youtube.com/watch?v=r-Jte8Y8zag)

### Fuzz Testing

[Fuzzing or fuzz testing](https://en.wikipedia.org/wiki/Fuzzing) is an automated software testing technique that involves providing invalid, unexpected, or random data as inputs to a computer program.

Fuzzing is a common method hackers use to find vulnerabilities of the system. For example:

- JavaScript injections can be executed if input is not sanitized properly, so a malicious JS code can end up in a database and then gets executed in a browser when somebody reads that data.
- SQL injection attacks can occur if data is not sanitized properly, so hackers can get access to a database (though modern ORM libraries can protect from that kind of attacks when used properly).
- Sending weird unicode characters, emojis etc. can crash your application.

There are a lot of examples of a problems like this, for example [sending a certain character could crash and disable access to apps on an iPhone](https://www.theverge.com/2018/2/15/17015654/apple-iphone-crash-ios-11-bug-imessage).

Sanitizing and validating input data is very important. But sometimes we make mistakes of not sanitizing/validating data properly, opening application to certain vulnerabilities.

Automated Fuzz testing tools can prevent such vulnerabilities. Those tools contain a list of strings that are usually sent by hackers, like malicious code snippets, SQL queries, unicode symbols etc. (for example: [Big List of Naughty Strings](https://github.com/minimaxir/big-list-of-naughty-strings/)), which helps test most common cases of different injection attacks.

Fuzz testing is a nice addition to typical testing methods described above and potentially can find serious security vulnerabilities or defects.

Example tools:

- [Artillery Fuzzer](https://www.npmjs.com/package/artillery-plugin-fuzzer) is a plugin for [Artillery](https://www.npmjs.com/package/artillery) to perform Fuzz testing.
- [sqlmap](https://github.com/sqlmapproject/sqlmap) - an open source penetration testing tool that automates the process of detecting and exploiting SQL injection flaws

Read more:

- [Fuzz Testing(Fuzzing) Tutorial: What is, Types, Tools & Example](https://www.guru99.com/fuzz-testing.html)

## API Security

Below are some generic recommendations to protect your APIs and ensure at least good basic level of security:

- Ensure you don’t store sensitive information in your Authentication tokens.
- Use [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security) protocol
- Validate all inputs and requests
- Ensure you encrypt all sensitive information stored in your database
- Ensure that you are using safe cryptographic algorithms for encryption
- Never store secrets (passwords, keys, etc.) in the sources in version control (like github). Use environmental variables to store secrets. Put files with your secrets (like `.env`) to `.gitignore`.
- Monitor vulnerabilities in any third party software / libraries you use
- Don’t pass sensitive data in your API queries, for example: https://example.com/login/username=john&password=12345

Read more:

- [OWASP Top Ten](https://owasp.org/www-project-top-ten/)

### Rate Limiting

Enforce a limit to the number of API requests within a time frame, this is called Rate Limiting or API throttling

By default there is no limit on how many request users can make to your API. This may lead to problems, like [DoS](https://en.wikipedia.org/wiki/Denial-of-service_attack) or brute force attacks, performance issues like high response time etc.

To solve this, implementing [Rate Limiting](https://en.wikipedia.org/wiki/Rate_limiting) is essential for any API.

Also enforce rate limiting for login attempts. Lock a user account for specific period of time after a given number of failed attempts

- In NodeJS world, [express-rate-limit](https://www.npmjs.com/package/express-rate-limit) is an option for simple APIs.
- Another alternative is [NGINX Rate Limiting](https://www.nginx.com/blog/rate-limiting-nginx/).
- [Kong](https://konghq.com/kong/) has [rate limiting plugin](https://docs.konghq.com/hub/kong-inc/rate-limiting/).

Read more:

- [Everything You Need To Know About API Rate Limiting](https://nordicapis.com/everything-you-need-to-know-about-api-rate-limiting/)
- [Rate-limiting strategies and techniques](https://cloud.google.com/solutions/rate-limiting-strategies-techniques)
- [How to Design a Scalable Rate Limiting Algorithm](https://konghq.com/blog/how-to-design-a-scalable-rate-limiting-algorithm/)

## Documentation

Here are some useful tips to help users/other developers to use your program.

### Document APIs

Use [OpenAPI](https://swagger.io/specification/) (Swagger) or [GraphQL](https://graphql.org/) specifications. Document in details every endpoint. Add description and examples of every request, response, properties and exceptions that endpoints may return or receive as body/parameters. This will help greatly to other developers and users of your API.

Example files:

- [user.response.dto.ts](https://github.com/Sairyss/domain-driven-hexagon/blob/master/src/modules/user/dtos/user.response.dto.ts) - notice `@ApiProperty()` decorators. This is [NestJS Swagger](https://docs.nestjs.com/openapi/types-and-parameters) module.
- [create-user.http.controller.ts](https://github.com/Sairyss/domain-driven-hexagon/blob/master/src/modules/user/commands/create-user/create-user.http.controller.ts) - notice `@ApiOperation()` and `@ApiResponse()` decorators.

Read more:

- [Documenting a NodeJS REST API with OpenApi 3/Swagger](https://medium.com/wolox/documenting-a-nodejs-rest-api-with-openapi-3-swagger-5deee9f50420)
- [Best Practices in API Documentation](https://swagger.io/blog/api-documentation/best-practices-in-api-documentation/)

### Add Readme

Create a simple readme file in a git repository that describes basic app functionality, available CLI commands, how to setup a new project etc.

### Write self-documenting code

Code can be self-documenting to some degree. One useful trick is to separate complex code to smaller chunks with a descriptive name. For example:

- Separating a big function into a bunch of small ones with descriptive names, each with a single responsibility;
- Moving in-line primitives or hard to read conditionals into a variable with a descriptive name.

This makes code easier to understand and maintain.

Read more:

- [Tips for Writing Self-Documenting Code](https://itnext.io/tips-for-writing-self-documenting-code-e54a15e9de2?gi=424f36cc1604)

### Prefer statically typed languages

Types give useful semantic information to a developer and can be useful for creating self-documenting code. Good code should be easy to use correctly, and hard to use incorrectly. Types system can be a good help for that. It can prevent some nasty errors at a compile time, so IDE will show type errors right away.

Applications written using statically typed languages are usually easier to maintain, more scalable and better suited for large teams.

**Note**: For smaller projects/scripts/jobs static typing may not be needed.

Read more:

- [Static Types vs Dynamic Types](https://instil.co/blog/static-vs-dynamic-types/)

### Avoid useless comments

Writing readable code, using descriptive function/method/variable names and creating tests can document your code well enough. Try to avoid comments when possible and try to make your code legible and tested instead.

Use comments only when it's really needed. Commenting may be a code smell in some cases, like when code gets changed but a developer forgets to update a comment (comments should be maintained, too).

> Code never lies, comments sometimes do.

Use comments only in some special cases, like when writing an counter-intuitive "hack" or performance optimization which is hard to read.

For documenting public APIs use code annotations (like [JSDoc](https://en.wikipedia.org/wiki/JSDoc)) instead of comments, this works nicely with code editor [intellisense](https://code.visualstudio.com/docs/editor/intellisense).

Read more:

- [Code Comment Is A Smell](https://fagnerbrack.medium.com/code-comment-is-a-smell-4e8d78b0415b)
- [// No comments](https://medium.com/swlh/stop-adding-comments-to-your-code-80a3575519ad)

## Database Best Practices

### Backups

Data is one of the most important things in your business. Keeping it safe is a top priority of any backend service.

Here are some basic recommendations:

- Create backups frequently and regularly
- Use remote storages for your backups. Backing up your data and storing it on the same disk as your original data is a road to losing everything. When your storage breaks you will lose an original data and your backups. So keep your backups separately
- Keep backups encrypted and protected. Backup encryption ensures data is protected from leaks and that your data will be what you expect when you recover it
- Consider retention span. Keeping every backup forever isn’t feasible due to a limited amount of space for storage
- Monitor the backup and restore process

Read more:

- [Backup and Recovery Best Practices](https://sqlbak.com/blog/backup-and-recovery-best-practices)
- [The 7 critical backup strategy best practices to keep data safe](https://searchdatabackup.techtarget.com/feature/The-7-critical-backup-strategy-best-practices-to-keep-data-safe)

### Managing Schema Changes

Migrations can help for database table/schema changes:

> Database migration refers to the management of incremental, reversible changes and version control to relational database schemas. A schema migration is performed on a database whenever it is necessary to update or revert that database's schema to some newer or older version.

Source: [Wiki](https://en.wikipedia.org/wiki/Schema_migration)

Migrations can be written manually or generated automatically every time database table schema is changed. When pushed to production it can be launched automatically.

**BE CAREFUL** not to drop some columns/tables that contain data by accident. Perform data migrations before table schema migrations and always backup database before doing anything.

Example: [Typeorm Migrations](https://github.com/typeorm/typeorm/blob/master/docs/migrations.md) - it automatically generates sql table schema migrations like this: [1611765824842-CreateTables.ts](https://github.com/Sairyss/domain-driven-hexagon/blob/master/src/infrastructure/database/migrations/1611765824842-CreateTables.ts)

Read more:

- [What are database migrations?](https://www.prisma.io/dataguide/types/relational/what-are-database-migrations#what-are-the-advantages-of-migration-tools)
- [Database Migration: What It Is and How to Do It](https://www.cloudbees.com/blog/database-migration)

### Data Seeding

To avoid manually creating data in the database, [seeding](https://en.wikipedia.org/wiki/Database_seeding) is a great solution to populate database with data for development and testing purposes.

Example package for nodejs: [typeorm-seeding](https://www.npmjs.com/package/typeorm-seeding#-using-entity-factory).

Example file: [user.seeds.ts](https://github.com/Sairyss/domain-driven-hexagon/blob/master/src/modules/user/database/seeding/user.seeds.ts)

## Configuration

- Store all configurable variables/parameters in config files. Try to avoid using in-line literals/primitives. This will make it easier to find and maintain all configurable parameters when they are in one place.
- Never store sensitive configuration variables (passwords/API keys/secret keys etc) in plain text in a configuration files or source code.
- Store sensitive configuration variables, or variables that change depending on environment, as [environment variables](https://en.wikipedia.org/wiki/Environment_variable) ([dotenv](https://www.npmjs.com/package/dotenv) is a nice package for that) or as a [Docker/Kubernetes secrets](https://www.bogotobogo.com/DevOps/Docker/Docker_Kubernetes_Secrets.php).
- Create hierarchical config files that are grouped into sections. If possible, create multiple files for different configs (like database config, API config, tasks config etc).
- Application should fail and provide the immediate feedback if the required environment variables are not present at start-up.
- For most projects plain object configs may be enough, but there are other options, for example: [NestJS Configuration](https://docs.nestjs.com/techniques/configuration), [rc](https://www.npmjs.com/package/rc), [nconf](https://www.npmjs.com/package/nconf) or any other package.

Example files:

- [ormconfig.ts](https://github.com/Sairyss/domain-driven-hexagon/blob/master/src/infrastructure/configs/ormconfig.ts) - this is typeorm database config file. Notice `process.env` - those are environmental variables.
- [.env.example](https://github.com/Sairyss/domain-driven-hexagon/blob/master/.env.example) - this is [dotenv](https://www.npmjs.com/package/dotenv) example file. This file should only store dummy example secret keys, never store actual development/production secrets in it. This file later is renamed to `.env` and populated with real keys for every environment (local, dev or prod). Don't forget to add `.env` to [.gitignore](https://github.com/Sairyss/domain-driven-hexagon/blob/master/.gitignore) file to avoid pushing it to repo and leaking all keys.

## Logging

- Try to log all meaningful events in a program that can be useful to anybody in your team.
- Use proper log levels: `log`/`info` for events that are meaningful during production, `debug` for events useful while developing/debugging, and `warn`/`error` for unwanted behavior on any stage.
- Write meaningful log messages and include metadata that may be useful. Try to avoid cryptic messages that only you understand.
- Never log sensitive data: passwords, emails, credit card numbers etc. since this data will end up in log files. If log files are not stored securely this data can be leaked.
- Avoid default logging tools (like `console.log`). Use mature logger libraries (for example [Winston](https://www.npmjs.com/package/winston)) that support features like enabling/disabling log levels, convenient log formats that are easy to parse (like JSON) etc.
- Consider including user id in logs. It will facilitate investigating if user creates an incident ticket.
- In distributed systems a gateway can generate an unique correlation id for each request and pass it to every system that processes this request. Logging this id will make it easier to find related logs across different systems/files.
- Use consistent structure across all logs. Each log line should represent one single event and can contain things like a timestamp, context, unique user id or correlation id and/or id of an entity/aggregate that is being modified, as well as additional metadata if required.
- Use log managements systems. This will allow you to track and analyze logs as they happen in real-time. Here are some short list of log managers: [Sentry](https://sentry.io/for/node/), [Loggly](https://www.loggly.com/), [Logstash](https://www.elastic.co/logstash), [Splunk](https://www.splunk.com/) etc.
- Send notifications of important events that happen in production to a corporate chat like Slack or even by SMS.
- Don't write logs to a file from your program. Write all logs to [stdout](https://www.computerhope.com/jargon/s/stdout.htm) (to a terminal window) and let other tools handle writing logs to a file (for example [docker supports writing logs to a file](https://docs.docker.com/config/containers/logging/configure/)). Read more: [Why should your Node.js application not handle log routing?](https://www.coreycleary.me/why-should-your-node-js-application-not-handle-log-routing/)
- Logs can be visualized by using a tool like [Kibana](https://www.elastic.co/kibana).

Read more:

- [Make your app transparent using smart logs](https://github.com/goldbergyoni/nodebestpractices/blob/master/sections/production/smartlogging.md)

## Health monitoring

Additionally to logging tools, when something unexpected happens in production, it's critical to have thorough monitoring in place. As software hardens more and more, unexpected events will get more and more infrequent and reproducing those events will become harder and harder. So when one of those unexpected events happens, there should be as much data available about the event as possible. Software should be designed from the start to be monitored. Monitoring aspects of software are almost as important as the functionality of the software itself, especially in big systems, since unexpected events can lead to money and reputation loss for a company. Monitoring helps fixing and sometimes preventing unexpected behavior like failures, slow response times, errors etc.

Health monitoring tools are a good way to keep track of system performance, identify causes of crashes or downtime, monitor behavior, availability and load.

Some health monitoring tools already include logging management and error tracking, as well as alerts and general performance monitoring.

Here are some basic recommendation on what can be monitored:

- Connectivity – Verify if user can successfully send a request to the API endpoint and get a response with expected HTTP status code. This will confirm if the API endpoint is up and running. This can be achieved by creating some kind of 'heath check' endpoint.
- Performance – Make sure the response time of the API is within acceptable limits. Long response times cause bad user experience.
- Error rate – errors immediately affect your customers, you need to know when errors happen right away and fix them.
- CPU and Memory usage – spikes in CPU and Memory usage can indicate that there are problems in your system, for example bad optimized code, unwanted process running, memory leaks etc. This can result in loss of money for your organization, especially when cloud providers are used.
- Storage usage – servers run out of storage. Monitoring storage usage is essential to avoid data loss.

Choose health monitoring tools depending on your needs, here are some examples:

- [Sematext](https://sematext.com/), [AppSignal](https://appsignal.com/), [Prometheus](https://prometheus.io/), [Checkly](https://www.checklyhq.com/), [ClinicJS](https://clinicjs.org/)

Read more:

- [Essential Guide to API Monitoring: Basics Metrics & Choosing the Best Tools](https://sematext.com/blog/api-monitoring/)

## Static Code Analysis

> Static code analysis is a method of debugging by examining source code before a program is run.

For JavasScript and TypeScript, [Eslint](https://www.npmjs.com/package/eslint) with [typescript-eslint plugin](https://www.npmjs.com/package/@typescript-eslint/eslint-plugin) and some rules (like [airbnb](https://www.npmjs.com/package/eslint-config-airbnb) / [airbnb-typescript](https://www.npmjs.com/package/eslint-config-airbnb-typescript)) can be a great tool to enforce writing better code.

Try to make linter rules reasonably strict, this will help greatly to avoid "shooting yourself in a foot". Strict linter rules can prevent bugs and even serious security holes ([eslint-plugin-security](https://www.npmjs.com/package/eslint-plugin-security)).

> **Adopt programming habits that constrain you, to help you to limit mistakes**.

For example:

Using _explicit_ `any` type is a bad practice. Consider disallowing it (and other things that may cause problems):

```javascript
// .eslintrc.js file
  rules: {
    '@typescript-eslint/no-explicit-any': 'error',
    // ...
  }
```

Also, enabling strict mode in `tsconfig.json` is recommended, this will disallow things like _implicit_ `any` types:

```json
  "compilerOptions": {
    "strict": true,
    // ...
  }
```

Example file: [.eslintrc.js](https://github.com/Sairyss/domain-driven-hexagon/blob/master/.eslintrc.js)

[Code Spell Checker](https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker) may be a good addition to eslint.

Read more:

- [What Is Static Analysis?](https://www.perforce.com/blog/sca/what-static-analysis)
- [Controlling Type Checking Strictness in TypeScript](https://www.carlrippon.com/controlling-type-checking-strictness-in-typescript/)

## Code formatting

The way code looks adds to our understanding of it. Good style makes reading code a pleasurable and consistent experience.

Consider using code formatters like [Prettier](https://www.npmjs.com/package/prettier) to maintain same code styles in the project.

Read more:

- [Why Coding Style Matters](https://www.smashingmagazine.com/2012/10/why-coding-style-matters/)

## Make application easy to setup

There are a lot of projects out there which take effort to configure after downloading it. Everything has to be set up manually: database, all configs etc. If new developer joins the team he has to waste a lot of time just to make application work.

This is a bad practice and should be avoided. Setting up project after downloading it should be as easy as launching one or few commands in terminal. Consider adding scripts to do this automatically:

- [package.json scripts](https://krishankantsinghal.medium.com/scripting-inside-package-json-4b06bea74c0e)
- [docker-compose file](https://docs.docker.com/compose/)
- [Makefile](https://opensource.com/article/18/8/what-how-makefile)
- Database seeding and migrations (described below)
- or any other tools.

Example files:

- [package.json](https://github.com/Sairyss/domain-driven-hexagon/blob/master/package.json) - notice all added scripts for launching tests, migrations, seeding, docker environment etc.
- [docker-compose.yml](https://github.com/Sairyss/domain-driven-hexagon/blob/master/docker/docker-compose.yml) - after configuring everything in a docker-compose file, running a database and a db admin panel (and any other additional tools) can be done using only one command. This way there is no need to install and configure a database separately.

## Code Generation

Code generation can be important when using complex architectures to avoid typing boilerplate code manually.

[Hygen](https://www.npmjs.com/package/hygen) is a great example.
This tool can generate building blocks (or entire modules) by using custom templates. Templates can be designed to follow best practices and concepts based on Clean/Hexagonal Architecture, DDD, SOLID etc.

Main advantages of automatic code generation are:

- Avoid manual typing or copy-pasting of boilerplate code.
- No hand-coding means less errors and faster implementations. Simple CRUD module can be generated and used right away in seconds without any manual code writing.
- Using auto-generated code templates ensures that everyone in the team uses the same folder/file structures, name conventions, architectural and code styles.

**Note**:

- To really understand and work with generated templates you need to understand what is being generated and why, so full understanding of an architecture and patterns used is required.

## Version Control

Make sure you are using version control systems like [git](https://git-scm.com/). Version control systems can track changes you make to files, so you have a record of what has been done, and you can revert to specific versions should you ever need to. It will also help coordinating work among programmers allowing changes by multiple people to all be merged into one source.

Below are some good practices to use together with git.

### Pre-push/pre-commit hooks

Consider launching tests/code formatting/linting every time you do `git push` or `git commit`. This prevents bad code getting in your repo. [Husky](https://www.npmjs.com/package/husky) is a great tool for that.

Read more:

- [Git Hooks](https://githooks.com/)

### Conventional commits

Conventional commits add some useful prefixes to your commit messages, for example:

- `feat: added ability to delete user's profile`

This creates a common language that makes easier communicating the nature of changes to teammates and also may be useful for automatic package versioning and release notes generation.

Read more:

- [conventionalcommits.org](https://www.conventionalcommits.org/en/v1.0.0-beta.2/)
- [Semantic Commit Messages](https://gist.github.com/joshbuchea/6f47e86d2510bce28f8e7f42ae84c716)
- [Commitlint](https://github.com/conventional-changelog/commitlint)
- [Semantic release](https://github.com/semantic-release/semantic-release)

## API Versioning

API versioning is the practice of transparently managing changes to your API

API versioning allows you to incorporate the latest changes in a new version of your API thereby still allowing users to have access to the older version of your API without breaking your users application.

If you need to create a new version of an endpoint, a simple solution would be to create new version of [DTOs](https://github.com/Sairyss/domain-driven-hexagon#DTOs) and a URL like this: `/v2/users`.

Read more:

- [How to Version a REST API](https://www.freecodecamp.org/news/how-to-version-a-rest-api/)
