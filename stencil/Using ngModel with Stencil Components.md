# Using ngModel with Stencil Components

[Stencil](https://stenciljs.com) is a tool for building modern Web Components. Stencil combines some of the best features from traditional frameworks but outputs 100% standards-compliant Custom Elements, part of the Web Component spec. The Web Component collections created with Stencil can then be used in projects that use any application framework, including no framework at all.

When you create a component that will be used in a form you want to be able to bind it to an [NgModel](https://angular.io/api/forms/NgModel). This article explains how to integrate your Web Components into forms on [Angular](https://angular.io) projects using [Control Value Accessors](https://angular.io/api/forms/ControlValueAccessor). A control value accessor is an interface that acts as a bridge between your component and the Angular forms API, allowing you to use ngModel, form validation, and the like with your component.

### Creating a Component

When creating your components, there are a few guidelines that you should follow to make sure your component will work well with Angular or other frameworks:

1. Make small, concrete UI components. Generally, these components should represent leaf nodes and are objects that cannot be broken down further.
1. Break monolithic components into smaller components. For example, if you have a list if items, do not just have a single component for the list that is set up via JavaScript. Rather, have a list component that itself contains item components.
1. Use attributes and properties to configure the component.
1. Use properties rather than methods to change the state of the component.
1. Use events to listen for changes in the control.

Following these guidelines makes the components that you are designing work just like existing HTML elements. This is just general good design and results in components that will integrate well with existing frameworks.

### Wiring it Up

Given a small, concrete UI component that uses a `value` property for its input and signals changes via the `input` event, we could manually wire it up as such:

```html
<my-special-input [value]="myData" (input)="myData=$event.target.value"></my-special-input>
```

This gets us our two-way binding in so far as `myData`'s value is concerned. However, in Angular, we really want to use `ngModel` so we can take advantage of features such as form validations. Something like this: 

```html
<my-special-input [(ngModel)]="myData"></my-checkbox>
```

This makes our custom input element behave just like a standard input element. The good news is that this is not only possible but is also very easy to do via the use of Control Value Accessors.

#### Using Default Control Value Accessor

The API conventions that have been established for inputs is that it has a property called `value` that is used to set and change the state of the input. Changes that the input makes to the value as the user interacts with it are communicated back via the `input` event.

If you have a custom element that follows these conventions, you can use `ngModel` with it by using the [Default Value Accessor](https://angular.io/api/forms/DefaultValueAccessor) via the `ngDefaultControl` directive.

For example, the following custom element follows the `input` API conventions:

```tsx
import { Component, Prop, PropWillChange } from '@stencil/core';

@Component({
  tag: 'test-vowel-input',
  styleUrl: 'vowel-input.scss'
})
export class NameEntry {

  @Prop() value: string;
  private parsedValue: string;

  @PropWillChange('value') handleValueChange(newValue: string) {
    this.parseValue(newValue);
  }

  componentWillLoad() {
    this.parseValue(this.value);
  }

  render() {
    return (
      <div>
        <input placeholder="Enter Only Vowels"
          value={this.parsedValue}
          onKeyDown={(event: UIEvent) => this.limitToVowels(event as any as KeyboardEvent)}/>
      </div>
    );
  }

  private limitToVowels(evt: KeyboardEvent) {
    if (this.isEditKey(evt) || this.isVowelKey(evt)) {
      return;
    }

    evt.preventDefault();
  }

  private isEditKey(evt: KeyboardEvent) {
    return evt.keyCode === 46 || evt.keyCode === 8 || evt.keyCode === 9 || evt.keyCode === 27 || evt.keyCode === 13 ||
      evt.keyCode === 110 || evt.keyCode === 190 ||
      (evt.keyCode == 65 && (evt.ctrlKey === true || evt.metaKey === true)) ||
      (evt.keyCode == 67 && (evt.ctrlKey === true || evt.metaKey === true)) ||
      (evt.keyCode == 88 && (evt.ctrlKey === true || evt.metaKey === true)) ||
      (evt.keyCode >= 35 && evt.keyCode <= 39);
  }

  private isVowelKey(evt: KeyboardEvent) {
    return !evt.ctrlKey && !evt.metaKey &&
      (evt.keyCode === 65 || evt.keyCode === 69 || evt.keyCode === 73 || evt.keyCode === 79 || evt.keyCode === 85);
  }

  private parseValue(str: string): void {
    this.parsedValue = (str || '').replace(/[^AEIOU]/ig, '');
  }
}
```

The `test-vowel-input` component limits the input values to just vowels, and it follows the standard `input` API conventions. That is, it takes a `value` property for input, and it emits `input` events as the user interacts with it.

The HTML required to wire this up so we use `ngModel` looks like this:

```html
<test-vowel-input ngDefaultControl [(ngModel)]="info"></test-vowel-input>
```

The `ngDefaultControl` directive tells Angular to use the [Default Value Accessor](https://angular.io/api/forms/DefaultValueAccessor). The custom element can now be used just like any other element in the form.

#### Creating a Custom Control Value Accessor

There may be cases where the API conventions established for inputs do not make sense or cases where you are interfacing to a component someone else made and they did not follow those conventions. In those cases, you can create your own custom control value accessor.

Here is a custom element that allows the user to pick from the list of names that all end in "illy". It could easily follow the `input` API conventions, but it does not. Instead, it uses the `illy` to input the state and the `illyChange` event to signal user interaction. It is a rather silly component, but it will do for the illustration.

```tsx
import { Component, Event, EventEmitter, Prop, PropWillChange, State } from '@stencil/core';

@Component({
  tag: 'test-illy-picker',
  styleUrl: 'illy-picker.scss'
})
export class IllyPicker {
  private illys = ['Billy', 'Dilly', 'Killy', 'Lilly', 'Nilly', 'Philly', 'Zilly'];

  @Prop() illy: string;
  @State() selectedIlly: string;

  @Event() illyChange: EventEmitter;

  @PropWillChange('illy')
  handleIllyChange(newIlly: string) {
    this.setIllyState(newIlly);
  }

  componentWillLoad() {
    this.setIllyState(this.illy);
  }

  private setIllyState(newIlly: string) {
    this.selectedIlly = newIlly;
    this.illyChange.emit(newIlly);
  }

  clicked(evt) {
    this.setIllyState(evt.target.textContent);
  }

  render() {
    return (
      <ul>
        {this.illys.map(
          i =>
            i === this.selectedIlly ? <li class="selected">{i}</li> : <li onClick={(event: UIEvent) => this.clicked(event)}>{i}</li>
        )}
      </ul>
    );
  }
}
```

We could wire this up to effectively have a two-way binding as follows:

```HTML
<div>
  <test-illy-picker [illy]="myIlly" (illyChange)="myIlly=$event.detail"></test-illy-picker>
</div>
<div>You picked: {{myIlly}}</div>
```

But we really want to use `ngModel` so we can get all of the Angular form related goodies that come with it. To do this, we will need to create a custom [Control Value Accessor](https://angular.io/api/forms/ControlValueAccessor). To do this, create a directive that implements the `ControlValueAccessor` interface.

```ts
import { Directive, ElementRef, HostListener, Renderer2 } from '@angular/core';
import { ControlValueAccessor, NG_VALUE_ACCESSOR } from '@angular/forms';

@Directive({
  selector: 'test-illy-picker',
  providers: [{ provide: NG_VALUE_ACCESSOR, useExisting: IllyValueAccessorDirective, multi: true }]
})
export class IllyValueAccessorDirective implements ControlValueAccessor {
  constructor(private element: ElementRef, private renderer: Renderer2) {
    this.onChange = () => {};
  }

  onChange: (value: string) => void;

  writeValue(value: string) {
    this.renderer.setProperty(this.element.nativeElement, 'illy', value);
  }

  @HostListener('illyChange', ['$event.detail'])
  _handleIllyChange(value: string) {
    this.onChange(value);
  }

  registerOnChange(fn: (value: string) => void) {
    this.onChange = value => {
      fn(value);
    };
  }

  registerOnTouched() {}
}
```

The key methods here are:

* `writeValue` - this method is called by the Angular Forms API to write the specified value to the custom element
* `registerOnChange` - this is called by the Angular Forms API to register a function to call when the user interacts with the custom element
* a `@HostListener` - this method listens for the proper event to be emitted and then calls the registered `onChange` callback

Notice how this code matches up to the way we wired up the custom element without using `ngModel`.

Also, notice the selector that was used. I chose to use the tag for my custom element, meaning that this Control Value Accessor only works for this specific custom element. The selector could be an array of custom element tags, or a more generic string like `illyValueAccessorDirective` if this is more generic API that will be used across several different custom elements, similar to how `ngDefaultControl` works. Use whatever makes sense.

Now that we have the Control Value Accessor in place and associated with our custom element tag we are able to use `ngModel` as such:

```html
<div>
  <test-illy-picker [(ngModel)]="myIllyModel"></test-illy-picker>
</div>
<div>You picked: {{myIllyModel}}</div>
```

### Sample Code

I have a couple of sample applications that show this code at work. Please feel free to fork these repositories and explore further.

* [Test Components](https://github.com/kensodemann/test-components) - the component collection
* [Test Components Usage](https://github.com/kensodemann/test-components-usage) - an Angular application that uses the components

Happy Coding!!
