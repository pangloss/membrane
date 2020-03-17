# Membrane

### Membrane is a platform agnostic library for creating user interfaces.

Membrane provides 3 layers:

1. A UI framework, `membrane.component`, that deals with state management for GUIs
2. A platform agnostic model for graphics and events
3. Multiple graphics backends that provide concrete implementations for #2

While these 3 layers are made to work together, they can also be mixed and matched with other implementations. For example, you could use your favorite UI framework and the other layers to reach another platform. Alternatively, you could provide your own ncurses graphics backend and leverage the ui framework and graphics model.

The currently supported platforms are Mac OSX, Linux, and the Web via WebGL. Support for Windows and other platforms is coming soon!

[Tutorial](/docs/tutorial.md)  
[Docs](https://phronmophobic.github.io/membrane/api)  
[Examples](https://github.com/phronmophobic/membrane/tree/master/src/membrane/example)  
[Distributing your desktop app](/docs/distribution.md)  
[Targeting WebGL](/docs/webgl.md)  
<!-- Guides   -->
<!-- Design Philosophy   -->
<!-- FAQ   -->

## Usage
Leiningen dependency:

```
[com.phronemophobic/membrane "0.9.6-beta-SNAPSHOT"]
```


## A Simple Example without the UI Framework

Screenshot:
![simple counter](/docs/images/counter1.gif?raw=true)

```
(ns counter
  (:require [membrane.skia :as skia]
            [membrane.ui :as ui
             :refer [horizontal-layout
                     button
                     label
                     on]]))

(defonce counter-state (atom 0))

;; Display a "more!" button next to label showing num
;; clicking on "more!" will increment the counter
(defn counter [num]
  (horizontal-layout
   (on :mouse-down (fn [[mouse-x mouse-y]]
                     (swap! counter-state inc)
                     nil)
       (button "more!"))
   (spacer 5 0)
   (label num (ui/font nil 19))))

(comment
    ;; pop up a window that shows our counter
    (skia/run #(counter @counter-state)))

```

## Simple Example using `membrane.component` UI Framework

Screenshot:
![simple counter](/docs/images/counter2.gif?raw=true)

```

(ns counter
  (:require [membrane.skia :as skia]
            [membrane.ui :as ui
             :refer [horizontal-layout
                     vertical-layout
                     button
                     label
                     on]]
            [membrane.component :as component
             :refer [defui run-ui run-ui-sync defeffect]])
  (:gen-class))


;; Display a "more!" button next to label showing num
;; clicking on "more!" will dispatch a ::counter-increment effect
(defui counter [& {:keys [num]}]
  (horizontal-layout
   (on :mouse-down (fn [[mouse-x mouse-y]]
                     [[::counter-increment $num]])
       (ui/button "more!"))
   (ui/label num)))

(defeffect ::counter-increment [$num]
  (dispatch! :update $num inc))

(comment
  ;; pop up a window showing our counter with
  ;; num initially set to 10
  (run-ui #'counter {:num 10}))
```

Here's an exmaple of how you can use your `counter` component.

Screenshot:
![couning counter](/docs/images/counter3.gif?raw=true)

```
;; Display an "Add Counter" button
;; on top of a stack of counters
;;
;; clicking on the "Add Counter" button will
;; add a new counter to the bottom of the stack
;; 
;; clicking on the counters' "more!" buttons will
;; update their respective numbers
(defui counter-counter [& {:keys [nums]}]
  (apply
   vertical-layout
   (on :mouse-down (fn [[mx my]]
                     [[::add-counter $nums]])
       (ui/button "Add Counter"))
   (for [num nums]
     (counter :num num))))

(defeffect ::add-counter [$nums]
  (dispatch! :update $nums conj 0))

(comment
  ;; pop up a window showing our counter-counter
  ;; with nums initially set to [0 1 2]
  (run-ui #'counter-counter {:nums [0 1 2]}))

```

## Fun Features


```
;; graphical elements are values
;; no need to attach elements to the dom to get layout info
(bounds (vertical-layout
         (ui/label "hello")
         (ui/checkbox true)))
>> [30.867887496948242 29.489776611328125]


;; events are pure functions that return effects which are also values
(let [mpos [15 15]]
  (ui/mouse-down
   (ui/translate 10 10
                 (on :mouse-down (fn [[mx my]]
                                   ;;return a sequence of effects
                                   [[:say-hello]])
                     (ui/label "Hello")))
   mpos))
>> ([:say-hello])


;; horizontal and vertical centering!
(skia/run #(let [rect (ui/with-style :membrane.ui/style-stroke
                        (ui/rectangle 200 200))]
             [rect
              (ui/center (ui/label "hello") (bounds rect))]) )


;; save graphical elem as an image
(let [todos [{:complete? false
              :description "first"}
             {:complete? false
              :description "second"}
             {:complete? true
              :description "third"}]]
  (skia/draw-to-image! "todoapp.png"
                       (todo-app :todos todos :selected-filter :all)))

;; use spec to generate images of variations of your app
(doseq [[i todo-list] (map-indexed vector (gen/sample (s/gen ::todos)))]
  (skia/draw-to-image! (str "todo" i ".png")
                       (ui/vertical-layout
                        (ui/label (with-out-str
                                    (clojure.pprint/pprint todo-list)))
                        (ui/with-style :membrane.ui/style-stroke
                          (ui/path [0 0] [400 0]))
                        (todo-app :todos todo-list :selected-filter :all))))

```


That's it! For more in-depth info, check out the documentation.

[Tutorial](/docs/tutorial.md)  
[Docs](https://phronmophobic.github.io/membrane/api)  
[Examples](https://github.com/phronmophobic/membrane/tree/master/src/membrane/example)  
[Distributing your desktop app](/docs/distribution.md)  
[Targeting WebGL](/docs/webgl.md)  
<!-- Guides   -->
<!-- Design Philosophy   -->
<!-- FAQ   -->










