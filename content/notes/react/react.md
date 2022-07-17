---
title: ReactJS
summary: "notes for React"
url: "/notes/react/"
date: 2022-05-19T19:12:44-04:00
showtoc: true
draft: false
ShowRssButtonInSectionTermList: true
---


# Functional Components:
A React functional component is a simple JavaScript function that accepts props and returns a React element. It has become standard practice to use Functional Components - vs Class Based Components.

Can create a Functional Component 2 ways:
* Declaring normal named functions 
``` javascript
function Component() {
    return (...{JSX CODE}...)
}
```
* Using an arrow function
``` javascript
const Component = () => {
    return {...{JSX CODE}...}
}
```
What is cool about JSX code is that it looks like HTML but you can actually insert Javascript code. For example, lets say you wanted your website
to greet the user by name. If the user didn't input their name, however, then you just want to call them `User`.

``` java
const App = () => {
    const name = "jchun"
    return (
        <div className="App">
            <h1> Welcome to the website { name ? name : "User" }! </h1>
        </div>
    )
}
```
As you can see, the functional component `App` is an arrow function. The component returns a react element that contains JSX code. In the JSX code, I have a ternary statement written in Javascript wrapped in {} where if the name exists, then it will print out "Welcome to the website {name}", but if name is null then it will print out "Welcome to the website User"!

When the variable `name` has a value:
![jchun](/notes/react/pictures/welcome.png)

When the variable `name` doesn't have a value (e.g `const name = null;`)
![jchun](/notes/react/pictures/nonwelcome.png)

As you can see, this allows you to dynamically change the website depending on certain variables or states.

* **Note** if you want to render adjacent html into your javascript you need "react fragments" which are just <> / </>

## Using Multiple Functional Components & Props

You can create multiple components and then put them in your larger component. Let's say we are displaying the home page with the `App` component from earlier. We can then create a `Person` component and then use it in our `App` component.

``` java {hl_lines = [15]}
const Person = (props) => {
    return (
        <>
            <h1>First Name: { props.firstName }</h1>
            <h2>Last Name:  { props.lastName }</h2>
            <h2>Age:        { props.age }</h2>
        </>
    );
};


const App = () => {
    return (
        <div className="App">
            <Person firstName = "Josh" lastName = "Chun" age = "20" />
        </div>
    );
}; 
```
I simply used the Person component I made by writing `<Person />` in my JSX code. I then used something called _props_ to send data to the Person component from the App component. I specified the `firstName`, `lastName`, and `age`. If you then look at my Person component, I used the props by calling `{ props.firstName }`, etc... You can see how components are super useful because lets say I wanted to copy and paste that 4 times, instead of just copy and pasting HTML, 
``` java
<>
    <h1>First Name: { firstName }</h1>
    <h2>Last Name:  { lastName }</h2>
    <h2>Age:        { age }</h2>
</>
<>
    <h1>First Name: { firstName }</h1>
    <h2>Last Name:  { lastName }</h2>
    <h2>Age:        { age }</h2>
</>
```
etc...

I can just do:
``` java
<Person firstName = ".." lastName = ".." age = ".." />
<Person firstName = ".." lastName = ".." age = ".." />
<Person firstName = ".." lastName = ".." age = ".." />
<Person firstName = ".." lastName = ".." age = ".." />
```
# States

States are plain javascript objects used by React to represent a piece of information about the component's current state. It is completely managed by the component itself. For example, if you want to have a counter button on your website, you need your button to hold its current value, or _state_.

Lets say we have a simple counter on our website with 2 buttons to increment and decrement the counter.
``` java
<button> - <button/>
<h1> 0 </h1>
<button> + <button/>
```
Right now, the buttons won't actually do anything and the counter will always display 0 because we are not managing the components **state**.

To use a state, we simply take advantage of Javascripts **array destructuring** and the **useState()** function.
Any time you use a function in React and it starts with "Use", those are called **Hooks**.

``` java {hl_lines=[2]}
const App = () => {
    const [counter, setCounter] = useState(0);

    return (
        <div className="App">
            <button> - <button/>
            <h1> 0 </h1>
            <button> + <button/>
        </div>
    );
}
```
In the first argument of the array, you use the variable that you want to change `Counter`. In the second argument, you use the function that you're going to use to change that variable `setCounter`; it is standard practice to name the function "set" + variable. The `0` being passed to `useState()`, is the initial value we want for `Counter`.

## Events

Events are actions that are triggered by some type of user action or system generated event (mouse click, button press, etc...) This is how we will actually change the counter when the user clicks the + button or - button.

``` java {hl_lines = [5, 7]}
    const [counter, setCounter] = useState(0);

    return (
        <div className="App">
            <button onClick={() => setCounter(prevCount => prevCount - 1)}> - <button/>
            <h1> {counter} </h1>
            <button onClick={() => setCounter(prevCount => prevCount + 1)}> - <button/>> + <button/>
        </div>
    );
}
```

As you can see from the highlighted lines, we setup these events by implementing an `onClick` callback function. A callback function is just an anonymous function (like lambda in python), that will be triggered on an event. To actually change the **state** of the counter, we use th e anonymous arrow function `() => setCounter()` to call the state-changing function we defined earlier in the code. Inside of the `setCounter()` function, we create another callback function that increments/decrements the previous count by one. 

![event](/notes/react/pictures/event.png)

* **Note**: The #1 rule is **never** manually change the state variable (e.g. counter). It is no longer a _normal_ variable, but it is now apart of the state and your application will break. They can only be changed by their own setter function (setCounter)

## useEffect()

Like `useState()`, `useEffect()` is a React Hook. useEffect allows functional components to have **life cycle methods**. For example if we have,
``` java
    useEffect(() => {
        setCounter(100)
    }, [])
```
this will set the counter to 100 every time the page reloads. Note we use setCounter function because we cannot manually change counter ourselves (e.g. counter = 100). The second argument is the **dependencies array**, where you specify on what event the hook will run. As you can see, the array is empty so it will default set the counter to 100 every time the the component is reloaded (when page is refreshed). If you put `counter` in the array, however, then the counter will be set to 100 every time you manually try to change the counter.

# Async + Await (Promise Functions)

I still need to delve deeper into promise functions and asynchronous code, but they are very useful in React (and vanilla JS in general). Async and await are basically just syntactic sugar for promise functions. We use async and await when we want a function to be **non-blocking**. As in, if we call a function that might take a while (retrieving data from online), we don't want to hold up our entire program. Specifically, when we use `fetch` and `.json()` in JS, those are **promise** functions by themselves and we need to use **async** and **await**.

``` javascript
const getData = async () => {
    const response = await fetch('some/website/file.json');
    const data     = await response.json();
};

getData();
```
Using this function, you can additonally add a **state**, so that you can render up the JSON data to your website. For example, if you write
``` jsx {hl_lines = [2, 7]}
const App = () => {
    const [rawData, setRawData] = useState({Title: "Default", Runtime : "--"})

    const getData = async() => {
        const response = await fetch("http://www.omdbapi.com/?i=tt3896198&apikey=37f30b34");
        const data = await response.json();
        setRawData(data)
    };

    useEffect(() => {
        getData();
    }, [])

    return (
        <div className="App">
            <PrintData data={rawData} />
        </div>
    );
};
```
You can set `setRawData` as the function to change your state and then call it in the `getData` function. In the example above I used `useEffect` to fetch some JSON data from an API and then use it to change the state of `rawData`. I then send `rawData` to the functional component `PrintData`, which looks like:
``` jsx
const PrintData = ({ data }) => {
    return (
        <>
            <h1> Hello { data.Title } </h1>
            <p> { data.Runtime } </p>
        </>
    );
};
```


