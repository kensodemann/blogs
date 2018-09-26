# Unit Testing with Mocked Services

When creating unit tests, it is best to concentrate on the module under test. One way to accomplish this is to use mock the services that your module depends on. There are serveral strategies that can be used to do this. The easiest and most effective is to use `mock` objects consisting of Jasmine `spies`.

There are several advantages to using mocked services:

* The test setup is simplified. Without mocks, the test would need to configure inputs to dependent services in order to receive specific outputs. This is often a complex, time consuming, and fragile process.
* Since the outputs of mocks can be controlled very easily, it is easier to control the flow through the test.
* Expected results can be simplfied. Rather than expecting to see potentially obscure effects of calling a specific service the test can be written to actually verify the service was called with specific parameters.
* The code is tested in isolation. Problems with the dependent services cannot cause the test for the current module to fail if a mock is used. Thus the code under test is the only code being tested.
* The tests are easier to understand and reason about. Thus, they are also easier to maintain.


Despite this, there are a couple of disadvantages to using mocked services in tests:

* The dependent services may not perform as advertised. If a mock is written to conform to an advertised behavior, but the actual behavior of the service is different, errors will occur in production. This problem is often mitigated via wide ranging integration testing occuring in combination with the more specifically focused unit tests.
* Maintaining the mocks can be difficult. This problem can be mitigated through clean code organization and applying the [SOLID principles](https://en.wikipedia.org/wiki/SOLID) to the code.


## What are Mocks and Spies?

A `spy` is a function that is used in lieu of an actual function. The `spy` tracks how many times it is called and which paramaters are passed with each call. The test is able to control the value returned by each call of the `spy`.

A `mock` in an object, usually consisting entirely of `spies`, that is used in lieu of an instance of a real object. I practice, a `mock` is typically used in lieu of an Angular service.

## Creating Mocks

The easiest way to create a mock for a service is via `jasmine.createSpyObj()`. This function takes two parameters: a `baseName` that is used as the base name for the spies in the mock, and a `methodNames`. The returned mock will contain the spies with the specified method names.

The `methodNames` parameter can take an array of strings. In this case, the `authSpy` mock will contain two spies, one named `login` and one named `logout`. The return value of these `spies` is not defined. 

```TypeScript
let authSpy = jasmine.createSpyObj('AuthenticationService', [
  'login',
  'logout'
]);
```

The `methodNames` parameter can also take an object. In this case, the `authSpy` mock will contain two spies, one named `login` and one named `logout`. The return value of these `spies` is defined based on the `methodNames` object. In the following example, the `login` spy will return an Observable of `false` and the `logout` spy will return an Observable of `null`.

```TypeScript
let authSpy = jasmine.createSpyObj('AuthenticationService', {
  login: of(false),
  logout: of(null)
});
```

The second form of `jasmine.createSpyObj()` is usually the most useful as it allows the test to define default return values. The return value can then be overridden on a test by test level via code such as:

```TypeScript
authSpy.login.and.returnValue(of(true));
```

### Mocks for Local Services

Local services are generally stored in their own folder along side their own unit test. It is often beneficial to add a module to this folder that creates the `mock` to use for that service. This way, the definition of the mock is centralized, when the service is modified the `mock` can be modified as needed at the same time.

Following this pattern, the folder for the `authentication` service contains the following files:

* `authentication.service.ts` - the service
* `authentication.service.spec.ts` - the unit test for the service
* `authentication.service.mock.ts` - the mock of the service

The `authentication.service.mock.ts` file contains a single function that is used to generate the `mock`. The generated `mock` contains `spies` that all return resonable default values.

```TypeScript
import { of } from 'rxjs';

export function createAuthenticationServiceMock() {
  return jasmine.createSpyObj('AuthenticationService', {
    login: of(false),
    logout: of(null)
  });
}
```

### Mocks for Library Services

It is likely that the same services from other libraries are used throughout the code. A module containing functions that create `mock` objects for these services helps to standarize tests that depend on these libraries. For example:

```TypeScript
export function createPlatformMock() {
  return jasmine.createSpyObj('Platform', {
    ready: Promise.resolve()
  });
}

export function createStorageMock() {
  return jasmine.createSpyObj('Storage', {
    get: Promise.resolve(),
    ready: Promise.resolve(),
    remove: Promise.resolve(),
    set: Promise.resolve()
  });
}
```

## Using Mocks

Using the `mocks` in a test involes the following steps: create the `mocks`, inject the `mocks`, setup the `spies` (as needed), execute the code under test, and `expect` the results.

### Create the Mocks

Before each test, create a new mock object either by calling the `create` functions associated with the services to be mocked or by calling `jasmine.createMockObj()` directly. This should be done before the test module is configured to ensure the object exists when setting up the injector. Creating a new mock before each test ensures that the mock object is clean for each test.

```TypeScript
beforeEach(async(() => {
  authenticationService = createAuthenticationServiceMock();
  navController = createNavControllerMock();
  ...
});
```

### Inject the Mocks

When configuring the testing module, set up the injector to inject the mock via `{ provide: MockedClass, useValue: mockObject }`.

```TypeScript
TestBed.configureTestingModule({
  declarations: [LoginPage],
  imports: [FormsModule, IonicModule],
  providers: [
    { provide: AuthenticationService, useValue: authenticationService },
    { provide: NavController, useValue: navController }
  ],
  schemas: [CUSTOM_ELEMENTS_SCHEMA]
}).compileComponents();
```

### Setup the Spies

In some tests, the value returned by the spy may need to be controlled. In that case, use `spy.and.returnValue()` to set the return value. To set different return values based on the arguments passed, use `spy.withArgs(...).and.returnValue()`. Finally, to set a different return value for each expected call use `spy.and.returnValues([])`.

```TypeScript
authenticationService.login.and.returnValue(of(false));
authenticationService.login.withArgs(
  'ken@test.org',
  'MyCorrectPassword'
).and.returnValue(of(false));
otherService.doIt.and.returnValues(1, 7, 3.14);
```

### Execute the Code and Expect the Results

Call the code under test and verify that the the spy has been called the proper number of times or that it was called with the proper parameters.

```TypeScript
it('performs the login', () => {
  component.signInClicked();
  expect(authenticationServiceMock.login).toHaveBeenCalledTimes(1);
});
```

## Using Matchers

When using the `toHaveBeenCalled()` and `toHaveBeenCalledTimes()` matchers it is generally best to use the more restrictive `toHaveBeenCalledTimes()` matcher when expecting that something has been called and the less restrictive `toHaveBeenCalled()` matcher when expecting that something has _not_ been called.

Let's have a look at some code to test:

```TypeScript
doSomething() {
  if (this.callIt) {
    someService.doIt();
  }
}
```

Here are a couple of test cases to exercise this code:

```TypeScript
it('is called when callIt is true', () => {
  component.callIt = true;
  component.doSomething();
  expect(someService.doIt).toHaveBeenCalledTimes(1);
});

it('is not called when callIt is false', () => {
  component.callIt = true;
  component.doSomething();
  expect(someService.doIt).not.toHaveBeenCalled();
});
```

If `toHaveBeenCalled()` were used in the first test, the test would pass if `someService.doIt()` were called two or more times. The `someService.doIt()` functiom should omly be called once, so the test should fail if it is called any other number of times. 

If `toHaveBeenCalledTimes(1)` where used in the second test, the test would pass if `someService.doIt()` is not called, but it would also pass if `someService.doIt()` were called two or more times. The test should fail any time `someService.doIt()` is called.

## Putting it All Together

Here is a short example test that puts this all together.

```TypeScript
import { CUSTOM_ELEMENTS_SCHEMA } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { async, ComponentFixture, TestBed } from '@angular/core/testing';
import { IonicModule, NavController } from '@ionic/angular';

import { of } from 'rxjs';

import { AuthenticationService } from '../services/authentication/authentication.service';
import { LoginPage } from './login.page';
import { createAuthenticationServiceMock } from '../services/authentication/authentication.service.mock';
import { createNavControllerMock } from '../../../test/mocks';

describe('LoginPage', () => {
  let authenticationServiceMock;
  let navCtrl;

  let component: LoginPage;
  let fixture: ComponentFixture<LoginPage>;

  beforeEach(async(() => {
    authenticationServiceMock = createAuthenticationServiceMock();
    navCtrl = createNavControllerMock();
    TestBed.configureTestingModule({
      declarations: [LoginPage],
      imports: [FormsModule, IonicModule],
      providers: [
        { provide: AuthenticationService, useValue: authenticationServiceMock },
        { provide: NavController, useValue: navCtrl }
      ],
      schemas: [CUSTOM_ELEMENTS_SCHEMA]
    }).compileComponents();
  }));

  beforeEach(() => {
    fixture = TestBed.createComponent(LoginPage);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  describe('clicking the "Sign in" button', () => {
    it('performs the login', () => {
      component.signInClicked();
      expect(authenticationServiceMock.login).toHaveBeenCalledTimes(1);
    });

    it('passes the entered e-mail and password', () => {
      component.email = 'ken@test.org';
      component.password = 'I Drink the Teas';
      component.signInClicked();
      expect(authenticationServiceMock.login).toHaveBeenCalledWith(
        'ken@test.org',
        'I Drink the Teas'
      );
    });

    describe('on success', () => {
      beforeEach(() => {
        authenticationServiceMock.login.and.returnValue(of(true));
        component.email = 'ken@test.org';
        component.password = 'I Drink the Teas';
      });

      it('clears the entered email and password', () => {
        component.signInClicked();
        expect(component.email).toBeFalsy();
        expect(component.password).toBeFalsy();
      });

      it('clears any existing error message', () => {
        component.errorMessage = 'failed to log in';
        component.signInClicked();
        expect(component.errorMessage).toBeFalsy();
      });

      it('navigates to the main page', () => {
        component.signInClicked();
        expect(navCtrl.navigateRoot).toHaveBeenCalledTimes(1);
        expect(navCtrl.navigateRoot).toHaveBeenCalledWith('/tabs/(car-shows:car-shows)');
      });
    });

    describe('on failure', () => {
      beforeEach(() => {
        authenticationServiceMock.login.and.returnValue(of(false));
        component.email = 'ken@test.org';
        component.password = 'I Drink the Teas';
      });

      it('clears just the password', () => {
        component.signInClicked();
        expect(component.email).toEqual('ken@test.org');
        expect(component.password).toBeFalsy();
      });

      it('displays an error message', () => {
        component.signInClicked();
        expect(component.errorMessage).toEqual('Invalid e-mail address or password');
      });

      it('does not navigate', () => {
        component.signInClicked();
        expect(navCtrl.navigateRoot).not.toHaveBeenCalled();
      });
    });
  });
});
```

## Parting Words

Using mocked services in unit tests helps to make the tests easier to write, understand, and maintain. Using mocks also provides better isolation of the code that is under test. Jasmine makes the creation of mock objects very easy. There are a couple of potential downsides to using mock objects, but those can be mitigated by keeping the code clean and organized and by including integration testing in the overall test strategy for the application.

Happy testing!