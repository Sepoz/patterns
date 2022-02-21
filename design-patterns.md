## Singleton Pattern

Singletons are classes that can be instantiated once and can be accessed globally.

```javascript
let count = 0;

const counter = {
	increment() {
		return ++count;
	}

	decrement() {
		return --count;
	}
}

Object.freeze(counter);
export { counter };
```

The Object.freeze method make sure that consuming code cannot modify the Signleton.

## Proxy Pattern

With a proxy object we can get more control over the interations with certain objects.
Instead of interactiong with the target object directly, we will interact with the Proxy object.

Create a person object.

Instead of interacting with this object directly, we want to interact with a proxy object. In JavaScript, we can easily create a new proxy with by creating a new instance of Proxy.

```javascript
const person = {
	name: "john doe",
	age: 42,
	nationality: "italian",
};

const personProxy = new Proxy(person, {
	get: (obj, prop) => {
		console.log(`The value of ${prop} is ${obj[prop]}`);
	},
	set: (obj, prop, value) => {
		console.log(`Changed ${prop} from ${obj[prop]} to {value}`);
		obj[prop] = value;
	},
});

personProxy.name; // The value of name is john doe
personProxy.age = 43; // Changed age from 42 to 43
```

The second argument of Proxy is an object that represents the handler.
Although there are many methods that you can add to the Proxy handler, the two most common ones are get and set.

A proxy can be useful to add validation. A user shouldn't be able to change a person's age to a string value, or givr him an empty name. Or if the user is trying to access a property on the object that does not exist, we should let the user know.

```javascript
const personProxy = new Proxy(person, {
	get: (obj, prop) => {
		if (!obj[prop]) {
			console.log(
				`Hmm.. this property doesn't seem to exist on the target object`
			);
		} else {
			console.log(`The value of ${prop} is ${obj[prop]}`);
		}
	},
	set: (obj, prop, value) => {
		if (prop === "age" && typeof value !== "number") {
			console.log(`Sorry, you can only pass numeric values for age.`);
		} else if (prop === "name" && value.length < 2) {
			console.log(`You need to provide a valid name.`);
		} else {
			console.log(`Changed ${prop} from ${obj[prop]} to ${value}.`);
			obj[prop] = value;
		}
	},
});
```

## Provider Pattern

In some cases we want to make data available to to many components in an application, we can pass data to components using props but this can be difficult if almost all of the components need access to the value of the props.

```javascript
function App() {
  	const data = { ... }

  	return (
    	<div>
      	<SideBar data={data} />
      	<Content data={data} />
    	</div>
 	)
}

const SideBar = ({ data }) => <List data={data} />
const List = ({ data }) => <ListItem data={data} />
const ListItem = ({ data }) => <span>{data.listItem}</span>

const Content = ({ data }) => (
  	<div>
    	<Header data={data} />
    	<Block data={data} />
  	</div>
)
const Header = ({ data }) => <div>{data.title}</div>
const Block = ({ data }) => <Text data={data} />
const Text = ({ data }) => <h1>{data.text}</h1>
```

Passing props down this way can get quite messy.
Rather than passing that data down each layer through props, we can wrap all components in a Provider. A Provider is a higher order component provided to us by the a Context object. We can create a Context object, using the createContext method that React provides for us.
The Provider receives a value prop, which contains the data that we want to pass down. All components that are wrapped within this provider have access to the value of the value prop.

```javascript
const DataContext = React.createContext()

function App() {
  	const data = { ... }

  	return (
    	<div>
      		<DataContext.Provider value={data}>
        		<SideBar />
        		<Content />
      		</DataContext.Provider>
    	</div>
  	)
}
```

Each component can get access to the data, by using the useContext hook. This hook receives the context that data has a reference with, DataContext in this case. The useContext hook lets us read and write data to the context object.

```javascript
const DataContext = React.createContext();

function App() {
  	const data = { ... }

  	return (
    	<div>
      		<SideBar />
      		<Content />
    	</div>
  )
}

const SideBar = () => <List />
const List = () => <ListItem />
const Content = () => <div><Header /><Block /></div>


function ListItem() {
  	const { data } = React.useContext(DataContext);
  	return <span>{data.listItem}</span>;
}

function Text() {
  	const { data } = React.useContext(DataContext);
  	return <h1>{data.text}</h1>;
}

function Header() {
  	const { data } = React.useContext(DataContext);
  	return <div>{data.title}</div>;
}
```

We no longer have to worry about passing props down several levels through components that don't need the value of the props.
