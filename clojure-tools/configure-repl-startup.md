# Configure REPL on Startup
A Clojure REPL starts in the `user` namespace by default.  Clojure will automatically load code from a `user.clj` file into the REPL to carry out common startup tasks.

* load project code into the REPL by requiring namespaces
* set the default namespace with `in-ns`
* call functions to run an application or service
* start components (i.e for mount, component, integrant)

> #### HINT:: Example project
> [practicalli/clojure-configure-repl](https://github.com/practicalli/clojure-configure-repl) project contains example code for configuring the REPL start up


## Create a `dev/user.clj` file and `:dev` alias
Create a `dev/user.clj` file with a namespace called `user`.

`user.clj` should include a namespace definition

```clojure
(ns user)
```

[practicalli/clojure-deps-edn]({{ P9IClojureDepsEdn }}) includes a `:dev` alias which adds the `dev` directory to the project classpath.
  Alternatively, edit the `deps.edn` file and add the following code:

```clojure
 :dev
  {:extra-paths ["dev"]}
```

Running a Clojure REPL with the `-A:dev` alias will make the `dev/user.clj` file available to be loaded by the REPL.

In this example the `dev/` path is added to the project and then the REPL is run using rebel readline.

```shell
clojure -A:dev:rebel
```

> #### Hint::Using the `dev/` directory
> The `user.clj` code should not be included in live deployments, such as jars and uberjars.  Including the `dev/` directory via the :dev alias keeps the `user.clj` and any other development only code separate from deployment actions.


## Requiring namespaces
By requiring a namespace in the `dev/user.clj` file, the code defined in that namespace will be loaded into the REPL once started.

Add a require expression to the namespace definition in `dev/user.clj`

```clojure
(ns user
  :require [practicalli.project-namespace])
```

Require loads all the expressions into the REPL, so functions are immediately available.


## Calling functions
Functions from the required namespace can be called, to start the application for example.

```clojure
(ns user
  :require [practicalli.project-namespace])

(practicalli.project-namespace/-main)
```

## Changing Namespaces
When using the REPL directly it can be changed into a specific namespace using the `in-ns` function.

Changing to the project namespace allows the `-main` namespace to be called
```clojure
(ns user
  :require [practicalli.project-namespace])

(in-ns 'practicalli.project-namespace)

(-main)
```


## Starting Component Lifecycle Services
Clojure has several library to manage the lifecycle of components that make up the application, especially those components with state. Components can be started and stopped in a specific order.

Example component lifecycle libraries included

* [mount](https://github.com/tolitius/mount)
* [integrant](https://github.com/weavejester/integrant)
* [component](https://github.com/stuartsierra/component)

In Clojure it is idiomatic to define the component lifecyle services in a namespace called `dev`.  In the `def/user.clj` file, add the following `ns` declaration to require the `dev` namespace and change to that namespace with `in-ns`

```clojure
(ns user
  (:require [dev]))

(in-ns 'dev)
```
Now define code in the `dev/dev.clj` file that controls the component lifecycle services library for the project.


## Example project with component lifecycle
{% tabs mount="Mount", integrant="Integrant", component="Component" %}

<!-- Mount example -->
{% content "mount" %}
Using mount, its common to define a `dev.clj` file with `go`, `stop` and `restart` functions that manage the lifecycle of mount components.  A `start` function contains the list of components with optional state.

Require the mount namespace and the main namespace for the project, which should contain all the code to start and stop services.

```clojure
(ns user
  :require [mount.core :refer [defstate]]
           [practicalli.app.main])
```



```clojure
(defn start []
  (with-logging-status)
  (mount/start #'practicalli.app.conf/environment
               #'practicalli.app.db/connection
               #'practicalli.app.www/business-app
               #'practicalli.app.service/nrepl))
```

The `go` function calls `start` and marks all components as ready.
```clojure
(defn go
  "Start all states defined by defstate"
  []
  (start)
  :ready)
```

The `stop` function stops all components, removing all non-persistent state.

```clojure
(defn stop [] (mount/stop))
```

The reset function that calls `stop`, refreshes the namespaces so that stale definitions are removed and starts all components (loading in any new code).
```clojure
(defn reset
  "Stop all states defined by defstate.
  Reload modified source files and restart all states"
  []
  (stop)
  (namespace/refresh :after 'dev/go))
```

* [Example dev.clj file for mount](https://github.com/tolitius/mount/blob/master/dev/clj/dev.clj)

> #### Hint::Use `dev` namespace during development
> Require `practicalli.app.dev` namespace rather than main, to start components in a development environment.


<!-- Integrant example -->
{% content "integrant" %}

TODO: [pull requests accepted]({{ book.P9IPullRequestsUrl }})

<!-- Component example -->
{% content "component" %}

TODO: [pull requests accepted]({{ book.P9IPullRequestsUrl }})


{% endtabs %}
<!-- End of Clojure editors -->


## Reference
* [Mount project on GitHub](https://github.com/tolitius/mount)
* [Mount - collection of Clojure/Script mount apps](https://github.com/tolitius/stater)
* [Component](https://github.com/stuartsierra/component)
* [A tutorial to Stuart Sierra's Component](https://cb.codes/a-tutorial-of-stuart-sierras-component-for-clojure/)
* [Refactoring to Components](https://lacinia.readthedocs.io/en/latest/tutorial/component.html) - Walmart Labs Lacinia
* [Integrant](https://github.com/weavejester/integrant)
* [Compojure and Integrant](https://the-frey.github.io/2017/12/14/compojure-and-integrant)
* [Build a Clojure web app using Duct](https://circleci.com/blog/build-a-clojure-web-app-using-duct/) - CircleCI
* [Reloading Woes - Lambda island](https://lambdaisland.com/blog/2018-02-09-reloading-woes)