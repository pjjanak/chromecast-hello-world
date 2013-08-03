# What's This? #

NB: The source for the finished product created using this tutorial can be found [here](http://github.com/pjjanak/chromecast-hello-world)

Yesterday I set out on a treck to start writing my first Chromecast app. What I found was a woefully incomplete amount
of documentation via Google and several disaparate little projects that other developers had worked on. While the
task was not insurmountable because of this, I thought why not just write up a simple Hello World with commentary
to make any would be Chromecast Dev's life easier. So, that's what this is: a barebones sender and receiver to help you
understand how this all fits together. For now, I only have a Chrome sender. Soon I'll probably work on the equivalent for
Android. But, let's get started!

# Get Yourself Whitelisted #

I'm not going to go into too much detail with this, because Google did a pretty good job of outlining this process. You
can go [here](https://developers.google.com/cast/whitelisting) and just follow the instructions. It took me about 12 hours
to hear a response. One thing to mention is that to start Chromecast development you're going to need to host your receiver app
somewhere. Github has free pages hosting, but in the long run that would be a little limited. So, look into your
options and make the decision that works best for you right now.

Also, while you're waiting for this you might as well follow the second set of steps on that page and whitelist `localhost`
in the Chromecast extension.

# I'm Whitelisted...now What? #

Now the fun part! These apps aren't particularly difficult once you get the hang of it, so don't sweat it if you were
a little confused from Google's documentation. Right now, it's not very good!

To start off with, we're going to go ahead and create our receiver app.

## The Receiver ##

For both the receiver and the sender app I use a little bit of jQuery. I'm just going on the assumptiong that you understand
it, as well as basic HTML, CSS, and Javascript. If not, you may want to start reseraching and learning those first. Having
that base knowledge is important for being able to create useful Chromecast apps.

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
        
Not so bad, right? Let's break it down. The html chunk should be pretty familiar to you:

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
you use the same on that you defined earlier on the `Receiver`. This tells our `ChannelHandler` to only worry about
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
        
The second paramter, as you should know from your Javascript training, just delegates the `onMessage` function as our callback
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
