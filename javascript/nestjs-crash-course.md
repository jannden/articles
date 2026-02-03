## NestJS Crash Course for the Impatient

_Originally published on Aug 20, 2023 [here](https://javascript.plainenglish.io/nestjs-crash-course-1868c443cc33)._

---

NestJS is a popular backend framework that is especially useful when you want to give your project more structure.

In this crash course, we will explore all essential concepts with examples.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*Kh7XbImdrOtDznOTZvVSFQ.png)

DALL-E: an owl using a computer for NestJS, digital art

### What makes NestJS stand out

The advantages are:

- It's with TypeScript.
- Jest for testing.
- Dependency injection for higher efficiency and modularity of your applications.
- Predefined application architecture (similar to Angular) that provides best practices for testability, scalability, and maintainability.

### Installation

Run this command in your terminal to install NestJS:

```react
$ npm install -g @nestjs/cli
```

Create a new project with:

```react
$ nest new communication-app
```

This will create a new directory called `communication-app` containing the basic setup. Let's switch to that directory:

```react
$ cd ./communication-app
```

To start the server, run the following command:

```react
$ npm run start:dev
```

When you open [http://localhost:3000](http://localhost:3000/) in your browser now, you should see the *'Hello World!'* message.

### Starter project

Let's have a look at what's inside the starter project.

Open the `./src` folder and have a look at the files:

- `app.controller.ts `--- specifies which routes the server will respond to
- `app.controller.spec.ts `--- a test file for the controller
- ` app.``service``.ts ` --- contains the actual logic serving the controller
- `app.module.ts `---brings it all together in one neat package
- `main.ts`---the entry point for the whole app

### NestJS structure

NestJS has a pretty similar structure to Angular. If you are coming from a React background, it might feel a bit overwhelming at first. Let's dive into the structure step by step.

### Modules

Modules help organize your application into smaller parts. Each module groups related code, like controllers and services, making it easier to manage the codebase.

When you create a module, you specify what other modules it needs to work correctly, what services it provides, and what routes it handles.

Creating a new module is easy with the NestJS command-line tool. Let's generate a User module:

```react
$ nest generate module user
```

It will automatically generate the `./src/user/user.module.ts` file and connect it to the *AppModule*.

### Models

Models represent a specific type of data that your application will work with.

Create a new file `user/user.model.ts` and fill it with this content:

```react
export interface User {
  id: number;
  name: string;
  email: string;
}
```

The `User` interface represents a user object with three properties: `id`, `name`, and `email`. Each property is assigned a specific data type, such as `number` or `string`. This tells the application what it will work with.

### Services

Services are classes that contain the core logic of your application, like handling user actions.

The service classes are annotated with the `@Injectable` decorator so that they can be injected into controllers or other services using the dependency injection system.

Let's generate a *User* service. Open your project directory and simply type this into your terminal:

```react
$ nest generate service user
```

That command will generate a boilerplate for a NestJS service. You can extend it like this:

```react
import { Injectable } from '@nestjs/common';
import { User } from './user.model';

@Injectable()
export class UserService {
  private readonly users: User[] = [
    { id: 1, name: 'John Doe', email: 'john.doe@example.com' },
    { id: 2, name: 'Jane Doe', email: 'jane.doe@example.com' },
  ];

  findAll(): User[] {
    return this.users;
  }

  findOne(id: number): User {
    return this.users.find((user) => user.id === id);
  }

  create(userDto: Omit<User, 'id'>): User {
    const user = { ...userDto } as User;
    user.id = this.users.length + 1;
    this.users.push(user);
    return user;
  }
}
```

The `findAll()` method simply returns the entire `users` array, while the `findOne()` method takes an `id` argument and uses the `Array.find()` method to retrieve the user with the matching ID.

Then we added a `createUser()` method that takes an argument of type `User` omitting the `id`. It generates a new `id` for the user and adds the user to the array of `users`.

These are just simple example methods, but in a real-world application, you would likely have more complex logic for retrieving and manipulating user data, usually including a connection to the database.

### Data transfer objects (DTOs)

We saw that the `create` method of the `UserService` takes an argument of type `User` omitting the `id`. That's cumbersome and not reusable. We will change it now.

First, let's install an often-used library `class-validator`:

```react
$ npm install class-validator
```

This library helps us to specify what types are allowed within our data-transfer objects (DTOs).

Let's create a DTO for creating a user in `user/user.dto.ts`:

```react
import { IsEmail, IsString } from 'class-validator';

export class UserDto {
  @IsString()
  readonly name: string;

  @IsEmail()
  readonly email: string;
}
```

Now we can update the `create` method of `UserService` in `user/user.service.ts`:

```react
import { UserDto } from "./user.dto"

// ...

  // instead of: create(userDto: Omit<User, 'id'>): User {
  create(userDto: UserDto): User {

// ...
```

### Providers

Providers are a way to define reusable code. They usually contain logic that is not specific to a controller or module but is more general like connecting to a database, sending emails, or performing calculations.

Providers are defined as classes with the `@Injectable` decorator, just like services.

We will create a provider `user/logger.provider.ts` to log useful messages:

```react
import { Injectable } from '@nestjs/common';

@Injectable()
export class Logger {
  private readonly messages: string[] = [];

  log(message: string) {
    this.messages.push(message);
    this.printAllMessages();
  }

  printAllMessages(): void {
    console.log(this.messages);
  }
}
```

The `Logger` class again has the `@Injectable()` decorator, which makes it injectable as a provider in other parts of a NestJS application. The class has a private `messages` array property that stores all logged messages.

The `log()` method takes a `message` argument and adds it to the `messages` array, while the `printAllMessages()` method uses `console.log` to print the entire `messages` array to the terminal.

In order to make a provider available in the module, we need to import it into the *module *file. This is because we created the provider by hand and not by a NestJS CLI command as with the *service* we created earlier. CLI commands are available only for *modules, services, and controllers.*

Let's update `user/user.module.ts` like this:

```react
import { Module } from '@nestjs/common';
import { UserService } from './user.service';
import { Logger } from './logger.provider';

@Module({
  providers: [UserService, Logger],
})
export class UserModule {}
```

### Controllers

Controllers are responsible for handling incoming requests and returning responses to the client. In other words, controllers define the routes and associated request handlers that allow somebody to interact with your application.

Each controller is a class annotated with the `@Controller` decorator, which specifies the base route for all the routes defined in the controller. For example, if you have a *UserController *with a base route of `/user`, all the routes defined in that controller will be prefixed with `/user`.

The controller class contains methods that handle incoming requests. These methods are annotated with the HTTP verb (like `@Get`, `@Post`, `@Delete`, `@Put`) and the sub-route that they handle.

Let's generate a *User* controller with a NestJS CLI command:

```react
$ nest generate controller user
```

We can fill the newly generated `user/user.controller.ts` with this content:

```react
import { Body, Controller, Get, Param, Post } from '@nestjs/common';
import { Logger } from './logger.provider';
import { UserService } from './user.service';
import { User } from './user.model';
import { UserDto } from './user.dto';

@Controller('user')
export class UserController {
  constructor(
    private readonly userService: UserService,
    private readonly logger: Logger,
  ) {}

  @Get()
  findAll(): User[] {
    this.logger.log('Getting all users');
    return this.userService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string): User {
    this.logger.log(`Getting user with id ${id}`);
    return this.userService.findOne(+id);
  }

  @Post()
  create(@Body() userDto: UserDto): User {
    this.logger.log(`Creating user with name ${userDto.name}`);
    const user = this.userService.create(userDto);
    return user;
  }
}
```

To see it in action, start the server:

```react
$ npm run start:dev
```

And open <http://localhost:3000/user> in your browser. Hopefully, you will see a list of users.

### Lifecycles

Lifecycles are different stages of an application (initialization, starting, and stopping). NestJS provides hooks that allow you to run code at those specific moments. These hooks can be used to perform tasks such as connecting to a database, setting up the configuration, or starting a background task. Here are examples of such hooks:

- OnModuleInit---called when the host module has been initialized
- OnApplicationBootstrap --- called when the application has fully started
- OnModuleDestroy---called just before the host module is destroyed
- OnApplicationShutdown--- called when the application shuts down

Usually, lifecycles are implemented in Services or Providers, but for a clearer example, we will create a separate file `user/user.lifecycle.ts` and fill it with this code:

```react
import { Injectable, OnModuleInit } from '@nestjs/common';

@Injectable()
export class UserLifecycle implements OnModuleInit {
  onModuleInit() {
    console.log(`The module has been initialized.`);
  }
}
```

Now, let's connect it to the *UserModule* by updating the `user/user.module.ts` file, which will look like this:

```react
import { Module } from '@nestjs/common';
import { UserController } from './user.controller';
import { UserService } from './user.service';
import { Logger } from './logger.provider';
import { UserLifecycle } from './user.lifecycle';

@Module({
  controllers: [UserController],
  providers: [UserService, Logger, UserLifecycle],
})
export class UserModule {}
```

Thanks to this, you will see a console log every time the module gets initialized.

### Pipes

Pipes are a mechanism for processing and validating input data before it is handled by a controller method or passed to a service. For example, you can define a pipe that transforms a string into a number, and then use it in a controller method to ensure that the input is in the correct format.

NestJS has a couple of built-in pipes out of the box, such as for parsing dates, transforming data to a specific type, or validating data against a schema. But the real power comes with custom pipes, as I will show you now.

We haven't installed the `class-transformer` yet, but we will need it now, so let's run this command:

```react
$ npm install class-transformer
```

Let's create one in `user/user.pipe.ts` and fill it with this content:

```react
import { Injectable, PipeTransform } from '@nestjs/common';
import { UserDto } from './user.dto';

@Injectable()
export class EnrichedUserPipe implements PipeTransform {
  transform(userDto: UserDto): UserDto {
    const enrichedUser: UserDto = {
      ...userDto,
      createdAt: new Date(), // enriching the user with creation timestamp
    };
    return enrichedUser;
  }
}
```

The `EnrichedUserPipe` adds creation date to the user data. We will use it in the `UserService` thanks to the `@UsePipes` decorator, which we will put just above the `create` method in `user/user.service.ts` like this:

```react
import { UsePipes } from '@nestjs/common';
import { EnrichedUserPipe } from './user.pipe';

// ...

 @UsePipes(new EnrichedUserPipe())
 create(@Body() userDto: UserDto): User {

// ...
```

Since we enhanced our User Model and DTO, we should update those as well.

Here is the updated `user/user.dto.ts`:

```react
import { IsDate, IsEmail, IsString } from 'class-validator';

export class UserDto {
  @IsString()
  readonly name: string;

  @IsEmail()
  readonly email: string;

  @IsDate()
  readonly createdAt?: Date;
}
```

And here is the updated `user/user.model.ts`:

```react
export interface User {
  id: number;
  name: string;
  email: string;
  createdAt?: Date;
}
```

### Testing

NestJS uses the Jest testing framework by default which makes it easy to test controllers, services, and providers.

Here is an example `user/user.controller.spec.ts` where we test the `findAll` method of `UserService`:

```react
import { Test, TestingModule } from '@nestjs/testing';
import { UserController } from './user.controller';
import { UserService } from './user.service';
import { Logger } from './logger.provider';
import { User } from './user.model';

describe('UserController', () => {
  let userController: UserController;
  let userService: UserService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      controllers: [UserController],
      providers: [UserService, Logger],
    }).compile();

    userController = module.get<UserController>(UserController);
    userService = module.get<UserService>(UserService);
  });

  describe('findAll', () => {
    it('should return an array of users', async () => {
      const result: User[] = [
        { id: 1, name: 'John Doe', email: 'john.doe@example.com' },
        { id: 2, name: 'Jane Doe', email: 'jane.doe@example.com' },
      ];

      jest.spyOn(userService, 'findAll').mockImplementation(() => result);

      expect(userController.findAll()).toBe(result);
    });
  });
});
```

### Conclusion

Hope you got a good grasp on NestJS in this article 😊

If you found it helpful, please click the clap 👏 button.
And feel free to comment! I'd be happy to help :)
