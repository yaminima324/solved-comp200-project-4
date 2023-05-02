Download Link: https://assignmentchef.com/product/solved-comp200-project-4
<br>
<span class="kksr-muted">Rate this product</span>

In this project we will develop a powerful strategy for building simulations of possible worlds. The strategy will enable us to make modular simulations with enough flexibility to allow us to expand and elaborate the simulation as our conception of the world expands and becomes more detailed.

One way to organize our thoughts about a possible world is to divide it up into discrete objects, where each object will have a behavior by itself, and it will interact with other objects in some lawful way. If it is useful to decompose a problem in this way then we can construct a computational world, analogous to the “real” world, with a computational object for each real object.

Each of our computational objects has some independent local state, and some rules (or code) that determine its behavior. One computational object may influence another by sending it messages. The program associated with an object describes how the object reacts to messages and how its state changes as a consequence.

You may have heard of this idea in the guise of “Object-Oriented Programming systems”(OOPs!). Lan- guages such as C++ and Java are organized around OOP. While OOP has received a lot of attention recently, it is only one of several powerful programming styles. What we will try to understand here is the essence of the idea, rather than the incidental details of their expression in particular languages.

2. An Object System

Consider the problem of simulating the activity of a few interacting agents wandering around different places in a simple world. Real people are very complicated; we do not know enough to simulate their behavior in any detail. But for some purposes (for example, to make an adventure game) we may simplify and abstract this behavior.

Let’s start with the fundamental stuff first. We can think of our object oriented paradigm as consisting of classes and instances. Classes can be thought of as the “template” for how we want different kinds of objects to behave. The way we define the class of an object is with a basic “make object” procedure; when this procedure is applied, it makes for us a particular instance.

Our object instances are themselves procedures which accept messages. An object will give you a method if you send it a message; you can then invoke that method on the object (and possibly some argu- ments) to cause some action, state update, or other computation to occur.

2.1 Classes, Instances, and Methods

For example, our simulation world will consist of named objects. We can make a named object using the procedure make-named-object. A named object is a procedure that takes a message and returns the method that will do the job you want.1 For example, if we call the method obtained from a named object by the message NAME we will get the object’s name.

<pre>(define (make-named-object name)  (let ((root-part (make-root-object)))</pre>

<pre>    (lambda (message)      (case message</pre>

1We will use the special form case to do the dispatch. See the Scheme Reference Manual for details. In essence, this acts much like a cond, matching the first argument against the first clause of each subsequent term using eq?; when it finds one that matches, it evaluates and returns the subsequent part of that expression.

2

<pre>        ((NAMED-OBJECT?) (lambda (self) #T))        ((NAME) (lambda (self) name))        ((INSTALL) (lambda (self) ’INSTALLED))        ((DESTROY) (lambda (self) ’DESTROYED))        (else (find-method message root-part))))))</pre>

<pre>(define foo (make-named-object ’george))</pre>

<pre>((foo ’NAME) foo);Value: george</pre>

The first formal parameter of every method is self. The corresponding argument must be the object that needs the job done. This was explained in lecture, and we will see it again below.

Note that a named object inherits from a root object, which we treat as the most fundamental, and simplest, of classes.

<pre>(define (make-root-object)  (lambda (message)</pre>

<pre>    (no-method)))</pre>

This object simply provides a basis for providing common behaviors to all classes, which for now is simply a way of indicating that no method is available for the desired message. We will by convention use this class as the base for all other classes.

A named object has a method for four different messages: NAMED-OBJECT?, NAME, INSTALL and DESTROY. Depending on the message, a named object will return a method that confirms that it is indeed a named-object; it will return a method to return its name; it will return a method for installation that does nothing; and it will return a method for destruction that does nothing.

In the above example, we created an instance foo, then sent it the message NAME to get its name method, and finally applied that method to the object itself to get the name. Our system provides a preferred short-hand way of putting together the method lookup and method application, using ask. What ask does here is get the NAME method from foo and then call it with foo as the argument (so the value of foo will be bound to self in the method body). The full ask procedure is defined in the file objsys.scm, but here is a simplified version that works for messages requiring no arguments:

<pre>(define (simple-ask object message)  ((get-method message object) object))</pre>

<pre>(define (get-method message object)  (object message))</pre>

<pre>(simple-ask foo ’NAME);Value: george</pre>

We see that our system also provides the procedure get-method to request a method from an object, which simply sends the message to the object. There is a special way for our objects to say there is no method: (no-method), as shown in the root-object class definition above. This returns a special value that can be used later on in our system to detect when there is no method using the method? predicate, e.g.

<pre>(method? (foo ’NAME));Value: #T</pre>

<pre>(method? (foo ’SHAPE));Value:  #F</pre>

3

2.2 Inheritance and Subclasses

A thing is another kind of computational object which will be located somewhere in our world. In the code below we see that a thing is implemented as a message acceptor that intercepts some messages. If it cannot handle a particular message itself, it passes the message along to a private, internal named object (named-object-part) that it has made as part of itself to deal with such messages (see the last line in the definition of make-thing). Thus, we may think of a thing as a kind of named object except that it also handles the messages that are special to things. This arrangement is described in various ways in object-oriented jargon, e.g., “the thing class inherits from the named-object class,” or “thing is a subclass of named-object,” or “named-object is a superclass of thing.”

<pre>(define (make-thing name location)  (let ((named-object-part (make-named-object name)))</pre>

<pre>    (lambda (message)      (case message</pre>

<pre>        ((THING?) (lambda (self) #T))        ((LOCATION) (lambda (self) location))        ((INSTALL)</pre>

<pre>         (lambda (self)         ; Install: synchronize thing and place           (ask (ask self ’LOCATION) ’ADD-THING self)           (delegate named-object-part self ’INSTALL)))</pre>

<pre>        ((DESTROY)         (lambda (self)         ; Destroy: remove from place</pre>

<pre>           (ask (ask self ’LOCATION) ’DEL-THING self)</pre>

<pre>           (delegate named-object-part self ’DESTROY)))        ((EMIT)</pre>

<pre>         (lambda (self text)         ; Output some text           (ask screen ’TELL-ROOM (ask self ’LOCATION)</pre>

<pre>                (append (list "At" (ask (ask self ’LOCATION) ’NAME))                        text))))</pre>

<pre>        (else (get-method message named-object-part))))))</pre>

There are several other interesting aspects of the thing class definition above. We see that a thing instance will respond to the THING? message with a procedure that, when applied to the instance, will return #T. But an object that is not a thing will not find the THING? message and an error will result. To get around this problem, and for improved convenience as well, our system provides a procedure is-a that can be used to check the class of an object.

<pre>(define (is-a object type-pred)  (if (not (procedure? object))</pre>

<pre>      #f      (let ((method (get-method type-pred object)))</pre>

<pre>        (if (method? method)            (ask object type-pred)            #F))))</pre>

<pre>(define my-book (make-thing ’great-gatsby library))</pre>

<pre>((get-method ’THING? my-book) my-book);Value: #T((get-method ’NAMED-OBJECT? my-book) my-book);Value: #T</pre>

4

<pre>(is-a my-book ’THING?);Value: #T(is-a my-book ’NAMED-OBJECT?);Value: #T(is-a my-book ’EMOTION?);Value: #F</pre>

This enables us to ask an object if it is an instance of a particular class. For example, we can see that a thing we make is a thing, but also is a named-object (you can assume that library is a location previously made). How does the is-a procedure work? If we ask for the THING? method from a thing instance (my-book, in this case), my-book immediately gets and returns the method defined in make-thing. However, if we ask for the NAMED-OBJECT? method from my-book, the my-book object passes the message along to its internal named-object-part, where the NAMED-OBJECT? method is finally found and returned. The is-a utility procedure tries to find the appropriate type check method, and if found invokes it on the object, otherwise concluding that the object is not an instance of the requested type.

2.3 Delegation

Another idea shown in the “thing” class (which is specified by the make-thing procedure above) is that of delegation, which is the explicit use of an “internal” object’s method by the object. In the thing class, we see that the INSTALL method “shadows” or intercepts the INSTALL method in the named-object class. In make-thing, we want to first do some work to integrate the thing object into our simulation world (more on that later), but then we also want to invoke the superclass named-object INSTALL method in case something important happens there as well. But since the internal named-object-part is really not a “stand-alone” object all its own, we don’t ask it to do something on its own, instead we delegate the task to the internal object. To delegate is to have the internal object do the requested work, but on behalf of the full self object.

The important difference is that if we ask an object to do something, then the self value passed to the method will be the object itself. Using delegate, on the other hand, we can explicitly control what the self value will be that is passed to the method, and can thus have a part (inherited superclass) of the object do something to the whole object. This is perhaps the single most subtle and difficult aspect of our system, and you will explore this idea and issue in more detail in the exercises.

3. Classes for a Simulated World

When you read the code in objtypes.scm, you will see definitions of several different classes of objects that define a host of interesting behaviors and capabilities using the OOP style discussed in the previous section. Here we give a brief “tour” of some of the important classes in our simulated world.

3.1 Container Class

Once we have things, it is easy to imagine that we might want containers for things. We can define a utility container class as shown below:

<pre>(define (make-container)  (let ((root-part (make-root-object))</pre>

<pre>        (things ’())) ; a list of THING objects in container    (lambda (message)</pre>

<pre>(case message</pre>

5

<pre>        ((CONTAINER?) (lambda (self) #T))        ((THINGS) (lambda (self) things))        ((HAVE-THING?)</pre>

<pre>         (lambda (self thing)  ; container, thing -&gt; boolean           (not (null? (memq thing things)))))</pre>

<pre>        ((ADD-THING)         (lambda (self new-thing)</pre>

<pre>           (if (not (ask self ’HAVE-THING? new-thing))               (set! things (cons new-thing things)))</pre>

<pre>           ’DONE))        ((DEL-THING)</pre>

<pre>         (lambda (self thing)           (set! things (delq thing things))           ’DONE))</pre>

<pre>        (else (find-method message root-part))))))</pre>

Notice that a container does not inherit from named-object, so it does not support messages such as NAME or INSTALL. Containers are not meant to be stand-alone objects; rather, they are only meant to be used internally by other objects to gain the capability of adding things, deleting things, and checking if one has something.

3.1 Place Class

Our simulated world needs places (e.g. rooms or spaces) where interesting things will occur. The definition of the place class is shown below.

<pre>(define (make-place name)  (let ((named-obj-part (make-named-object name))</pre>

<pre>        (container-part (make-container))</pre>

<pre>        (exits ’()))    ; a list of exit    (lambda (message)</pre>

<pre>      (case message        ((PLACE?) (lambda (self) #T))        ((MAKE-NOISE)</pre>

<pre>         (lambda (self who)           (let ((interested (find-all self ’AWARE?)))</pre>

<pre>             (for-each (lambda (a) (ask a ’HEARD-NOISE who)) interested)</pre>

<pre>             ’noise-made)))        ((EXITS) (lambda (self) exits))        ((EXIT-TOWARDS)</pre>

<pre>         (lambda (self direction)  ; place, symbol -&gt; exit | #F           (find-exit-in-direction exits direction)))</pre>

<pre>        ((ADD-EXIT)         (lambda (self exit)</pre>

<pre>           (let ((direction (ask exit ’DIRECTION)))             (cond ((ask self ’EXIT-TOWARDS direction)</pre>

<pre>                    (error (list name "already has exit" direction)))                   (else</pre>

<pre>                    (set! exits (cons exit exits))                    ’DONE)))))</pre>

<pre>        (else         (find-method message container-part named-obj-part))))))</pre>

If we look at the first and last lines of make-place, we notice that place inherits from two different classes: it has both an internal named-object-part and an internal container-part. Here we use

6

the object oriented system procedure find-method (defined in objsys.scm) which will try to find the first matching method by looking (in order) in the provided internal objects. Thus, if we ask for the NAME method from a place instance, the method will be found in the internal named-object-part, while if we ask for the HAVE-THING? method from a place instance, the appropriate method will be found and returned from the internal container-part object. This idea is often termed “multiple inheritance”.

You can also see that our place instances will each have their own internal variable exits, which will be a list of exit instances which lead from one place to another place. In our object-oriented terminology, we can say the place class establishes a “has-a” relationship with the exit class. You should examine the objtypes.scm file to understand the definition for make-exit.

3.2. Mobile-thing Class

Now that we have things that can be contained in some place, we might also want mobile-things (made by make-mobile-thing) that can CHANGE-LOCATION.

<pre>(define (make-mobile-thing name location)  (let ((thing-part (make-thing name location)))</pre>

<pre>    (lambda (message)      (case message</pre>

<pre>        ((MOBILE-THING?) (lambda (self) #T))        ((LOCATION)     ; This shadows message to thing-part!</pre>

<pre>         (lambda (self) location))        ((CHANGE-LOCATION)</pre>

<pre>         (lambda (self new-location)           (ask location ’DEL-THING self)           (ask new-location ’ADD-THING self)           (set! location new-location)))</pre>

<pre>        ((ENTER-ROOM)         (lambda (self exit) #t))</pre>

<pre>        ((LEAVE-ROOM)         (lambda (self exit) #t))</pre>

<pre>        ((CREATION-SITE)         (lambda (self)</pre>

<pre>           (delegate thing-part self ’location)))        (else (get-method message thing-part))))))</pre>

When a mobile thing moves from one location to another it has to tell the old location to DEL-THING from its memory, and tell the new location to ADD-THING. Note that here we use the ask procedure, since we are sending a message to the specified location objects that exist external to the mobile-thing; it would be inappropriate to delegate in this situation.

3.3. Person Class

A person is a kind of mobile thing. When a person is made, an internal mobile thing is also made to handle messages such as CHANGE-LOCATION. The mobile thing is bound to a variable that is visible only within the person object – mobile-thing-part. When a person moves from one place to another, it does so by using the CHANGE-LOCATION method from its internal mobile-thing-part. However, it is the person that moves. Thus, it is the person that must be added or removed from the location, not the mobile thing from which the method was obtained. The internal mobile-thing-part is not a whole person – it is only a fragment of the person. To implement the desired behavior the CHANGE-LOCATION method needs to know the complete or whole moving object (the person), and this is what is passed to the method as self. This is crucial for you to understand if your objects are to maintain their integrity!

7

If we consider the (partial) definition of make-person, we also notice that a person is a container as well as a mobile thing. Again, this is an example of multiple inheritance. The idea here is that people can also “contain things” which they carry around with them when they move.

A person can SAY a list of phrases. A person can TAKE something, as well as DROP something. Some of the other messages a person can handle are briefly shown below; you should consult the full definition of make-person in objtypes.scm to understand the full set of capabilities a person instance has.

<pre>(define (make-person name birthplace)  (let ((mobile-thing-part (make-mobile-thing name birthplace))</pre>

<pre>        (container-part    (make-container))        (health            3)        (strength          1))</pre>

<pre>    (lambda (message)      (case message</pre>

<pre>        ((PERSON?) (lambda (self) #T))        ((STRENGTH) (lambda (self) strength))        ((HEALTH) (lambda (self) health))        ((SAY)</pre>

<pre>         (lambda (self list-of-stuff)           (ask screen ’TELL-ROOM (ask self ’location)</pre>

<pre>                (append (list "At" (ask (ask self ’LOCATION) ’NAME)                                 (ask self ’NAME) "says --")</pre>

<pre>                           list-of-stuff))))</pre>

<pre>        ((HAVE-FIT)         (lambda (self)</pre>

<pre>           (ask self ’SAY ’("Yaaaah! I am upset!"))</pre>

<pre>           ’I-feel-better-now))        ((PEOPLE-AROUND) (lambda (self) ...))        ...        ((TAKE) (lambda (self thing) ...))        ((LOSE)</pre>

<pre>         (lambda (self thing lose-to)           (ask self ’SAY (list "I lose" (ask thing ’NAME)))           (ask self ’HAVE-FIT)           (ask thing ’CHANGE-LOCATION lose-to)))</pre>

<pre>        ((DROP)         (lambda (self thing)</pre>

<pre>           (ask self ’SAY (list "I drop" (ask thing ’NAME)                                "at" (ask (ask self ’LOCATION) ’NAME)))</pre>

<pre>           (ask thing ’CHANGE-LOCATION (ask self ’LOCATION))))        ...</pre>

<pre>        (else (find-method message mobile-thing-part container-part))))))</pre>

3.4 Avatar Class

One kind of character you will use in this problem set is an avatar. The avatar is a kind of person who must be able to do the sorts of things a person can do, such as TAKE things or GO in some direction. However, the avatar must be able to intercept the GO message, to do things that are special to the avatar, as well as to do what a person does when it receives a GO message. This is again accomplished by explicit delegation. The avatar does whatever it has to, and in addition, it delegates to its internal person the processing of the GO message, with the avatar as self. Notice that we have a fairly fine degree of control over how inheritance and delegation are managed. In the case of the avatar, we first delegate to the internal person to handle the GO message, and then do something more after that (in this case, invoke the simulation clock).

8

<pre>(define (make-avatar name birthplace murder-details)  (let ((person-part (make-person name birthplace)))</pre>

<pre>    (lambda (message)      (case message</pre>

<pre>        ((AVATAR?) (lambda (self) #T))        ((LOOK-AROUND)          ; report on world around you</pre>

<pre>         (lambda (self) ...))        ((GO)</pre>

<pre>         (lambda (self direction)  ; Shadows person’s GO           (let ((success? (delegate person-part self ’GO direction)))</pre>

<pre>             (if success? (ask clock ’TICK))</pre>

<pre>             success?)))        ...</pre>

<pre>        ((TAKE) (lambda (self thing) ...))        (else (get-method message person-part))))))</pre>

The avatar also implements an additional message, LOOK-AROUND, that you will find very useful when running simulations to get a picture of what the world looks like around the avatar.

3.5 Autonomous-person Class

Our world would be a rather lifeless place unless we had objects that could somehow “act” on their own. We achieve this by further specializing the person class. An autonomous-player is a person who can move or take actions at regular intervals, as governed by the clock through a callback.

Our clock works by using what are known as “callbacks”. This means that we create an instruction which we install in the clock, with the property that every time the clock iterates, it executes all the instructions it has stored up. Each of these instructions sends a message to an object, causing it to synchronously execute an action. In the example below, installing an autonomous person causes the clock object to add an instruction that will send this object a “move-and-take-stuff” message, which will then cause this object to select an action. See the discussion on the clock in the objsys.scm file for details on how the clock operates. However, the template used below for sending the clock a “callback” will be valuable to you in creating your own objects and methods. Also note how, when an autonomous player dies, we send a “remove-callback” message to the clock, so that we stop asking this character to act.

<pre>(define (make-autonomous-player name birthplace activity miserly)  (let ((person-part (make-person name birthplace)))</pre>

<pre>    (lambda (message)      (case message</pre>

<pre>        ((AUTONOMOUS-PLAYER?) (lambda (self) #T))        ((INSTALL) (lambda (self)</pre>

<pre>                     (ask clock ’ADD-CALLBACK                          (make-clock-callback ’move-and-take-stuff self</pre>

<pre>                                               ’MOVE-AND-TAKE-STUFF))                     (delegate person-part self ’INSTALL)))</pre>

<pre>        ((MOVE-AND-TAKE-STUFF)         (lambda (self)</pre>

<pre>           ;; first move           (let loop ((moves (random-number activity)))</pre>

<pre>             (if (= moves 0)                 ’done-moving</pre>

<pre>                 (begin                   (ask self ’MOVE-SOMEWHERE)                   (loop (- moves 1)))))</pre>

9

<pre>           ;; then take stuff           (if (= (random miserly) 0)</pre>

<pre>                  (ask self ’TAKE-SOMETHING))           ’done-for-this-tick))</pre>

<pre>        ((DIE)         (lambda (self)</pre>

<pre>           (ask clock ’REMOVE-CALLBACK self ’move-and-take-stuff)</pre>

<pre>           (delegate person-part self ’DIE)))        ((MOVE-SOMEWHERE)</pre>

<pre>         (lambda (self)           (let ((exit (random-exit (ask self ’LOCATION))))</pre>

<pre>             (if (not (null? exit)) (ask self ’GO-EXIT exit)))))        ((TAKE-SOMETHING)</pre>

<pre>         (lambda (self)           (let* ((stuff-in-room (ask self ’STUFF-AROUND))</pre>

<pre>                  (other-peoples-stuff (ask self ’PEEK-AROUND))</pre>

<pre>                  (pick-from (append stuff-in-room other-peoples-stuff)))             (if (not (null? pick-from))</pre>

<pre>                 (ask self ’TAKE (pick-random pick-from))                 #F))))</pre>

<pre>        ...        (else (get-method message person-part))))))</pre>

3.6 Installation

One final note about our system. If you look in objtypes.scm, you’ll see that objects have an INSTALL method which does some appropriate initialization for a newly made object. For example, if you cre- ate a new mobile thing at a place, the object must be added to the place. As you’ll see in the code, we define two procedures for each type of object: make- and a create- procedure. The make proce- dure (e.g. make-person) simply makes a new instance of the object, while the create procedure (e.g. create-person) both (1) makes the object and (2) installs it. When you create objects in our simula- tion world, you should do this using the appropriate create procedure. Thus, to create a new person, use create-person rather than calling make-person directly.

The following distinction should also help you think about make-object versus create-object procedures. The make-object procedure should only be used “inside” our object oriented programming code: e.g., in objtypes.scm you “make” a stand-alone person or part of an person using, for example make-person or make-named-object or whatever. But this only gives you an object that is not yet connected up with our world. To get a fully functioning object in a particular world, you need to “create” that object. Thus you should use the create-object variant when you actually want to make and install an object in a simulation world, as we do in setup.scm.

Our world is built by the setup procedure that you will find in the file setup.scm. You are the deity of this world. When you call setup with your name, you create the world. It has rooms, objects, and people based on a small liberal arts school on the hills of a mighty city; and it has an avatar (a manifestation of you, the deity, as a person in the world). The avatar is under your control. It goes under your name and is also the value of the globally-accessible variable me. Each time the avatar moves, simulated time passes in the world, and the various other creatures in the world take a time step. The way this works is that there is a clock that sends an activate message to all callbacks that have been created. This causes certain objects to perform specific actions. In addition, you can cause time to pass by explicitly calling the clock, e.g. using (run-clock 20).

If you want to see everything that is happening in the world, do

10

<pre>(ask screen ’DEITY-MODE #t)</pre>

which causes the system to let you act as an all-seeing god. To turn this mode off, do

<pre>(ask screen ’DEITY-MODE #f)</pre>

in which case you will only see or hear those things that take place in the same place as your avatar is. To check the status of this mode, do

<pre>(ask screen ’DEITY-MODE?)</pre>

To make it easier to use the simulation we have included a convenient procedure, thing-named for referring to an object at the location of the avatar. This procedure is defined at the end of the file objsys.scm.

When you start the simulation, you will find yourself (the avatar) in one of the locations of the world. There are various other characters present somewhere in the world. You can explore this world, but the real goal is to get a diploma from Prof. Yuret on the graduation stage, and get out into the “real” world.

Here is a sample run of a variant of the system (we have added a few new objects to this version but it gives you an idea of what will happen). Rather than describing what’s happening, we’ll leave it to you to examine the code that defines the behavior of this world and interpret what is going on.

<pre>(setup ’george);Value: ready</pre>

<pre>(ask (ask me ’location) ’name);Value: soccer-field</pre>

<pre>(ask me ’look-around)</pre>

<pre>You are in soccer-fieldYou are not holding anything.You see stuff in the room: footballYou see other people: lambda-manThe exits are in directions: south;Value: ok</pre>

<pre>(ask me ’take (thing-named ’football))</pre>

<pre>At soccer-field george says -- I take football from soccer-field;Value: #[unspecified-return-value]</pre>

<pre>(run-clock 3)</pre>

<pre>suzy moves from suzy-cafe to student-centerlambda-man moves from soccer-field to eng-buildinglambda-man moves from eng-building to soccer-fieldAt soccer-field lambda-man says -- Hi georgeAt soccer-field lambda-man says -- I take football from georgeAt soccer-field george says -- I lose footballAt soccer-field george says -- Yaaaah! I am upset!comp200-student moves from library to gymcomp200-student moves from gym to libraryAt library comp200-student says -- I take engineering-book from library</pre>

11

<pre>prof-yuret moves from migros to cici-bufeAt cici-bufe prof-yuret says -- Hi ciciAt cici-bufe prof-yuret says -- I take kofte from cici-bufealyssa-p-hacker moves from eng-auditorium to eng-z21alyssa-p-hacker moves from eng-z21 to eng-auditoriumben-bitdiddle moves from graduation-stage to great-courtben-bitdiddle moves from great-court to cas-building--- the-clock Tick 0 ---At cici-bufe cici says -- Prepare to suffer, prof-yuret !At cici-bufe prof-yuret says -- Ouch! 3 hits is more than I want!At cici-bufe prof-yuret says -- SHREEEEK!  I, uh, suddenly feel very faint...At cici-bufe prof-yuret says -- I lose kofteAt cici-bufe prof-yuret says -- Yaaaah! I am upset!An earth-shattering, soul-piercing scream is heard...prof-yuret moves from cici-bufe to heavenlambda-man moves from soccer-field to eng-buildinglambda-man moves from eng-building to eng-z21comp200-student moves from library to great-courtcomp200-student moves from great-court to student-centerAt student-center comp200-student says -- Hi suzyalyssa-p-hacker moves from eng-auditorium to eng-z21At eng-z21 alyssa-p-hacker says -- Hi lambda-manAt eng-z21 alyssa-p-hacker says -- I take problem-set from eng-z21ben-bitdiddle moves from cas-building to sos-building--- the-clock Tick 1 ---cici moves from cici-bufe to computer-clubAt student-center suzy says -- Prepare to suffer, comp200-student !At student-center comp200-student says -- Ouch! 3 hits is more than I want!At student-center comp200-student says -- SHREEEEK!  I, uh, suddenly feel very faint...At student-center comp200-student says -- I lose engineering-bookAt student-center comp200-student says -- Yaaaah! I am upset!An earth-shattering, soul-piercing scream is heard...comp200-student moves from student-center to heavenAt heaven comp200-student says -- Hi prof-yuretlambda-man moves from eng-z21 to eng-auditoriumlambda-man moves from eng-auditorium to eng-z21At eng-z21 lambda-man says -- Hi alyssa-p-hackeralyssa-p-hacker moves from eng-z21 to eng-b30At eng-b30 alyssa-p-hacker says -- I try but cannot take white-boardben-bitdiddle moves from sos-building to cas-building--- the-clock Tick 2 ---;Value: done</pre>

<pre>(ask screen ’deity-mode #f);Value: #t</pre>

<pre>(ask me ’go ’south)</pre>

<pre>george moves from soccer-field to eng-building--- the-clock Tick 3 ---;Value: #t</pre>

<pre>(ask me ’go ’in)</pre>

12

<pre>george moves from eng-building to eng-z21At eng-z21 lambda-man says -- Hi georgelambda-man moves from eng-z21 to eng-auditoriumAt eng-z21 alyssa-p-hacker says -- Hi georgealyssa-p-hacker moves from eng-z21 to eng-building--- the-clock Tick 4 ---;Value: #t</pre>

<pre>(run-clock 3)</pre>

<pre>At eng-z21 lambda-man says -- Hi georgelambda-man moves from eng-z21 to eng-b30--- the-clock Tick 5 ---At eng-z21 lambda-man says -- Hi georgelambda-man moves from eng-z21 to eng-building--- the-clock Tick 6 ---</pre>

<pre>--- the-clock Tick 7 ---;Value: done</pre>

3.7 Changing the World

In parts of this project, you will be asked to elaborate or enhance the world (e.g. create new objects like the ones in setup.scm), as well as add to the behaviors or kinds of objects in the system (e.g. add modified classes to your submission file like the ones in objtypes.scm). If you do make such changes, you must remembertore-evaluatealldefinitionsandre-run(setup ’your-name),justtomakesurethatallyour definitions are up to date. An easy way to do this is to reload all the files (be sure to save your files to disk beforereloading),andthenre-evaluate(setup ’your-name).

4. Warm Up Exercises

Warm up exercises below will not be graded and we are not asking you to submit them. However, It is highly recomended that you do these exercises to capture the structure of the system. You should prepare these exercises early, in order to get a sense for the world you will be exploring.

Exercise 1: In the transcript above there is a line: (ask (ask me ’location) ’name). What kind of value does (ask me ’location) return here? What other messages, besides name, can you send to this value?

Exercise 2: Look through the code in objtypes.scm to discover which classes are defined in this system and how the classes are related. For example, place is a subclass of named-object. Also look through the code in setup.scm to see what the world looks like. Draw a class diagram and a skeletal instance diagram like the ones presented in lecture. You will find such a diagram helpful (maybe indispensable) in doing the programming assignment.

Exercise 3: Look at the contents of the file setup.scm. What places are defined? How are they inter- connected? Draw a map. You must be able to show the places and the exits that allow one to go from one place to a neighboring place.

13

Exercise 4: Aside from you, the avatar, what other characters roam this world? What sorts of things are around? How is it determined which room each person and thing starts out in?

Exercise 5: The avatar, as a person, may have possessions. How does the avatar handle the request (ask me ’things)?Inparticular,whichmethodisusedtorespondtotherequestandwhichvariableholdsthe list of possessions? Sketch a skeletal environment diagram to help. Note that we are not asking you to draw a fully detailed environment diagram here—it is huge and more confusing than helpful!

Exercise 6: Start the the simulation by typing (setup ’&lt;your name&gt;).Walk the avatar to a room that has an unowned object. Have the avatar take this object, only to drop

it somewhere else.

Exercise 7: You may find it useful to draw an environment diagram, in order to understand how objects inherit methods from other objects. For example, you might draw an environment diagram showing the state of the environment after evaluating:

<pre>(define foo (make-mobile-object ’george student-center))</pre>

Assume that student-center is bound to some procedure, but don’t worry about the details of that procedure.

Further, show the state of the environment after evaluating

<pre>(ask foo ’location)</pre>

Don’t worry about showing the frames created by calling ask or ask-helper.

Though it is more work, you may find it useful to think about what happens when other methods, such as install or name are called.

5. Programming Assignment

See that the three files objsys.scm, objtypes.scm and setup.scm are loaded for you in the project4.scm. Make sure that project4.scm loads correctly. Then start the simulation by typ- ing(setup ’&lt;your name&gt;).Playwiththeworldabit.Onesimplethingtodoistostaywhereyouare andruntheclockforawhilewith(run-clock &lt;ticks&gt;).Sincethecharactersinoursimulatedworld have a certain amount of restlessness, people should come walking by and say “Hi” to you. Try running the clock with the screen’s deity-mode parameter set to both true and false. When it is set to true, you see almost everything that happens everywhere in the simulation. When it is set to false, you see only what happens in the room you are in. You should set deity-mode to false when you are ready to “play” the game (that is to get Prof. Yuret and a diploma on the graduation stage at the same time so that you can graduate).

What to turn in: When preparing your answers to the questions below, please just turn in the procedures that you have either written or changed (highlighting the actual portions changed) for each problem. Put each of your solutions into your submission file, and submit that based on the “Project Submission Instructions” handout. Do not submit your transcript, written answers etc. in the files other than given. Only the programming exercises will be graded. You only need to submit your project4.scm file. Please do not overwhelm your TA with huge volumes of material!!

14

Setting up the world After setting up, do the following actions using appropriate commands:

• Check where your avatar is.• Make your avatar say its name.• Make your avatar say ”Hello World”.• Make your avatar go to another room.• Make your avatar take stuff from the place it is in. • Make your avatar toss the stuff it has.• Intentionally kill your avatar.

Understanding installation Note how install is implemented as a method defined as part of thing and autonomous-person. Notice that the autonomous-person version sends a callback to the clock that will “animate” the person, then delegates an install message from its self to its internal thing, which contains the INSTALL method responsible for adding the person to its birthplace. The relevant details of this situation are outlined in the code excerpts below:

<pre>(define (make-autonomous-person name birthplace laziness)  ;; Laziness determines how often the person will move.  (let ((person-part (make-person name birthplace)))</pre>

<pre>    ...      (case message</pre>

<pre>        ...        ((INSTALL)</pre>

<pre>         (lambda (self)           (ask clock ’ADD-CALLBACK</pre>

<pre>                (make-clock-callback ’move-and-take-stuff self                                     ’MOVE-AND-TAKE-STUFF))</pre>

<pre>           (delegate person-part self ’INSTALL)))   ; **        ...)))</pre>

<pre>(define (make-thing name location)  (let ((named-object-part (make-named-object name)))</pre>

<pre>    ...      (case message</pre>

<pre>        ...        ((INSTALL)</pre>

<pre>         (lambda (self) ; Install: synchronize thing and place             ...</pre>

<pre>             (ask (ask self ’LOCATION) ’ADD-THING self)             (delegate named-object-part self ’INSTALL))</pre>

…))))))Louis Reasoner suggests that it would be simpler if we change the last line of the make-autonomous-person

versionoftheinstallmethod(marked; **)toread:(ask person-part ’INSTALL) )) ; **

15

Alyssa points out that this would be a bug. “If you did that,” she says, “then when you make and install an autonomous person, and this person moves to a new place, he’ll be in two places at once!”

What does Alyssa mean? Specifically, what goes wrong? You may need to draw an appropriate envi- ronment diagram to help you to explain carefully.

Who just died? Explore the world until “An earth-shattering, soul-piercing scream is heard…”, which means that someone (hopefully not you) has just been murdered. Where does the victim go? If you know where the victim goes (and assuming you are not in deity-mode), what simple scheme expression can you evaluate to find out who just died?

Programming Exercise: But I’m too young to die!!

In the current setting, once a person’s health becomes negative, they die. Implement a simple kind of reincarnation, with the following behavior. When a person dies, they still will lose all of their possessions (check the code for person to see how this happens). However, a person will initially start with 3 lives. If upon time of death they still have lives left, they lose a life, but reappear in their birthplace. Thus, a person can continue through several death cycles. Implement this change, by modifying the make-person procedure.Thenreinstalltheworldandtestouttheidea(e.g.byasking(ask me ’die)).

NOW, FOR SOME REAL CHANGES! In the next several exercises you will extend the system to add additional behaviors and nuances.

Programming Exercise: Perhaps to arm oneself against a sea of ….

If you have been wandering around the world, you will have discovered that Suzy and Cici, two char- acters that live in the student center, have a penchant for eating people. So it might help if you can defend yourself against them.

Implement a new class of object, a weapon. Since a weapon should be something that can be transported from place to place, you should think about the class of objects from which it should inherit. The procedure that makes instances of a weapon should take as inputs a name, a location, and a maximal amount of damage that it can inflict (see setup.scm for the order of these arguments). A weapon should support these methods:

<ul>

 <li>WEAPON?: return #t, to indicate this is a weapon,</li>

 <li>DAMAGE: return the amount of damage the weapon can inflict,</li>

 <li>HIT: given the person using the weapon, and a target for the weapon, this method should “emit” some information about who is hitting whom with what, and then should cause the target to SUFFER an amount of damage. In particular, the amount of damage suffered should be a the DAMAGE of the weapon, and at least 1. An example of this in use might be<pre>     (ask (thing-named ’serious-weapon) ’hit me (thing-named     ’ben-bitdiddle))</pre>Turn in a listing of your procedure, and a transcript of your testing of it.Once you have tested out your code, you can uncomment the corresponding line in setup.scm to populate the world with weapons. Fell free to introduce your own weapons to the world. After that, try running the game, in which you look for a weapon, then use it to attack a target.</li>

</ul>

16

Programming Exercise: Good thing I’m armed and dangerous

To even up the game a bit, let’s allow other autonomous players also to use weapons. Create a new kind of object, a violent-person. This object should inherit the behaviors of an autonomous player, but with a few changes:

<ul>

 <li>Creating an instance of a violent person should include a parameter that specifies how frequently the person is violent;</li>

 <li>Upon installation, a callback should be sent to the clock, that will ask this object potentially to engage in a violent act on each clock tick;</li>

 <li>ACT-VIOLENTLY: The actual method for a violent act should first decide at random whether to actually do something violent (e.g. if the frequency parameter was 4, then with a 1 in 4 probability, the person should act violently). If the decision is affirmative, then the person should pick another person in this location at random, select a weapon from among its possessions at random, and assuming there is both a victim and a weapon it should hit the target victim with the weapon.Once you have tested out your code, you can change the code for the setup procedure to add violent players in the world (or change autonomous players to violent players) , and try running the game, in which you look for a weapon, then use it to attack a target, while hoping that others don’t attack you first.Programming Exercise: A good hacker could defuse this situationJust to make life a bit more difficult, let’s add some explosive devices to our world. In particular, we want you to create a class of object called a bomb. A bomb should inherit properties of mobile things (since they can be moved) and of aware things. If you look at the code for this project, you will see that an aware thing is simply an object that can sense noises, though initially it does nothing about it.However, when a person enters a room, he or she MAKES-NOISE, which causes the location to have HEARD-NOISE, and this in turn causes every AWARE object in that location to HEARD-NOISE. As a consequence, any AWARE object in a room is now able to react to the noise, provided they have an appropriate method. Building on this idea, a bomb should have the following properties:

  <ul>

   <li>ARM: a bomb can be armed by sending it an ARM message. This should set some appropriate internal state variable in the bomb.</li>

   <li>DISARM: a bomb can be disarmed by sending it a DISARM message. This should set some appropri- ate internal state variable in the bomb.</li>

   <li>TRIGGER: a bomb can be sent a TRIGGER message. If it is armed, this message will ACTIVATE the bomb, otherwise nothing will happen.</li>

   <li>HEARD-NOISE: If a bomb hears a noise, it attempts to TRIGGER itself.</li>

   <li>ACTIVATE: If a bomb successfully activates, it finds all the people at its location, does some pre- specified amount of damage (or suffering) to each, and destroys itself. It should probably also EMIT some information about what has happened.After implementing this idea of a bomb, you can modify your setup procedure to populate the world with some bombs. Have your character wander around, arm some bombs, and you can see that they detonate as expected.</li>

  </ul></li>

</ul>