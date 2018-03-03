# Functional Programming: Components do not exist

Components. They're present everywhere today. JavaScript use them a lot. Object Oriented Programming do it as well. And they're right.

That's why developers naturally reuse the same vocabulary when they're switching from one programming language to another. The paradigm shift from object oriented to functional programming changes a lot of things in daily life. Being able to keep the same vocabulary minimizes the changing cost. However, it creates a problem: object vocabulary is not always suited in functional world. Switching from paradigm induces unfortunately to learn a new vocabulary.

Components are the perfect example. Because they're omnipresent in JavaScript and totally missing in elm, beginners are often perplex. Let me explain:

According to how frameworks (like React) defines components, they're predefined reusable blocks able to communicate with other components. That's those components that I'm talking about.

This definition is really clear on communication: components are communicating each other. And communication in computer science can only be done by sending messages from one component to another one. (It is also possible to share memory, but it's often dangerous, and completely impossible in pure functional programming. And it's not about communication, but rather sharing data.)

Let's take a concrete example. According to the previous definition, every process in multi-threading program is a component when they can communicate with each other. At a higher level, a service in micro-services architecture is also a component, especially when they're communicating through REST with each other.

A component is always defined by a specific characteristic: it communicates with other components (and most of the time have a internal state).

In a typical React Single Page Application not using reducers like Redux, "component" makes sense. Each part of the application has an internal state, a rendering function, and a communication "interface". It can both have a particular behavior and communicates with every other components around. When talking about objects in general, "component" are really perfectly suited, because object oriented programming was created with internal state and message passing in mind.

Functional programming is about functions and data structures. Those data structures must be uncoupled as much as possible. Les structures de données seules ne communiquent pas vers l'extérieur. Il n'existe aucune communication, aucun passage de données avec le monde extérieur. Les données existent, sont structurées, et peuvent être utilisées, mais elle ne communiquent pas avec le monde extérieur. La caractéristique principale du composant n'est pas remplie : la structure de données n'est donc pas un composant. Il s'agit d'un paquet de données — structuré ou non — manipulable à l'aide de fonctions.

Mais puisqu'un exemple vaut mieux qu'un long discours, continuons dans le cas d'une application web, cette fois-ci construite à l'aide d'elm. En elm, l'ensemble des données de l'application réside dans une grosse structure de données (un record) : le modèle. Chaque évènement peut modifier l'état de l'application. Un nouveau modèle est donc généré en fonction de l'ancien modèle et de l'évènement reçu. De ce nouveau modèle, il est possible de déduire un nouvel affichage pour l'application. Aucune étape de ce parcours ne comporte de communication et de passage de message. En réalité, le seul passage de message existe entre l'application elle-même et le monde extérieur. L'application elle-même serait-elle un énorme composant ? *Selon la définition ci-dessus, oui.* L'instanciation d'une application elm donne accès à des ports permettant de communiquer avec le reste de l'application JavaScript.

Mais, si l'application entière est un composant, des composants internes se cachent-ils dans l'application ? Absolument pas. Une application elm (et plus généralement une application fonctionnelle pure) ne comporte aucun composant. Cela crée une confusion trop régulière auprès des débutants de la programmation fonctionnelle. En voulant à tout prix retrouver le même comportement qu'avec des composants, ils déforment le langage avec lequel ils sont en train de coder.

Continuons avec le cas d'une application elm. Lors de la génération de la vue (le HTML de l'application), il est courant de séparer les données selon la façon dont on les utilise visuellement. Si un sélecteur de date se trouve dans la toolbar, on aura tendance à placer la donnée de date dans la structure de la toolbar au sein de notre modèle. Maintenant, si cette même date est utile également dans le footer de l'application, les débutants ont tendance à vouloir réutiliser la date de la toolbar. En soit, cela est une bonne chose. Toutefois, puisque cette donnée n'existe que dans la toolbar, ils s'imaginent devoir générer un message spécifique adressé au footer au sein de elm avec la date indiquée dans la toolbar. Il s'agit exactement du cas où le footer est vu comme un composant indépendant de la toolbar. Toutefois, comme nous l'avons vu plus haut, les composants n'existent pas en elm. Pourquoi un tel problème se soulève-t-il ?

Prenons l'exemple d'un modèle simple mais suffisant. (L'exemple est évidemment trivial, mais soulève le point important de la communication inter-composants.)

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

Imaginons une fonction de vue elle aussi simple, résumant le problème :

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

On décèle immédiatement le problème. La date est ici utilisée deux fois, et on peut imaginer une update mettant à jour la toolbar. Comment renvoyer une information au footer pour indiquer la nouvelle date ?

Ce qu'il faut voir dans ce cas précis est que, contrairement à deux composants React comportant chacun leur espace mémoire propre, les deux dates résident ici dans le même espace mémoire. Il est tout à fait possible de réécrire cette fonction de vue différemment, ce qui apportera un nouveau point de vue :

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

Ici aucune magie, seules les fonctions `toolbarView` et `footerView` ont été remplacées par leur contenu (ce qui est toujours possible avec des fonctions pures). D'un coup, un détail saute tout de suite au yeux : les deux dates sont facilement accessibles du premier coup. Pourquoi ne pas réécrire le tout en enlevant la redondance ?

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

Soudainement, le problème disparait ! La date de la toolbar, autrefois hors d'atteinte dans la fonction `toolbarView` devient accessible du premier coup ! Le problème de communication disparait alors en même temps. La vraie question qui se pose alors est « Comment procéder pour retrouver ma structure de toolbar et de footer ? ». Cette question n'est plus à propos du langage, mais de la modélisation. En fusionnant simplement les deux structures de toolbar et de footer, l'inutilité de la structure de footer est immédiatement mise à jour ; là où une telle structure aurait été indispensable dans un modèle à composants. Il est temps de réécrire le modèle.

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