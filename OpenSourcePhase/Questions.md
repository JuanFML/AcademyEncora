### Question 1
## How does it generate a map-reduce query for a simple SELECT COUNT(*) FROM customer?

What is Hadoop?
Hadoop helps in parallel computing, first it retrieves the data and divides into clusters, then it works with the data locally as it has been already retrieved. 
It creates a Hadoop Distributed File System (HDFS)

What is MapReduce?
MapReduce is a programming paradigm that helps on the processing of data. The advantage is that it takes a file which is divided into chunks with the help of Hadoop. 
Then it processes this divided data in parallel. MapReduce maps all the data into key: value pairs (mappers), then the reducer takes this output and combines 
all the data with the same key and process them as needed. I watch [this video](https://www.youtube.com/watch?v=KzMXU9A3v4I) to understand MapReduce.

I will answer de question based on what I’ve read and what I practiced using an online editor for Hive which you can access it through this [link](https://demo.gethue.com/hue/editor?editor=58536). 
In Hive for a simple `SELECT * FROM table` won’t use a MapReduce (MR), this is because it is programmed to function in the most efficient possible way, 
so for this mentioned query it would be slower to use a MR. In contrast whenever an aggregate function is requested a MR comes in helpful and it makes it faster. 

Hive has an [`EXPLAIN`](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Explain) command which lets you see the “behind the scenes” process of a query, so if you use it will show you the MapReduce jobs. In Hive if a simple 
`SELECT COUNT(*) FROM table`  it won’t use a MR I confirmed this using the online editor and the EXPLAIN command, although I use a small table, but this true for newer 
versions of Hive. It doesn’t uses a MR because Hive doesn’t need it for simple queries such as this one, it is faster to make a direct fetch operation from HDFS than doing a MR, 
because a simple COUNT(*) doesn’t have a WHERE or GROUP BY, so no filtering or summarizing is done.

In contrast if you add a condition, for example `SELECT COUNT(*) FROM buyer GROUP BY buyer.id` (in this case the id repeats), it will kick in a MR job, 
once again I confirmed this with the EXPLAIN command. Now what happens in this scenario, is that you are asking for the count of buyers by the id, first the map job will 
group the data into key: value pairs, where the key would be the buyer.id, and the value would be one (because it is counting), then the reduce job will take all these pairs and 
combine the ones that have the same key, for example, you have a buyer with an id of 3, which appeared 
2 times in the table it would look like this in the map job {1:1},{2:1},{3:1}, {3:1}, where you have buyers with id 1 and 2, 
then the reduce job will know it has two equal keys and then sum the values which would end up with {1:1},{2:1},{3:2}.

### Question 2
## How does Async and Defer attributes work with JavaScript?
JavaScript runs normally in a synchronous way or single threaded, this means that it runs one command at a time in the order the code appears. 
This can be time consuming when things have to render, JS files tend to be heavier and takes longer to process, so whenever it appears in a HTML file it 
will halt the entire code until the JS file finishes.

For example, whenever you encounter a HTML file like this one:
```
<!DOCTYPE html>
<html>
  <head>
    <title>My test page</title>
    <script src="scripts/main.js"></script>
  </head>
  <body>
    ...
  </body>
</html>
```
It will run tag by tag until it is blocked by the `<script>` tag, then it will enter the JS file and it will download it and then execute it, after this it will finish with the 
remaining html tags, this can be very slow if it has to process a big JS file. One solution was to put the tag at the bottom of the file but is not a real solution
so this is when these two attributes come in. Async and defer help to download the JS file in an 
asynchronous way, which means it will run the html tags and at the same time it downloads the JS file, although it won't execute it asynchronously.

An `async` attribute would look like this in the tag `<script async src="scripts/main.js"></script>`. In this case it will tell the browser to download the JS file at the same 
time it keeps building the page, but whenever it finishes to download, it will stop everything and execute the JS file, then it will continue where the html file left off. 
The async attribute waits for no one, so it will run whenever it is ready, it can be before or after the DOMContentLoaded event, depending when it finished to download.

A `defer` attribute would just replace the async word in the tag and it will work similarly to the async attribute. With defer it will work the same as async up until 
it finishes downloading the JS file, then instead of executing it right away, the DOM will continue to build until it is ready and then 
it will execute the JS file, but before the DOMContentLoaded event. 
So in this case it runs continuously the html file and the end executes the JS file, although it is already downloaded. 
This attribute doesn't stop the rendering process, so it is preferred whenever you prioritize the rendering over the JS file. 

Whenever there are multiple JS files, the async attribute can execute them randomly without a defined order whatever downloads first, in contrast defer, at the end of the rendering it will execute each JS file in the order the script tags appeared.

Now what happens inside JavaScript whenever it encounters these two attributes.

### Question 3
## How do createContext and useContext hooks work in React?
To start with Context is used whenever you want to pass around data through your folders and files within your repository, without the need to pass the information through props from father to children and so on. To summarize it is a simple way of sharing data between files. This Data is generally used like global variables, for example in my experience I have used Context to save the session of a logged user, as well as the date chosen in another page, or you can also use it to store the preferred language of the page.

I am going to give examples in the case where a separate file is created for the context which is the mostly used case. First you have to use `createContext`, this will create a context object i.e. `const MyContext = createContext();` in here you are creating an object without an initial value, although it can be a string, int or an object which contains various values. Here is an example of a file using it:
```
import React, { createContext, Component } from "react";

export const MyContext = createContext();

export class ContextProvider extends Component {
  state = {
    loggedUser: null,
  };

  componentDidMount() {
    this.setState({ loggedUser: "Juan" });
  }

  render() {
    const { loggedUser } = this.state;
    return (
      <MyContext.Provider
        value={{
          loggedUser,
        }}
      >
        {this.props.children}
      </MyContext.Provider>
    );
  }
}
```
In here you are creating a context object which will be used in other files, every context comes with a Provider. Within the class component you are creating a variable called loggedUser this one is then passed as a prop value from the Provider, this means that everything inside it, will be able to read this value, inside the provider you are returning everything that is inside (children).

Then for you to be able to access anywhere your context, you have to wrap your tree of file for example in the index.js file of you React app, like the example shown below:
```
ReactDOM.render(
  <React.StrictMode>
    <ContextProvider>
      <App />
    </ContextProvider>
  </React.StrictMode>,
  document.getElementById("root")
);
```
Here you will have to import the class ContextProvider not MyContext because that is used for the values, then you wrap the App file within, this way you letting App pass (children) but with the value given in the context file.

Now for you to use the context value, you have to use the hook `useContext` this will allow you to use the value within a function or in the JSX. An example is shown below
```
import { MyContext } from "../../context/context";
...
const context = useContext(MyContext);
const loggedUser = context.loggedUser
...
```
In here you have to import the context object and then get the value you want in case there are more. The hook `useContext` can only accept a context object, then it will return the value given in the prop from the <MyContext.Provider> (in case there are more providers it gets the closest one in the tree), as well whenever the value changes it will update the value whenever `useContext` is used. 

There are multiple ways of using context, you could export it into only some specified files not to the entire App, you could have multiple providers, at the end you can manipulate context to your needs.
