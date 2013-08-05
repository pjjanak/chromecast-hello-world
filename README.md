# What's This? #

NB: The source for the finished product created using this tutorial can be found [here](http://github.com/pjjanak/chromecast-hello-world)

Yesterday I set out on a treck to start writing my first Chromecast app. What I found was a woefully incomplete amount
of [documentation](https://developers.google.com/cast/) via Google and several disaparate little projects that other
developers had worked on. While the task was not insurmountable because of this, I thought why not just write up a simple
Hello World with commentary to make any would be Chromecast Dev's life easier. So, that's what this is: a barebones
sender and receiver to help you understand how this all fits together. For now, I only have a Chrome sender. Soon I'll
probably work on the equivalent for Android. But, let's get started!

# Get Yourself Whitelisted #

I'm not going to go into too much detail with this, because Google did a pretty good job of outlining this process. You
can go [here](https://developers.google.com/cast/whitelisting) and just follow the instructions. It took me about 12 hours
to hear a response. One thing to mention is that to start Chromecast development you're going to need to host your receiver app
somewhere. Github has free pages hosting, but in the long run that would be a little limited. So, look into your
options and make the decision that works best for you right now.

An important note: The URL you give Google has to be the _exact_ location of your receiver app's index.html. If
you were to clone this repo and upload it to your server (say `http://mydomain.com`) then the receiver would be located
at `http://mydomain.com/receiver`. You would have to specify that as your URL when whitelisting. Alternatively, if you
just dumped the receiver app at the root directory of your domain, meaning it would found at `http://mydomain.com`, then
you would supply that URL. The way the process works now does not have any support for subdirectories. So you have to
make sure you're specific. Also, if you rename the receiver app from `index.html` to anything else, you'll have to 
specify that in the URL (e.g. `http://mydomain.com/receiver/sweetapp.html`).

Also, while you're waiting for this you might as well follow the second set of steps on that page and whitelist `localhost`
in the Chromecast extension.

# I'm Whitelisted...now What? #

Now the fun part! These apps aren't particularly difficult once you get the hang of it, so don't sweat it if you were
a little confused from Google's documentation. Right now, it's not very good!

To start off with, we're going to go ahead and create our receiver app.

## The Receiver ##

For both the receiver and the sender app I use a little bit of [jQuery](http://jquery.com/). I'm just going on the
assumption that you understand it, as well as basic HTML, CSS, and Javascript. If not, you may want to start
researching and learning those first. Having that base knowledge is important for being able to create useful
Chromecast apps.

So, one thing to remember is that all Receiver applications are written in HTML5/CSS/Javascript. For some, this might seem
awesome, and for others it might seem limited. Just remember what Chromecast is supposed to be: a media digester. You can
still do a lot within these confines, though, so don't get discouraged if you have big ideas.

Since our receiver is so tiny I'm just going to go ahead and show you the whole source and then make comments:

    <html>
        <head>
            <link rel="stylesheet" type="text/css" href="../css/receiver.css" />
        </head>
    <body>
        <div class="messages">
            <h1>Waiting for Messages...</h1>
        </div>
    </body>
    <script src="http://code.jquery.com/jquery-2.0.3.min.js"></script>
    <script src="https://www.gstatic.com/cast/js/receiver/1.0/cast_receiver.js"></script>
    <script src="messageTypes.js"></script>
    <script>
        $(function() {
            var receiver = new cast.receiver.Receiver('*** YOUR APP ID ****', ['*** YOUR NAMESPACE ***']),
                channelHandler = new cast.receiver.ChannelHandler('*** YOUR NAMESPACE ***'),
                $messages = $('.messages');
    		
            channelHandler.addChannelFactory(
                receiver.createChannelFactory('*** YOUR NAMESPACE ***'));
                
            receiver.start();
            
            channelHandler.addEventListener(cast.receiver.Channel.EventType.MESSAGE, onMessage.bind(this));
            
            function onMessage(event) {
                $messages.html(event.message.type);
            }
        });
    </script>
    </html>
        
Not so bad, right? Let's break it down. The HTML chunk should be pretty familiar to you:

    <html>
        <head>
            <link rel="stylesheet" type="text/css" href="../css/receiver.css" />
        </head>
        <body>
            <div class="messages">
                <h1>Waiting for Messages...</h1>
            </div>
        </body>
    ...
        
This just sets up where we're going to actually be displaying our content. I did some simple css too. You can look at the
finished code on Github if you're intersted.

Next comes the scripts. The first just gets us jQuery. The second is what actually gets us access to the Receiver API, so
make sure you don't leave it out! Otherwise, your app isn't going to work at all.

And then we have our script. Right away you start to see some garbage that mentions `cast`. As you might have guessed, this
is our first Receiver API call:

    ...
    var receiver = new cast.receiver.Receiver('*** YOUR APP ID ****', ['*** YOUR NAMESPACE ***']),
    ...

This creates a new `Receiver` object. This is what we'll be using to facilitate ways of communicating with the client and device.
Oh, and remember when we whitelisted our device? The e-mail you got should have contained your Application ID. That needs
to replace `*** YOUR APP ID ***`. As for your namespace, it can be anything you want, but should probably be related to
the application you're creating. For this tutorial we can use something like `HelloWorld`. Make sure it doesn't
clash with other namespaces, though, as this is how your Receiver will know what type of messages to listen to
and how your Sender will know what type of messages to send. More on that later...or right now:

    ...
    channelHandler = new cast.receiver.ChannelHandler('*** YOUR NAMESPACE ***'),
    ...
        
This little guy, as you might have guessed, is what handles our `Channels`. A `Channel` in this context is used to send
and receive simple JSON messages to and from the Sender application. The `ChannelHandler` allows us to hook into the events that
occur on the various `Channels` that get created as the application is being used. It also delegates to
a `ChannelFactory` (which you'll see a little later) to create new `Channels`. Again,  note the namespace. Make sure
you use the same one that you defined earlier on in the `Receiver`. This tells our `ChannelHandler` to only worry about
messages coming from that namespace.

The next line is just there to keep track of our messages div so we can swap out the text later. Following that we create our
`ChannelFactory`:

    ...
    channelHandler.addChannelFactory(
        receiver.createChannelFactory('*** YOUR NAMESPACE ***'));
    ...
        
That namespace again! Just fill it in like before. Here we're adding to our `ChannelHandler` a `ChannelFactory`. Like I said
eariler, this is how `Channels` actually get created. We use the `Receiver` to create the `ChannelFactory`.

Great! We've got our `Receiver` and our `ChannelHandler` all set up! Does it work yet? Just a little bit more! First we need
to tell the receiver to start listening to the world:

    ...
    receiver.start();
    ...
        
Then we make sure our `ChannelHandler` is listening for `MesssageEvents`:

    ...
    channelHandler.addEventListener(cast.receiver.Channel.EventType.MESSAGE, onMessage.bind(this));
    ...
        
The second parameter, as you should know from your Javascript training, just delegates the `onMessage` function as our callback
when we hear a `MessageEvent` and binds `this` as our context. `Channels` do have other types of events. We can also listen
for when the `Channel` is opened, closed, or when it has an error. For this tutorial we'll just stick with messages, though.

Last, we actually do something with the `MessageEvent`:

    ...
    function onMessage(event) {
        $messages.html(event.message.type);
    }
    ...
        
These `MessageEvents` have several parameters, but the thing that actually gets sent from the Sender application is the
`message` object, which is generally going to be represented as JSON. You'll see in the next section that we basically
define that JSON to look however we want. So there you go. Your first Receiver app! Now let's make a Sender to do
something with it!

## Sender ##

Before we get started writing the sender app, it'd be good to mention that you'll probably want some way of running a
web server on your local machine. Without it you can't really test this. For pretty much any platform,
I suggest [Mongoose](https://code.google.com/p/mongoose/). It's a stupid easy web server. You just drop the executable
into any directory you want to host, and launch it. You'll be able to access the folder from `localhost:8080`.

Now let's whip up a Sender so our Receiver can actually do something! This one is a little longer, so I won't paste in the whole
source. See my Github page for that.

First we have our HTML portion:

    <html data-cast-api-enabled="true">
    <head>
        <title>Hello World Chrome Sender</title>
        <link rel="stylesheet" type="text/css" href="../css/sender.css" />
    </head>
    <body>
        <div class="receiver-div">
            <h3>Choose A Receiver</h3>
            <ul class="receiver-list">
                <li>Looking for receivers...</li>
            </ul>
        </div>
        <button class="kill" disabled>Kill the Connection</button>
    </body>
    <script src="http://code.jquery.com/jquery-2.0.3.min.js"></script>
    <script src="http://underscorejs.org/underscore-min.js"></script>
    ...

Something you might notice is that we aren't importing any scripts for the Sender API. That's because this is handled by 
two things. You should remember way back when we were whitelisting our receiver that we also whitelisted `localhost`
in the Chromecast Extension for Chrome. This basically tells the extension that on the `localhost` domain we are allowing
the extension to inject the Sender API into any page. However, it won't just inject it all willy nilly. Our sender pages
require this line:

    <html data-cast-api-enabled="true">
    ...

to tell the extension that we want the API injected on this page. Make sure you don't forget it!

Everything else in the HTML section should be pretty clear. We have a container which will list out all the receivers that
get found and a button to disconnect from the receiver. We import jQuery and [Underscore](http://underscorejs.org)...sorry
I snuck that one in there. If you don't know about Underscore, you should check it out. It's a really great little
'functional' library for Javascript. I only use it for one thing here and I'll describe what's happening when we get there.

I've also done a little styling for this page, which you can look at on the Github. Moving right along, we'll jump into our
script. We declare our variables and then add some odd looking listener to our `window` object:

    var cast_api,
        cv_activity,
        receiverList,
        $killSwitch = $('.kill');
        
        window.addEventListener('message', function(event) {
            if (event.source === window && event.data &&
                    event.data.source === 'CastApi' &&
                    event.data.event === 'Hello') {
                initializeApi();
            }
        });
    ...
    
The event listener isn't too crazy. One of the consequences of the way the Sender API gets injected into our pages is that
we don't really have any control over when it happens. But, the Google Devs were smart and gave us a way of finding out
exactly when we can start interacting with it. After the Sender API is loaded, it emits a `MessageEvent` to tell us so.
So, we listen for this event. The if block checks to make sure the contents of the `MessageEvent` match what we're looking
for. As per Google's docs, this event looks like this:

    {
        source: 'CastApi',
        event: 'Hello',
        api_version: [x, y]
    }

The version number basically lets our Sender decide if it's compatible with this version of the API. Besides that, we
just make sure that the data in the `MessageEvent` matches the source and event described. Then we initialize our API:

    ...
    initializeApi = function() {
        if (!cast_api) {
            cast_api = new cast.Api();
            cast_api.addReceiverListener('*** YOUR APP ID ****', onReceiverList);
        }
    };
    ...

I've added a check in here which makes sure we aren't calling this if we all ready have an API. So, we create a new `CastApi`
object and attach a `ReceiverListener` to it. This will simply tell the `CastApi` that there are receivers which are listening
for our...what's that? Oh our App Id!

Bet you were wondering when that'd show up. In case you hadn't figured it out by now, the way this whole App Id business 
works is Google registers your device to listen for your App Id when you whitelist it. That App Id is tied to 
a particular website (the one you supplied when you whitelisted your device). This is how the Receiver knows 
which URL to hit when it gets a launch request...but now we're getting ahead of ourselves.

The second parameter to `addReceiverListener` is what callback to use when we get a list of valid receivers. Which brings
us to...

    ...
    onReceiverList = function(list) {
        if (list.length > 0) {
            receiverList = list;
            $('.receiver-list').empty();
            receiverList.forEach(function(receiver) {
                $listItem = $('<li><a href="#" data-id="' + receiver.id + '">' + receiver.name + '</a></li>');
                $listItem.on('click', receiverClicked);
                $('.receiver-list').append($listItem);
            });
        }
    };
    ...
    
According to Google's documentation the `ReceiverListener` fires at least once, regardless of if it's found anything, once
you register it. So, we start with a check to make sure we actually have receivers. Then we just take that list and
dump it out in our receiver list element. We store the id of the receiver in a data attribute on the anchor tag so we can
reference it later. Then we attach click handlers to the list items:

    ...
    receiverClicked = function(e) {
        e.preventDefault();
        
        var $target = $(e.target),
            receiver = _.find(receiverList, function(receiver) {
                return receiver.id === $target.data('id');
            });
        
        doLaunch(receiver);
    };
    ...
    
This isn't too out there...all standard Javascript/jQuery. This is where I end up using Underscore: `_.find(receiverList...`.
It should be pretty easy to figure out what's going on there. Underscore has a find function that takes two parameters:
the list you want to try to find something in, and a function to execute on each item in the list. `find` works by looping
through the list until the supplied function returns true. In our case, this is when we have found the receiver whose
id matches that of the link we clicked. Simple enough. After we've found the right receiver, we launch our application
on it:

    ...
    doLaunch = function(receiver) {
        if (!cv_activity) {
            var request = new cast.LaunchRequest('*** YOUR APP ID ***', receiver);
            
            $killSwitch.prop('disabled', false);
            
            cast_api.launch(request, onLaunch);
        }
    };
    ...
    
First we check to make sure we don't already have an `Activity` launched on the receiver. While nothing bad would happen if
we tried to launch another activity before closing out the first, I figured I'd put that check in there. As long
as that passes, we use the Cast API and create a `LaunchRequest`. There's that App Id again. Like I said
before, this is how your receiver knows which URL to launch. We also pass in which receiver the `LaunchRequest` should
go to. We then enable our kill switch and finally send our `LaunchRequest` to the receiver. The second parameter of the
`launch` function is the callback function for when the launch is succesful.

At this point if you just added an empty `onLaunch` function, had your receiver hosted, and were running a server on your
local machine, you'd be able to open up this page in your browser, click on the link to your receiver, and your app should
launch. Exciting, right? We'll just add a couple more things to show off some other interactions with the receiver. First,
that `onLaunch` function:

    ...
    onLaunch = function(activity) {
        if (activity.status === 'running') {
            cv_activity = activity;
            
            cast_api.sendMessage(cv_activity.activityId, '*** YOUR NAMESPACE ***', {type: 'HelloWorld'});
        }
    };
    ...
    
Once our application gets launched on the receiver, why don't we say something to it? First we check the `ActivityStatus`
that we got back from the `launch` function from before is in the running state. You can think of the `ActivityStatus`
as representing what our connection to the receiver is doing. It also contains the `id` of the `Activity`, which we use
to communicate with the recevier, as you can see in the next line.

Here we use the `Cast API` to send a message to our receiver. This function is one of two types of interactions you
can have with your receiver, and as far as I can tell, it isn't even documented! The other deals with `MediaRequests`,
but that's getting out of scope of this tutorial.

The first parameter to the `sendMessage`, again, is that `Activity` id. The second parameter is your namespace. If you
think way back to the receiver section of the tutorial, you'll remember that this is how the receiver knows which
messages to actually listen to. If you defined your namespace as `HelloWorld` use that here. Otherwise, just use
whatever you had before. Last we actually have the object which represents our message. You can literally put
whatever you want in here; it's just a freeform JSON object. For now we'll just include a `type` attribute and 
say this is a `HelloWorld` type message.

So, now if you reloaded your sender application and clicked on your receiver link you should see the placeholder text
'Waiting for messages...' change to 'HelloWorld' on your TV. Hey! Look at that! We made our TV do something!

Just one more thing to do and then the tutorial is over. Hooray! Let's add a click handler that will actually end
our session with the receiver:

    ...
    $killSwitch.on('click', function() {
        cast_api.stopActivity(cv_activity.activityId, function(){
            cv_activity = null;
        
            $killSwitch.prop('disabled', true);
        });
    });
    ...
    
This one is pretty straight forward. When we click on the kill switch, we call `stopActivity` on the `Cast API`. This does
exactly what it sounds like. We pass it the `Activity` id from our active activity and it tells the receiver we're done.
The second parameter is just a callback function for when the receiver tells us it successfully closed the app. When
that's done we null out our activity, so we can start a new one if we want, and disable the kill switch.

That's it! If you reload your sender you should now be able to

1.  Launch your activity on your receiver
2.  Observe the text on your TV changing
3.  Close out the application
4.  Rinse and repeat

Look at that! Your first end to end Chromecast app! Go forth and be merry and show all your friends that you can control
your TV with a web browser...but maybe do something a little more interesting with the app first. The sky is the limit now!

# Closing Thoughts #

I hope this has been useful for you. But this is only the tip of the iceberg...well maybe not iceberg. This device certainly
doesn't do everything, but it isn't supposed to. Really the depth from this device is going to be in the interesting
HTML5 apps people make that work with this thing. The interaction with the device itself, as you can see, is pretty simple.
It's just figuring out how to use the interaction model to make cool, useful apps.

However, this tutorial does not cover the other half of development on the Chromecast: media. There is a whole section of 
the API for the sender and receiver which covers media interaction, from playing/pausing to volume control. The same
principles we've talked about here apply, but the functions and objects you use are a little different in some places.
Perhaps in a future tutorial I'll cover that stuff. For now you should be able to muddle through the documentation
Google has provided.

Thanks for reading!
