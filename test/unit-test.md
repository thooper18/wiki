# Unit Test

## When the Coverage seems fucked up

### Remove SourceMaps

The option `--sourcemaps=false` or `-sm=false` can help the debug when an error occur.  
Nevertheless it can be responsible for weird coverage report (which make it innacurate).

Remove this option from the `ng test` command.

## Error: 1 timer(s) still in the queue.

To fix it, you must use the fakeAsync syntax with a tick and wait for stable fixture.

```javascript
it('should do ...', fakeAsync(() => {

  // Do some stuff here
    
  tick(500);
  fixture.whenStable().then(() => {
  
    // Add your expectation here
      
  });
}));
```

## Testing private methods

No, you can't.

## Spying private methods call

Use the `spyOn<any>` syntax and save the Spy in a variable.  
The expectation should be directly the Spy.

```javascript
it('should ...', () => {
  const spy: Spy = spyOn<any>(component, 'privateMethod');
  component.ngOnInit();
  expect(spy).toHaveBeenCalled();
});
```

## How to test an event in a directive

Check out this example where we want to `preventDefault()` on the event.

```javascript
@HostListener('click', ['$event'])
public onClick(event: any): void {
  event.preventDefault();
}
```

To test it, you must create your own directive and event objects.  
Then spy on the event's `preventDefault()` method.  
Simulate the HostListener and finally expect the call.

```javascript
it('should prevent default on event', () => {
  const directive: ExampleDirective = new ExampleDirective();
  const clickEvent: MouseEvent = new MouseEvent('click', null);
  spyOn(clickEvent, 'preventDefault');
  fixture.detectChanges();
  directive.onClick(clickEvent);
  expect(clickEvent.preventDefault).toHaveBeenCalled();
}
```

## How to test .emit()

In this example, we will test that the @Output `outputOnClick` emit correctly when `onClick()` is called.

```javascript
it('should emit', () => {
  let hasSubscribe: boolean = false;
  component.outputOnClick.subscribe(() => hasSubscribe = true);
  component.onClick();
  expect(hasSubscribe).toBe(true);
});
```

## How to test Observable data

Store the observable in a const and then subscribe to it to eval the callback when the next change.

```javascript
it('should equal...', fakeAsync(() => {
  const observable: Observable<number> = numberService.getNumber();
  observable.subscribe((number: number) => expect(number).toBe(1));
}));
```

## How to test a function which is called multiple times with params

Save the function as a spy and use the calls args to get all the data in one shot.

```javascript
it('should call the fn with 1 and then 2', () => {
  const fnSpy: Spy = spyOn(service, 'fn');
  expect(fnSpy.calls.allArgs()).toEqual([[1], [2]]);
});
```

## How to test .toHaveBeenCalledWith() when the param is a function

Instead of passing a real function, or even a noop, you can pass a jasmine type.  
The solution is then `jasmine.any(Function)`.

```javascript
it('should call login() with a callback', () => {
  spyOn(loginService, 'login');
  expect(loginService.login).toHaveBeenCalledWith(jasmine.any(Function));
});
```

## How to test the call of an internal method in a service's constructor

Use the prototype of the service.

```javascript
it('constructor() should call init()', () => {
  spyOn(AdmFwkKeyboardService.prototype, 'init');
  const newService: AdmFwkKeyboardService = new AdmFwkKeyboardService(logService);
  expect(AdmFwkKeyboardService.prototype.init).toHaveBeenCalled();
});
```
