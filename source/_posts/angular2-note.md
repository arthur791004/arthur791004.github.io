---
title: angular2-note
tags: angular2
---
# Say "Hello World" to Angular 2
Example

# Writing a Simple Angular 2 Component
```
import {Component} from 'angular2/core';

@Component({
    selector: 'my-app',
    template: '<h1>My First Angular App</h1>'
})
export class AppComponent { }
```

# Using events and refs
```
<input type="text" #myInput>
<button (click)="onClick(myInput.value)"></button>
```

# Events in Depth
```
<button (eventName)="eventHandler(arguments, ...)"></button>
```
* eventName: click, mouseover, ...
* eventHandler: function
* argument: $event, variable, ...

# Injecting a Service
* todo-service.ts
```
import { Injecttable } from 'angular2/core';

@Injectable()

export class TodoService {
  todos = [];
}
```

* man.ts
```
...
imrpot { TodoService } from './todo-service';
...
boostrap(App, [TodoService]);
```

* todo-input.ts
```
...
imrpot { TodoService } from './todo-service';
...
export class TodoInput {
  constructor(todoService: TodoService) {
    this.todoService = todoService;
  }

  onClick(value) {
    this.todoService.todos.push(value);
    console.log(this.todoService.todos);
  }
}
```

# Using the @Inject decorator
```
@Inject(TodoService)
```

# Using ng-for to repeat template elements
```
@Component({
    selector: 'todo-list',
    template: `
      <div>
        <ul>
          <li *ngFor="#todo if todoService.todos">
            {{ todo }}
          </li>
        </ul>
      </div>
    `
})

export class TodoList {
  constructor(public todoService: TodoService) {

  }
}
```

# Using ng-model for two-way binding
```
<input type="text" [(ngModel)]="todoModel">
{{ todoModel }}
```

# Adding a data model
```
export class TodoModel {
  status: string = "started";

  constructor(title: string = "") {
    this.title = title;
  }
}
```

# Template property syntax
```
<element [property]="prppertyValue"></element>
```
* property
  * ngClass, hidden, contentEditable, ...
* propertyValue
  * boolean, string, ...
* Example
```
<span [hidden]="'todo.status == 'completed'">{{ todo.title }}</span>
<button (click)="todo.toggle()">Toggle</button>
```

# Passing data to components with @Input

* todo-list.ts
```
<todo-item-renderer [todo]="todo"></todo-item-renderer>
```

* todo-item-renderer.ts
```
import { Component, Input } from 'angular2/core';

@Component({
  selector: 'todo-item-renderer',
  template: `
    <div>
      <span [hidden]="'todo.status == completed'">{{ todo.title }}</span>
      <button (click)="todo.toggle()">Toggle</button>
    </div>
  `
})

export class TodoItemRenderer {
  @Input() todo;
}
```

# ng-class and Encapsulated Component Styles
```
...

@Component({
  selector: 'todo-item-renderer',
  template: `
    <style>
      .completed {
        text-decoration: line-through;
      }
    </style>
    <div>
      <span [ngClass]="todo.status">{{ todo.title }}</span>
      <button (click)="todo.toggle()">Toggle</button>
    </div>
  `
})

...
```

# Controlling how Styles are Shared with View Encapsulation
* ViewEncapsulation
  * None, Native, Emulated

```
import { ViewEncapsulation } from 'angular2/core';

@Component({
  encapsulation: ViewEncapsulation.None,
  ...
})

...
```

# Using Pipes to Filter Data
* json, uppercase, ...

* search-pipe.ts
```
import { Pipe } from 'angular2/core';

@Pipe({
  name: 'search'
})

export class SearchPipe {
  transform(value) {
    return value.filter((item) => item.title.startsWith('s'));
  }
}
```

* todo-list.ts
```
@Component({
  pipes: [SearchPipe],
  ...
  template:`
    ... todos | search ...
  `
})

...
```

# Using Array ...spread to enforce Pipe immutability.
```
addTodo(todo: TodoModel) {
  this.todos = [...this.todos, todo];
}
```

# Refactoring mutations to enforce immutable data in Angular 2

* started-pipe.ts
```
import { Pipe } from 'angular2/core';

@Pipe({
  name: 'started'
})

export class SearchPipe {
  transform(value) {
    return value.filter((item) => 'started' === item.status);
  }
}
```

* todo-item-renderer.ts
```
import { Ouput, EventEmitter } from 'angular2/core';

...
  <button (click)="toggle.emit(todo)">Toggle</button>
...

export class TodoItemRenderer {
  @Input() todo;
  @Output() toggle = new EventEmitter();
}
```

* todo-list.ts
```
...
  (toggle)="todoService.toggleTodo($event)"
...
```

* todo-service.ts
```
...
  toggleTodo(todo: TodoModel) {
    todo.toggle();

    const i = this.todos.indexOf(todo);
    this.todos = [
      ...this.todos.slice(0, i),
      todo,
      ...this.todos.slice(i + 1)
    ];
  }
...
```

# Build a select dropdown with *ngFor in Angular 2
* started-pipe.ts
```
import { Pipe } from 'angular2/core';

@Pipe({
  name: 'started'
})

export class SearchPipe {
  transform(value, [status]) {
    return value.filter((item) => item.status === status);
  }
}
```

* todo-list.ts
```
...
  <li *ngFor="#todo of todoService.todos | started: status">
...

export class TodoList {
  @Input() status;
  ...
}

...
```

* main.ts
```
...
  <status-selector (select)="status = $event"><status-selector>
  <todo-list [status]="status'"></todo-list>
...
```

* status-selector.ts
```
import { Component, Output, EventEmitter } from 'angular2/core';

@Component({
  selector: 'status-selector',
  template: `
    <div>
      <select #sel (change)="select.emit(sel.value)>
        <option *ngFor="#status of statuses">
          {{ status }}
        </option>
      </select>
    </div>
  `
})

export class StatusSelector {
  @Output select = new EventEmitter;
  statuses = ['started', 'completed'];

  ngOnInit() {
    this.select.emit(this.statuses[0]);
  }
}
```

# Filter items with a custom search Pipe in Angular 2
* search-pipe.ts
```
import { Pipe } from 'angular2/core';

@Pipe({
  name: 'search'
})

export class SearchPipe {
  transform(value, [term]) {
    return value.filter((item) => item.title.startsWith(term));
  }
}
```

* todo-list.ts
```
...
  <li *ngFor="#todo of todoService.todos
    | started: status
    | search: term">
...
export class TodoList {
  @Input() status;
  @Input() term;
  ...
}

...
```

* search-box.ts
```
import { Component, Output, EventEmitter } from 'angular2/core';

@Component({
  selector: 'search-box',
  template: `
    <div>
      <input #input type="text" (input)="update.emit(input.value)">
    </div>
  `
})

export class SearchBox {
  @Output() update = new EventEmitter();

  ngOnInit() {
    this.update.emit('');
  }
}
```

* main.ts
```
...
  <search-box (update)="term = $event"></search-box>
  <todo-list
    [status]="status"
    [term]="term">
  </todo-list>
...
```

# Organizing Angular 2 projects by feature
* refactor folder structure...
* types:
  * components
  * pipes
  * services

# Overview of Angular 2 and what to learn next...
* Component
  * directive
  * Input
    * @Input() input <---> [input]="input"
  * Output
    * @Output() output <---> (output)="outputHandler($event)"
    * EventEmitter
  * ngFor
  * event
  * data binding
  * property
* Service
  * data
* Pipe
  * transform
  * EventEmitter

# Reference
* https://egghead.io/courses/angular-2-fundamentals
* http://learnangular2.com/outputs/
