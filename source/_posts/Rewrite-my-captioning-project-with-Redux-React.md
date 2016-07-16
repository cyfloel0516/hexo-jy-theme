title: Rewrite my captioning project with Redux-React
date: 2016-07-15 22:18:15

categories:
- Development

tags:
- JavaScript
- React
- Redux
---

In my current project, we are developing a captioning web application which allows instructors and students to manage lecture videos, correct the captions and also communicate with others by writting comments or make bookmarks. First we used jQuery and the template library Mustache. They worked very well at first because the only UI component who needs the captions data is the main captions editor(see the figure below). So we can simply use jQuery to manipulate the UI component when data changes. 
<!-- more -->
![](main-editor.png)

But later on, we believed we need a captions global view in the left side(see the figure below). It is a very simple function but think about that: when user edits the captions from the main editor, we need to change the captions in the global view. Conversely, when user clicks a sentence in the global view, we need to change the current sentence in the main editor. 
![](left-panel.png)

Actually, we can still handle this because there are only two UI components we need to care about. But what if there are more components in future? There may be 10 UI components in where the captions may be changed and in consequence we need to update 10 UI components when the captions data changes. It will be definitely a disater for maintenace and that lead me to make a decision that to rewrite the application with React and Redux.

For most of my previous projects, jQuery was always the standard way to manipulate the DOM. In jQuery approach, we manipulate the DOM when the data changes. This approach was good in a long time when the website is not that complicated, however, with the increased complexity of web application and the increased popularity of SPA, development and maintenance becomes more and more difficult. 

In my opinion, the reason is that we cannot figure out the state of the UI because the UI can be changed from everywhere. The result is that we have to be change many places of code when the UI component behavior is changed. Everything is good if the application is small and simple but it will become a disaster when you have 10 or even 100 places in where you need to manipulate the UI. 

There are many frameworks such like Backbone and Angular are born to solve this problem. All these frameworks allow you to define the UI component based on the data and the UI will be updated automatically when you change the data. That benefit is that you can define the relation between data and UI component and the framework will do the UI update job for you, therefore, you don't need to write DOM manipulation code anymore.

So why I choose React and React as the solution instead of Backbone or Angular?

## Why React

There are already a lot of good solutions for the UI layer of web application: template solution such like Mustache and Jade, data binding solution such like Vue, Backbone and Angular. However, React comes with some great features but only focus on the UI layer. There are three features which I really like:

1. One of the most important features of React is the virtual DOM. The idea is to do the DOM comparison first on the JavaScript virtual DOM object and then decide if an update to DOM is necessary. As we all known, DOM manipulate is really slow so the a comparison operation on JavaScript can save us a lot of time from unnecessary DOM manipulation. Moreover, you can write your own function to decide if an update is necessary. 

2. Another important feature of React is it allow us to write modular UI component. You can write, share or reuse the component. In the past, everything become messy if the UI component is complicated but with React modular component, you can divide your big component into many small components. This feature makes development and maintenance much more easier than before.

3. The last one is the one-way data binding. Many people like two-way binding but in my opinion, one-way binding is more explicitly and also provides better performance than two-way binding. In most cases, two-way binding is unnecessary and can make your application slow and difficult to maintain.

In my project without React, I used Mustache to render data to HTML. For example, I have got a captions array and want to render the editor UI component with this array. The code should look like this(I omit the template code because it is not important):
```js
var $caption = $(Mustache.render($('#caption-template').html(), caption));
$(".caption-blocks").append($caption);
```
Then, if the captions data is changed, then we will need to invoke the above codes again to re-render the UI. The problem is that if we have too many UI componets then I will have many lines of UI update codes in one place and obviously it will make further maintenance difficult. 

Now let's define the UI component with React:
```js
var Captions = React.createClass({
    captionBlocks: function () {
        // Make UI component from captions array
        var blocks = [];
        for (var key in this.props.captions) {
            var capion = this.props.captions[key];
            var captionComponent = <CaptionBox key={key}  caption={capion} {...this.props}/>;
            blocks.push(captionComponent);
        }
        return blocks;
    },
    render: function () {
        return (
            <div id="caption-container">
                {this.captionBlocks() }
            </div>
        );
    }
});
```

The good thing is that the component only focus on presenting the data. But notice that we still need to pass the captions data to this component. If without Redux then we have to manually pass the data by calling `setState` function of this component to update it. It is not that different with the jQuery re-render approach because we need to do the UI update job by ourselves anyway. But after introducing the Redux I believe you will like this solution because everything become so clear and easy.

## Why Redux

As mentioned earlier, the biggest problem in my project is there are many places in where we may change the data and also there are many UI component which we need update. So how can we manage our data efficiently and update the UI when the data changes? Like Redux's offical site said, Redux is here to manage the data(or state) of your application. You can check the motivation of Redux [here](http://redux.js.org/docs/introduction/Motivation.html), but basically the following reasons are why I think Redux is great for building complicated web application and solve our problem:

1. Redux uses single data store makes it easy to build a complicated application because all data is in one place, you can easily check the current state of you application
2. The unidirectional data flow makes the logic more predictable and easier to understand. All data changes comes from action and ends in reducer so you can track or simulate the behavior of you application easily
3. Redux encourage immutable state makes test and debug much simpler

In the previous section of "why React", I rewrite the jQuery render code with React component. But we still need to manually invoke `setState` to update React component. Is there a way which allows us to focus on the data and let the library do the rest of work such like update the UI component? 

The anwser is definitely yes. The basic idea is to define the data store with Redux and define the relation between data store and React component. After defining the relation, the UI component will be updated whenever the relative data is changed. Currently the best solution is to use [React-Redux](https://github.com/reactjs/react-redux). This library provides us a tool to connect the Redux data store to the React component and it will update the React component when the data store is changed. So the steps is:

1. Define the data center store, action and also the reducer with Redux. You can check the example [here](http://redux.js.org/docs/introduction/Examples.html) 
2. Connect React component to Redux data store with React-Redux. You just need two functions to do this: 
    * `mapStateToProps`: map Redux state to the property of React component
    * `mapDispatchToProps`: enable React to dispatch action to Redux
    You can check the examples [here](http://redux.js.org/docs/basics/UsageWithReact.html)

So the code directory structure should look like this:
```
|- actions      Redux action directory
|- components   React UI components directory
|- containers   React-Redux connector directory
|- reducers     Redux reducer directory
|- app.js       Create the Redux data store, load containers and render it
```

## Summary
Now what is the difference with our old jQuery solution? 

First, UI component only needs to know how to present the data, and with the modular component of React, we can maintain the UI layer easily. 

Second, we have a data center to store all the application data in one place. The only way to update application state is to dispatch Redux action.

Last, we connect Redux data store to our React component based on our demand. We can decide which component needs which part of the data in Redux data store.

So the magic thing is that we update our application state by dispatching action to Redux, Redux update the data store and then React-Redux notify the corresponding React component that there are some data changed. At last, React perform comparison on virtual DOM to decide whether to update the real DOM. As you see, the data flow is quite clear and intuitive and that is why I like this solution, it make the development and maintenance of complicated web application much simpler.



