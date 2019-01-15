# express-audi-decorators
express decorators which utilize the aurelia-dependency-injection library

## overview

This is a convience library to make creating routes for express REST services cleaner. It relies on the use of the [aurelia-dependency-injection](https://aurelia.io/docs/fundamentals/dependency-injection#introduction) library to create instances of controllers.

## getting started

>*$ npm install express-audi-decorators aurelia-dependency-injection*

---

1. Create models, business logic and any data access classes required

UserService.ts
```typescript

import {autoinject} from 'aurelia-dependency-injection';
import {UserRepository} from '../repositories/UserRepository';

@autoinject
export class UserService {
    private _userRepository: UserRepository;

    constructor(userRepository: UserRepository) {
        this._userRepository = userRepository;
    }

    public async getUser(id: number): Promise<User> {
        let user = await this._userRepository.getUser(id);
        return user;
    }
}

```

2. Create your REST API

UserController.ts
```typescript
import {autoinject} from 'aurelia-dependency-injection';
import {UserService} from '../services/UserService';
import {Response as ExpressResponse} from 'express';
import {Controller, Get, CatchAndSendError, Response, NumParam} from 'express-audi-decorators';

@autoinject()
@Controller('/users')
export class UserController {
    private _userService: UserService;

    constructor(userService: UserService) {
        this._userService = userService;
    }

    @CatchAndSendError()
    @Get('/:userId')
    public async getUser(@Response() res: ExpressResponse, @NumParam('userId') id: number): Promise<void> {
        let user = await this._userService.getUser(id);
        res.location(`/api/users/${user.id}`);
        res.json({user: user});
    }
}

```

3. Set up dependency injection container and start an HTTP listener

index.ts  
```typescript
import {Container} from 'aurelia-dependency-injection';
import {registerController} from 'express-audi-decorators';
import * as express from 'express';
import {Router} from 'express';
import {UserController} from './controllers/UserController';

let container = new Container();
container.makeGlobal(); // creates singleton instance of container

// can register manually instantiated instances if desired

// create express application
const app: express.Application = express();

const apiRouter = Router();

// register controllers
registerController(apiRouter, UserController);

app.use('/api', apiRouter);

// run http listener
const server: Server = app.listen(8080, (error: Error) => {
    if (error != null) {
        console.log(error);
    } else {
        console.log('server listening over insecure http on port 8080');
    }
});
```

4. You should now have a useable api endpoint at:
> localhost:8080/api/users/[id]