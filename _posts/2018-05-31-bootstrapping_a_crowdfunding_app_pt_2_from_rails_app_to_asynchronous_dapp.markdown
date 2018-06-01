---
layout: post
title:      "Bootstrapping a CrowdFunding App Pt. 2: From Rails App to Asynchronous dApp"
date:       2018-05-31 21:59:40 -0400
permalink:  bootstrapping_a_crowdfunding_app_pt_2_from_rails_app_to_asynchronous_dapp
---


In [Part 1](http://fullstackfollies.com/bootstrapping_a_crowdfunding_app_on_rails), I talked about creating the backbone CRUD functionality of ThingFunder (creative name, I know) in Rails. I revisited the project recently as a learning exercise to try out some new tech. In this post, we'll take a look at the crowdfunding problems that I'm interested in trying to solve, go over how I used Web3.js (via [Metamask](http://metamask.io)) and [Solidity](http://solidity.readthedocs.io) to add Ethereum smart contract functionality, and look into bettering the user experience by using AJAX/jQuery and the [Handlebars](https://handlebarsjs.com/) templating engine to make the app load data asynchronously.

To prefice, I love the concept of crowdfunding, of democratizing product creation and allowing not private venture capitalists or investment funds, but instead the consumers themselves, to be the ones to directly empower product creators. There are some issues though...

**The Accountability Problem**

Admittedly, I've been burned a few times backing some of these projects. There have been awesome looking projects who's makers, after raising all of their requested funds from the community, [disappeared](https://www.kickstarter.com/projects/22285033/mansion-lord-a-murder-mystery-rpg-business-sim/comments). There have been other projects that deliver, but at a [fraction of the quality](https://www.indiegogo.com/projects/kreyos-the-only-smartwatch-with-voice-gesture-control#/comments) of their promised goods. Since projects don't have a defined deadline for when they need to be delivered to the backers, there is yet a third type of burn: a funded project that extends its development time, with very sparse to no updates from their creators, [sometimes by years and years](https://www.kickstarter.com/projects/omocat/omori/comments), therefore putting into question whether the makers are even still working on the darn thing, or whether your money's gone forever into who knows where. 

Backers rely on the good faith of creators to deliver on their promises. There are no legal ramifications for not updating your backers, or, really, delivering on the project at all. Therefore, there isn't any incentive (besides, maybe preserving your reputation and an appeal to general good nature) in place for the creator to deliver fast and transparent updates to their backers once they've received all of their funding. This creates a frustrating user experience for backers, creating distrust, and also allowing incompetent makers (or worse, scammers) easy access to funds with lax accountability for delivering.

Not to put too much of a damper on things though. For every dud, there are a ton of awesome projects that do get funded and deliver on the goods. However, one or two bad projects are still one or two too many. 

**Ethereum and Programmable Money**

[Ethereum](https://ethereum.org/) is a blockchain app platform. It allows developers to create applications, called dApps (short for decentralized applications), whose logic is executed, and data is persisted, on a decentralized blockchain as opposed to centralized servers. The scripts are executed by the Ethereum Virtual Machine, which exists on top of the Ethereum blockchain. It allows developers to program logic into monetary dealings in the form of "smart contracts," scripts that send money (in the form of [cryptocurrency](http://fullstackfollies.com/crypto_compare_ruby_gem) on the Ethereum blockchain) *directly* between parties (from wallet to wallet on the Ethereum blockchain with no bank or payment processor middleman), only when certain criteria are met. I thought this could be applied in interesting ways to the crowdfunding accountability problem. 

Instead of sending the entirety of a backer's pledge to the project maker right away, imagine a system where a smart contract is utilized to allow backers to send the funds in smaller increments. The first installment is sent immediately upon choosing to back a project, so the maker has starting funds to begin work on the project. However, each subsequent increment is sent only when the maker provides some sort of concrete update as to the progress of the project that requires additional funds. This would incentivize project makers to be more transparent with backers, because they need to win their backers' good faith to receive the rest of the funding in order to bring the project to completion.

Challenge accepted! To bootstrap our solution, I needed to learn how to do a few things:

1. Write the smart contract logic
2. Integrate our web app with the Ethereum blockchain
3. Make the app asynchronous and instant 

**1. Writing the smart contract logic**

[Solidity](http://solidity.readthedocs.io) is the programming language for writing smart contracts on the Ethereum virtual machine. It's high-level, though statically typed, and a little more verbose than JavaScript. Besides a few small differences, the syntax by-and-large mirrors JavaScript in order to be as small of a learning curve as possible for most web developers. One of the most popular (and efficient) IDEs for writing and debugging Ethereum smart contracts is called [Remix](http://remix.ethereum.org) and runs directly on your internet browser of choice. 

A smart contract in Solidity is similar in concept to a class or object in Ruby. Each Solidity smart contract begins with the declaration of what version of Solidity your contract expects to use, and the contract declaration starting with the contract name in all caps (A JS or Ruby parallel would be a class declaration), followed by variable and constant declaration. Notice that each variable requires you to also prefice it with its type.

```
pragma solidity ^0.4.12;

contract ThingFunder {
	address public backer;
	address public maker; 
	uint divisor;
	uint timesRefunded;
	uint timesBacked;
```

What follows is the contract constructor, which is a function that instantiates all of the constants and variables that the contract needs to know about. In our case, we also need to add one argument to the constructor function: the maker's wallet address. As a Solidity convention, user inputted variables begin with an underscore (e.g. _maker). In our case, we also want to automatically send the first increment of the backer's pledge to the maker upon contract creation. Therefore, the constructor sets a variable called 'divisor', which determines how much each individual increment will be (to make things easy for this project, I just elected to divide the backer's total pledge by 4 to create equal 25% increments), and sends the first increment of this amount to the maker's wallet. 

```
	constructor (address _maker) payable {
		backer = msg.sender;
		maker = _maker;
		divisor = address(this).balance/uint(4);
		maker.transfer(divisor);
	}
```
	
We'd also need to write all of the smart contract functions within the contract declaration. We won't go through the entirety of the contract, but, along with some setters, getters and refund logic, the major function that I needed to create was payoutToMaker(), which sends the next increment of a backer's pledge to the maker. This should be envoked by backers every time a maker provides proof that progress has been made on their project.

```
	function payoutToMaker() {
		if (msg.sender == backer) {
			maker.transfer(divisor);
		}
	}
	
}
```

It's beyond the scope of this article to explain the Remix IDE, but with it, you can compile your contract directly in the browser and see any errors (the Compile tab), as well as test any contract functions (the Run tab). When everything is working, you can deploy the contract onto the blockchain directly from Remix.

![](http://imgur.com/jF9H1gf.png)
> Remix IDE

**2. Integrating our web app with the Ethereum blockchain**

A Chrome extension called [Metamask](http://metamask.io) turns Chrome into an Ethereum browser by injecting the Web3.js JavaScript library into the browser, thereby allowing websites to retrieve data from the blockchain and letting users manage their cryptocurrency and contracts via [JSON-RPC](http://en.wikipedia.org/wiki/JSON-RPC) . It allows developers to build out the smart contract functionality they want in JavaScript by interacting with various objects and functions built into the library. In our case, we'll use another helpful library called [EthJS](https://github.com/ethjs) for talking to the Web3 object and the blockchain. 

The first thing we need to do is check for this global Web3 object, which Metamask, if installed, injects into the browser. We need to write a conditional that, if the user does not have Metamask installed, instructs them to download the extension in order to use the app. We can build this out to be prettier, but for now, let's simply pop up a crude alert window. If Web3 is detected, we envoke the startApp function and pass in the Web3 object, and thus all of its functionality, into it. We trigger this on page load:

```
window.addEventListener('load', function() {
	if (typeof web3 !== 'undefined') {
		startApp(web3);
		console.log('loaded!');
	} else {
		// Warn the user that they need to get a web3 browser
		// Or prompt them to install MetaMask, maybe with a nice graphic. For now:
		alert('Please download MetaMask to use this dApp');
	}
})
```

To set up our app functionality, we use a few different EthJS modules, instantiating them into constants ('Eth' and 'EthContract') for reference throughout the app. We pass in the global Web3 object (injected by MetaMask) upon app start, getting the current provider (the blockchain that Web3 is currently talking to, which can be edited in the settings) and placing it, and all of its functionality, into a variable for reference in our code (which we've called 'eth').

```
const Eth = require('ethjs-query');
const EthContract = require('ethjs-contract');

function startApp(web3) {
	const eth = new Eth(web3.currentProvider);
	const contract = new EthContract(eth);

	initContracts(contract);
}
```

It's important to note that the Ethereum virtual machine cannot understand our high-level Solidity code. For it to run the functions we've written, our contract needs to be compiled into low-level bytecode and corresponding ABI (Application Binary Interface) and deployed to the blockchain. A useful framework for easy contract compilation and deployment is the sweetly named [Truffle](http://truffleframework.com/docs/). 

Afterwards, we need to set up an instance of EthContract with all of the functionality of our specific compiled contract. By passing in the compiled contract's abi as an argument (which we got from Truffle compilation), and instantiating it in a JS constant (in our case, named 'ThingFunder'), we now have access to all of the contract's functions in JS representation, which we can call in our app and pass to Metamask and the blockchain via the injected Web3.js functionality. 

```
const bytecode = [contract bytecode]

const abi = [contract abi]

const ThingFunder = contract(abi)
```

Now we're able to trigger contract functionality via jQuery event listeners in our app, and create ThingFunder instances from our constructor function! For each backer, these are placed in 'thingFunderInstance' variables. Below, we envoke the payoutToMaker() function of our contract, passing in the maker and backer addresses.

```
$('div.wrapper').on('click', '.nextInstallment', function(event) {
    event.preventDefault();
    address = $(this).attr('data-contract');
    thingFunderInstance.payoutToMaker({to: address, from: web3.eth.accounts[0]})
    .then(function (txHash) {
      $('div.rewards').html('<p>Sending transaction...</p>')
    }).catch(console.error);
});
```

becomes...

![](http://i.imgur.com/wnaRzOF.gif)

> Clicking the nextInstallment button launches MetaMask, which interfaces with the current blockchain provider via JSON RPC to send money via our deployed contract's payoutToMaker() function, asynchronously updating the DOM afterwards.

Now that our app has smart contract functionality, it's just a matter of passing data from our backend database to the smart contract, and updating our ORM with the new information coming from the blockchain. This was tricky because Web3 uses JavaScript and the data that we needed to pass our smart contract is stored in our Rails ActiveRecord ORM. In other words, our data is server-side and our functionality is client-side, and they have to play catch. **The client-side can't talk to the server-side very easily (they can't play back-and-forth catch with their data).** What I ended up doing was using controller actions to render data as JSON on page load and updating certain DOM elements' data attributes with that JSON. Now that the front-end knows what I need to pass into Web3, I was able to use jQuery to select those data attributes when certain click events were triggered and send that information over to my smart contract.

**3. Making the app asynchronous and instant.**

There's a lot of conditional logic in ThingFunder that changes how the DOM looks. Whether a project has met their funding, whether a project's deadline has come and gone, whether you've donated to a project already, whether you've commented on a project, and whether you've just deleted a comment - all of this affects what information is displayed to the user. Not only that, but this information changes frequently with user interaction. Now that our app has smart contract functionality, we need to make all of this new data, as it changes, show up on our pages in real-time without refreshes (Imagine if you needed to manually refresh the page each time to see whether an email, or payment, or, whatever else, was successfully sent - it would make the internet a lot more annoying and take much longer). 

Enter templating engines and Handlebars, a JavaScript library that makes asynchronous DOM changes easier and less messy. When updating the database via AJAX requests, we can already update the DOM by using jQuery and string interpolation of the new data into the DOM's markup. Something like this:

```
$.ajax({
	url: action,
	data: params,
 	dataType: “json”,
	method: “POST”
}).success(function(json) {
	html = “”
	html += “<li>” + json.name + “</li>”
	$(“ul.todo-list”).append(html)
})
```

This works well for small requests, but gets messy for bigger ones where we would need to manually build a huge string for this complicated code. Instead, we can use templates: pieces of markup with the dynamic data that we want to change asynchronously, abstracted into general variables. 

```
<ul class="todo-list">
    <li>this</li>
    <li>is</li>
    <li>HARD-CODED</li>
</ul>

<!-- vs. -->

<script id="sample-template" type="text/x-handlebars-template">
<li>{{uno}}</li>
<li>{{dos}}</li>
<li>{{tres}}</li>
</script>
```

Our templating engine, in this case, Handlebars, knows to replace each of those general variables (in the above example, 'first,' 'second,' and 'third') with the specific data that we feed into the template (this is called 'context' in Handlebars parlance), retrieved from our AJAX request (Handlebars also comes with a bunch of [built-in helpers](https://handlebarsjs.com/builtin_helpers.html), and the ability to make custom helpers, which is super useful).

```
$.ajax({
	url: action,
	data: params,
 	dataType: “json”,
	method: “POST”
}).success(function(json) {

  let source = $(“#sample-template”).html()
	let template = Handlebars.compile(source);
	
	let html = template(json)

	$(“ul.todo-list”).append(html)
})
```
> In this case, we take the returned JSON data and instead of manually interpolating it into a string, we feed the JSON into our Handlebars template  (in this case, called #sample-template) to automatically guarantee the correct markup based on our provided context (our JSON).

I created multiple Handlebars templates for each section of my markup that I needed dynamically changed every time data was modified on my backend. However, the data I received from my Rails server was not yet in JSON, they were Ruby objects. I needed to modify my controllers to deliver JSON data to the browser when requests specified that format in order for my JavaScript to be able do anything with it. Luckily, Rails provides the to_json method which takes our object and, well, converts it to JSON.

A simple example is the #create action of my Comments controller, which, with the to_json method, now distinguishes what type of response data it sends back to the browser depending on my request (html or JSON).

```
def create
    @commentable = Project.find(params[:comment][:commentable_id])
    @comment = @commentable.comments.new(comment_params)
		
    if @comment.save
        respond_to do |f|
            f.html {redirect_to project_path(@commentable), notice: 'Your comment was successfully posted!'}
            f.json {render :json => @comment.to_json(:include => {:user => {:only => :username}})}
        end 
    else
        redirect_to session.delete(:return_to), notice: "Your comment wasn't posted! Try again!"
    end
end
```
	
My AJAX post request, when triggered, can then create the comment and pass in the returned JSON data into my Handlebars comments template, updating the DOM dynamically.

```
$("form#new_comment").submit(function(e){
    e.preventDefault();
		
    Comment.templateSource = $("#comment-section-template").html();
    Comment.template = Handlebars.compile(Comment.templateSource);

    let $form = $(this);
    let action = $form.attr("action);
    let params = $form.serialize();
		
    $.ajax({
        url: action,
        data: params,
        dataType: "json",
        method: "POST"
    }).success(function(json){
        let comment = new Comment(json);
        let commentMarkup = comment.renderCommentMarkup();
        $('div.c-section').prepend(commentMarkup);
    })
})
```

becomes:

![](http://i.imgur.com/C9SQy1s.gif)
> Asynchronous comment creation and deletion

**Note:** One thing that is good to remember when working with asynchronous data is that event listeners won't work on dynamically loaded DOM elements. This caused a bug in my app because, when I dynamically load a new project's show page, most of the DOM is destroyed and updated to reflect the new project data, including the comment form. The click handler targeting the 'new_comment' form no longer works, because that DOM element no longer exists, and is instead replaced by a new, dynamically loaded 'new_comment' form.  

The solution is to not place the event listener on the element itself, but a static parent/container element. This parent element will still be there upon the asynchronous DOM change, and so the event listener will still be intact, and child elements (including the new comment form), will react to the listener.

```
$("div.wrapper").on("submit", 'form#new_comment', function(e){
	e.preventDefault();
		
	// etc...

})
```
> Target the static wrapper as opposed to the dynamically rendered new_comment form

Not bad for a few weeks' work. It's not done by any means, but I learned a lot in a short period of time, and I'm pretty happy with how the project has turned out for now. I'm going to do some more code clean up, then push it up for you guys to try. For now, check out a quick walkthrough of the added features below:

<iframe width="560" height="315" src="https://www.youtube.com/embed/CUj_MpX006A?rel=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

