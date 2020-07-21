---
id: high-level-design-fe
title: High Level Design
---

## Clean Code

### Variables

Give meaningful names to variables

### Functions

1. Function name should describe what it does.
2. A function should do only one thing.
3. Limit the amount of function parameters, preferably two. if you need more you can send the parameters inside an object.
4. If you find yourself writing the same code more than once, even if its a little bit different, try to generalize it in to a function.
5. Functions should be pure

### Classes

### React components

1. Make your React component as short as possible
2. Name Your Components Using Standard Naming Conventions
3. Reduce the number of props to the minimum
4. Put Independent Functions Outside of Your Custom Hooks
5. Split your components to container and presentational components
6. Folderize Your Components

### DRY

DRY is an acronym that stands for "Don’t Repeat Yourself."
If you find yourself writing the same code in different places, consider writing it only once and just reuse it, whether as a function or as a component.

### KISS

KISS is an acronym that stands for "Keep it simple stupid."

### SOLID

Single Responsibility Principle (SRP)
Open/Closed Principle (OCP)
Liskov Substitution Principle (LSP)
Interface Segregation Principle (ISP)
Dependency Inversion Principle (DIP)

### Testing

When you write your code, you should write it in a way it would be easy to test it. make your code predictable and testable.

### Error handling

### Comments

If you write clean code, it should be self commenting.

### Type script

## Keep it simple

TBD

## Where should the business logic reside?

TBD

## Separation of concerns

TBD

## Compose in Frontend

Composition takes a big place in React's architecture and design.
When building components in React, we usually build them using other components.
Generic base components will help us reuse and extend them to higher components without repeating ourselves.
Lets take a Button component for example:

```jsx
const Button = ({ onClick, className, children }) => {
  return (
    <button onClick={onClick} className={classnames('button', className)}>
      {children}
    </button>
  )
}
```

That's a nice solid base Button.  
What do we do when we need to extend it and also have `PrimaryButton` and `SecondaryButton`?
If we follow the composition pattern, we should do something like this:

```jsx
const PrimaryButton = ({ onClick, className, children }) => {
  return (
    <Button onClick={onClick} className={classnames('primary', className)}>
      I'm a Primary Button
    </Button>
  )
}
```

```jsx
const SecondaryButton = ({ onClick, className, children }) => {
  return (
    <Button onClick={onClick} className={classnames('secondary', className)}>
      I'm a Secondary Button
    </Button>
  )
}
```

A different way to do that is using props to encapsualte that logic within the button.  
That way, we don't really compose the base button but extend it, that's not ideal.

```jsx
const Button = ({ onClick, className, children, variant }) => {
  return (
    <button
      onClick={onClick}
      className={classnames('button', className, {
        primary: variant === 'primary',
        secondary: variant === 'secondary'
      })}
    >
      {children}
    </button>
  )
}
```

This approach can make our components too big and handle too much logic, that's why we preffer composing over extending.

## Functional components over Class components

TBD

## Get data from store only using selectors

TBD

## Top-Down / Undirectional Data Flow

React's data flow is a one-way data flow (AKA one-way binding), we can use either `props` or `state` to pass that data.  
Usually, the data should be gathered in a top component, and it will pass the data to which of it's children who needs it.  
Having said that, "Lifting state up" and "Co-locating state" are two important issues we need to understand.  
"Lifting state up" means that if we have two child components who consume the same state, this state should be kept on their common parent.  
"Co-locating state" means that if we have a child component consuming state that only it needs, it **shouldn't** be kept on the parent component.  
We need to remember that every state change on the parent will cause **all** of it's children to re-render (unless they bail out of the render process), that's why we should

## Typescript patterns

TBD

## Data Provider pattern

TBD
