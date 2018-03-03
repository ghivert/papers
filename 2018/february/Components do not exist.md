# Functional Programming: Components do not exist

Components. They're present everywhere today. JavaScript use them a lot. Object Oriented Programming do it as well. And they're right.

_This article comes from the French version on my blog [here](https://guillaumehivert.io/article/programmation-fonctionnelle--les-composants-nexistent-pas)._

_Vous pouvez retrouver [la version française de cet article sur mon blog](https://guillaumehivert.io/article/programmation-fonctionnelle--les-composants-nexistent-pas) pour tous les francophones !_

That's why developers naturally reuse the same vocabulary when they're switching from one programming language to another. The paradigm shift from object oriented to functional programming changes a lot of things in daily life. Being able to keep the same vocabulary minimizes the changing cost. However, it creates a problem: object vocabulary is not always suited in functional world. Switching from paradigm induces unfortunately to learn a new vocabulary.

Components are the perfect example. Because they're omnipresent in JavaScript and totally missing in elm, beginners are often perplex. Let me explain:

According to how frameworks (like React) defines components, they're predefined reusable blocks able to communicate with other components. That's those components that I'm talking about.

This definition is really clear on communication: components are communicating each other. And communication in computer science can only be done by sending messages from one component to another one. (It is also possible to share memory, but it's often dangerous, and completely impossible in pure functional programming. And it's not about communication, but rather sharing data.)

Let's take a concrete example. According to the previous definition, every process in multi-threading program is a component when they can communicate with each other. At a higher level, a service in micro-services architecture is also a component, especially when they're communicating through REST with each other.

A component is always defined by a specific characteristic: it communicates with other components (and most of the time have a internal state).

In a typical React Single Page Application not using reducers like Redux, "component" makes sense. Each part of the application has an internal state, a rendering function, and a communication "interface". It can both have a particular behavior and communicates with every other components around. When talking about objects in general, "component" are really perfectly suited, because object oriented programming was created with internal state and message passing in mind.

Functional programming is about functions and data structures. Those functions and data structures must be uncoupled as much as possible. Data structures alone are not communicating. There is no communication, no message passing to the outside world. Data exist, they are structured, and they can be used. But they never are communicating each other. The main characteristic of a component is not present: the data structure is not a component. It's a bunch of data — structured or not — which can be used with functions.

Because an example is better than a long speech, keep going with a web application, built with elm. In elm, all data remains in a huge record: the model. Each event is able to modify the state of the application. A new model is generated in function of the old model and the received event. From this new model, it's possible to generate a new view for the application. There is no communication or message passing in this process. Actually, the only message comes from the outside world. From ports, and everything else. According to the previous definition, the application itself is a huge component able to communicate with the rest of the JavaScript and the web page.

But, in React, even though the entire application is a component, are some internal components hiding in elm? Absolutely not. An elm application (and more generally a pure functional application) do not involve any component. This creates too much confusion for beginners functional programmers. They try to mimic the same behavior in elm than in React, and they distort the language they use.

Let's keep going with an elm application. During view generation (i.e. HTML generation), we often separate data according to the way we are visually using it. If a date picker is in the toolbar, we generally put the date data inside the toolbar structure in the model. Now, what if this data is also useful in the footer of the app? Beginners often want to reuse the same date. And they are right. However as the data exist only in the toolbar, they imagine that they must generate a message to transmit the data the to footer, with the date from the toolbar. It is the exact case where the footer is seen as an independent component from the toolbar. But, as we already saw, components do not exist in elm. Why such a problem?

Let's take a simpler example. (Of course, the example is trivial, but the inter-component communication problem is present.)

```elm
type alias Model =
  { toolbar :
    { date : Date
    , color : Color
    }
  , footer :
    { date : Date }
  }
```

Let's imagine a view function, simple too, but representative of the problem:

```elm
view : Model -> Html msg
view { toolbar, footer } =
  Html.div []
    [ toolbarView toolbar
    , footerView footer
    ]

toolbarView : { date : Date, color : Color } -> Html msg
toolbarView { date, color } =
  Html.div []
    [ datepicker date ]

footerView : { date : Date } -> Html msg
footerView { date } =
  Html.div []
    [ dateView date ]
```

We can immediately saw the problem. The date here is used twice, and we can imagine an update function updating the toolbar. How to send an information to the footer with the new date?

What's important here, contrary to React components having their own state, both date reside in the same record. It is totally possible to rewrite differently this view function:

```elm
view : Model -> Html msg
view { toolbar, footer } =
  Html.div []
    [ Html.div []
      [ datepicker toolbar.date ]
    , Html.div []
      [ dateView footer.date ]
    ]
```

No magic here, only the `toolbarView` and `footerView` functions have been replaced by their content (which is always possible with pure functions). One detail is now evident: both date are easily directly accessible. Why not rewrite everything and remove redundancy?

```elm
view : Model -> Html msg
view { toolbar } =
  Html.div []
    [ Html.div []
      [ datepicker toolbar.date ]
    , Html.div []
      [ dateView toolbar.date ]
    ]
```

Suddenly, the problem disappears! The date in the toolbar, previously out of reach in `toolbarView`, becomes directly accessible. The communication problem disappears in the same time. The real question becomes "How to procede to get my structures of toolbar and footer?". This question is not anymore about the language, but about design. By merging the two structures of toolbar and footer, the uselessness of the footer record is uncovered; when such a structure would have been necessary in a components-based model. It's time to rewrite the model.

```elm
type alias Model =
  { date : Date
  , color : Color
  }
```

With both models of toolbar and footer disappeared, the structure is more compact, and the code much simpler:

```elm
view : Model -> Html msg
view { date, color } =
  Html.div []
    [ toolbarView date color
    , footerView date
    ]

toolbarView : Date -> Color -> Html msg
toolbarView date color =
  Html.div []
    [ datepicker date ]

footerView : Date -> Html msg
footerView date =
  Html.div []
    [ dateView date ]
```

And voilà! The code is simpler, the need for communication gone!

In general, the "need for communication between components" is a symptom of a broader comprehension problem. A symptom of a design problem. When wanting to reduce the scope of each variable, we end up wanting to make communicating parts of code which don't need it. Then, it is important to identify the common values, and merging them. We must redesign the model, think again about the data. The above code is the perfect example. In this example, it seems really easy, but think about a model with 4 levels of nesting, with 6 fields each times. When hard developing, it's easy to miss what is evident in a program with few lines of code.

```elm
-- Yes, I'm talking about a model like this.
type alias Model =
  { home : Home
  , articles : Articles
  }

type alias Articles =
  { user : User
  , content : List Article
  }

type alias Article =
  { title : String
  , text : Content
  , userId : Int
  }

type alias Content =
  { images : List String
  , texts : List String
  }

type alias User =
  { id : Int
  , name : Maybe String
  }  
```

In the above model, we can note that design missed brainstorming. Indeed, an Article always belongs to an user. Then, why does it have a `userId`? Why articles are not directly in the `User` structure? This does not seem grave in a first time. But what will happen when adding features? Is a `searchUserById` necessary each time we get an article? Will we have to add `userId` for everything belonging to a user? This can quickly lead to communication or data transmission problem (which lead to message creation in elm)…

To conclude, I would like to insist that a view is only one way to display the data from a model. There could be other views for the same model. The view is deriving from the data, and not the opposite. That means that in an elm application, the model must always be designed before the view, because you can easily do what you want with the model. Depending on the view used for the render, it is only a particular vision, always truncated from the model. On a user profile edit page, we don't care about the last trends on Medium. We are focusing on editing our profile. When browsing the latest trends on Medium, we don't care about what our profile looks like. But all those data, from user information to latest trends articles are present in the browser, in RAM. It is only a choice, to select only a part of the model, because the view allows it. Using only a part of the model allows the user to focus on what matters for him. It's not because data are organized in a particular way in our application that the view should mirror them. The view is uncoupled form the data as much as possible.

The usual vocabulary as well as the concepts are often different in functional programming than in object oriented programming. Learn a new vocabulary and getting accustomed to new habits is required. But you should not be afraid! Just like you learn a new language, like japanese, it can be hard at first, but once used to it, it becomes pleasing to use, and we evolve really quick! So my only advice would be: forget the component et jump into elm!

This article came to me after my discussions at the [elm meetup in Paris](https://www.meetup.com/fr-FR/Meetup-Elm-Paris) (in which you are obviously welcome!). Many beginners were talking continually about components and tried to send messages between components, like in React. But all of this didn't make sense. Few days after, I'm finally able to formulate what bothered me, in hope to reassure everyone looking to jump in functional programming.