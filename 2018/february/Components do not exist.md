# Functional Programming: Components do not exist

Components. They're present everywhere today. JavaScript use them a lot. Object Oriented Programming do it as well. And they're right.

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

Les deux modèles de toolbar et de footer envolés, la structure devient bien plus compacte, avec un code de vue plus simple :

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

Et voilà ! Le code est simplifié, le besoin de communication a disparu !

En règle général, le « besoin de communication entre composants » est symptomatique d'un problème de compréhension. Symptomatique d'un problème de modélisation. À force de vouloir réduire au maximum le scope de chaque variable, on se retrouve à vouloir faire communiquer entre eux des morceaux de code qui n'en ont pas besoin. Il est alors important d'identifier les valeurs qui nécessitent d'être mises en commun, et de les fusionner. Il faut retravailler son modèle, repenser le partage des données. Le code ci-dessus en est un parfait exemple. Dans cet exemple, cela parait évidemment très facile, mais pensez à un modèle avec 4 niveaux de profondeur, avec 6 champs à chaque fois. Lors d'une séance de développement intensive, il est facile de passer à côté de ce qui peut paraître évident dans un code de quelques lignes.

```elm
-- Oui, je parle bien d'un Model de ce type.
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

Dans le modèle ci-dessus, il est amusant de noter que la modélisation manque cruellement de réflexion. En effet, un Article appartient toujours à un utilisateur. Pourquoi comporte-t-il alors un `userId` ? Pourquoi les articles ne sont-ils pas dans la structure `User` directement ? Cela ne parait pas très grave dans un premier temps, mais que se passera-t-il lors de l'ajout de fonctionnalités ? Va-t-on devoir effectuer un `searchUserById` à chaque fois que l'on obtient un article ? Va-t-on devoir rajouter des `userId` pour tout ce qui appartient à l'utilisateur ? Cela peut rapidement amener à des problèmes de communication ou de transmission de données (et donc de création de messages en elm)…

Pour finir, j'aimerais insister sur le fait qu'une vue n'est qu'une façon parmi tant d'autres d'afficher les données organisées d'un modèle. La vue dérive des données, et non pas l'inverse. Ce qui signifie que dans une application elm, le modèle doit être pensé en priorité sur la vue, puisque celle-ci peut toujours s'accommoder du modèle — qui lui ne peut s'adapter à la vue. Selon la vue utilisée pour le rendu, il s'agit d'une vision particulière et souvent tronquée du modèle : dans le cas de l'édition de son profil, on se fiche royalement de connaître la liste des articles en tendance sur Medium. Lors de l'affichage des articles en tendance sur Medium, on se fiche royalement de ce que contient notre profil. Pourtant, toutes ces informations se trouvent en permanence dans la RAM de notre navigateur, dans notre programme elm. Il ne s'agit que d'un choix, de sélectionner une partie de notre modèle seulement, car notre vue nous le permet. Ne pas tout utiliser, pour ne pas noyer notre utilisateur sous les informations qui ne l'intéresse pas. Ce n'est pas parce que nos données sont organisées d'une manière particulière dans notre application que la vue doit s'organiser de la même manière. Celle-ci est découplée des données autant que possible.

Le vocabulaire habituel ainsi que les concepts sont souvent différents en programmation fonctionnelle par rapport à la programmation objet, et il faut donc réapprendre un vocabulaire particulier et prendre de nouvelles habitudes. Mais cela ne doit pas faire peur ! De la même manière que lorsque l'on apprend une langue étrangère, cela peut être difficile au premier abord, mais une fois que l'on se débrouille avec, on prend un plaisir immense et on évolue extrêmement vite ! Alors je n'ai qu'un conseil à vous donner : abandonnez vos composants, et passer au plus vite à elm !

Cet article m'est venu suite à mes discussions au dernier [meetup elm de Paris](https://www.meetup.com/fr-FR/Meetup-Elm-Paris) (auquel je vous convie bien évidemment !). De nombreux débutants me parlaient continuellement de composants et souhaitaient envoyer des messages entre composants, à l'instar de React. Mais cela n'avait pas de sens, et je n'arrivais pas à mettre des mots dessus. Quelques jours après, je suis enfin capable de formuler ce qui me gênait, dans l'espoir de rassurer ceux qui souhaiteraient faire le grand saut dans la programmation fonctionnelle.

_La traduction anglaise de ce texte se trouve également sur Medium. Longue vie à la langue française !_