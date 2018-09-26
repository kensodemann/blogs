# Unit Testing

With the upcoming release of Ionic v4, projects that use `@ionic/angular` are tightly integrated with the Angular CLI. One exciting aspect of this is that Ionic v4 projects are created with unit tests supported out of the box.

Since we are using the Angular CLI to do most of the heavy lifting, most of the important details pertaining to unit testing an `@ionic/angular` application is contained in the <a href="https://angular.io/guide/testing" target="_blank">Angular Testing Guide</a>. I suggest keeping that guide handy as we work through unit testing Ionic applications. The goal of this series of articles is not to replace that guide, but rather to kick-start the unit testing process and to provide some best practices that will help successfully create productive test suites.

## The Role of Testing

Why write unit tests in the first place? Writing tests may add some time to the initial development effort. However, that extra effort pays for itself over time in the form of reduced maintenance costs. Well written tests allow:

* bugs to be found and fixed earlier in the development cycle when they are cheaper and easier to fix
* features or bug fixes to be added without introducing new bugs
* the code to be refactored more confidently
* the behavior of code constructs to be exercised and examined

Notice the emphasis on uncovering bugs. That is intentional. The purpose of testing is *not* to verify that the code is correct. Rather, the purpose of testing is to find where the code is broken. There are many reasons for this distinction. The most important reason is that we tend to succeed in what we set out to do. If we are trying to verify that the code works, that is exactly what we are likely to do. Thus, we are more likely to just test the simple paths. On the other hand, if our goal is to find bugs we are more likely to test the edge cases and the limits of the code. This is where the bugs tend to hide. If we set out to find as many bugs as possible, that is what we are going to do, and the code will be better for it.

Keeping these goals in mind, good test suites have the following attributes:

* the tests are fast, independent, and repeatable
* the test are expandable, understandable, and maintainable
* the tests validate a wide variety of different inputs 
* the tests handle edge cases and limits
* as much as possible the tests are tied to requirements rather than to implementation

## Getting Started

Let's start by generating a new Ionic project. From the command prompt, type `ionic start unit-tests tabs --type=angular`. When asked answer `N` to the questions about Cordova and including Ionic Pro. This creates an Ionic application using the `tabs` starter template. Let's take a look around.

Under the `src` directory the following tests were generated with the starter application:

* `src/app/app.component.spec.ts`
* `src/app/pages/tabs/tabs.page.spec.ts`
* `src/app/pages/home/home.page.spec.ts`
* `src/app/pages/contact/contact.page.spec.ts`
* `src/app/pages/about/about.page.spec.ts`

For now, let's just try running the tests to make sure everything works. From the root directory of the project, type `npm test`. The project will compile, chrome will launch, and the tests will run. You should see 6 passed tests.

Notice that the test command does not exit. It is waiting and will re-run as code changes are saved. When the code is under development, it is often desirable to continuously run the tests as changes are made. Sometimes, such as when doing a continuous integration build, the desired behavior is to run the tests once and exit. To accomplish this, type `npm test -- --watch-=false`.

## General Test Structure

A typical unit test for a page looks like this:

```typescript
import { CUSTOM_ELEMENTS_SCHEMA } from '@angular/core';
import { async, ComponentFixture, TestBed } from '@angular/core/testing';

import { ContactPage } from './contact.page';

describe('ContactPage', () => {
  let component: ContactPage;
  let fixture: ComponentFixture<ContactPage>;

  beforeEach(async(() => {
    TestBed.configureTestingModule({
      declarations: [ContactPage],
      schemas: [CUSTOM_ELEMENTS_SCHEMA],
    }).compileComponents();
  }));

  beforeEach(() => {
    fixture = TestBed.createComponent(ContactPage);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  describe('onAddContactClicked', () => {
    it('', () => {
      ...
    });
    ...
  });
});
```

Key elements of this test are:

* the whole test is wrapped by a `describe()` that defines the module being tested
* the code in the `beforeEach()` callbacks is executed before each test
* it is possible to define multiple `beforeEach()` callbacks
* the first `beforeEach()` above wraps the callback in Angular's [async(https://angular.io/api/core/testing/async)]() testing function, this is one way to handle asynchronous code, other methods will be examined in future articles
* calls to `describe()` can be nested to describe functionality within the module, this is often done by method or by a specific behavior or condition
* each `describe()` can have its own set of `beforeEach()` callbacks defined
* the test cases are defined by via the `it()` functions

## Conclusion

This article just scratches the surface of this subject. In future articles, I will cover:

* mocking dependencies
* testing asynchronous code
* testing pages and components
* testing services and pipes
* end to end testing and how it differs from unit testing

If there are other topics that you would like to see covered, please let me know in the comments.