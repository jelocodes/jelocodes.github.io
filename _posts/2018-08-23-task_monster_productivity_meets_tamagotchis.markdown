---
layout: post
title:      "Task Monster: Productivity Meets Tamagotchis "
date:       2018-08-23 23:22:06 -0400
permalink:  task_monster_productivity_meets_tamagotchis
---


When I was growing up (and you know, also now), I really liked virtual pets. I had keychains of digital pets that I took care of (see Tamagotchi), and games where I could raise and battle them (see Neopets, Digimon and Pokémon). The conceat has always been that these little critters prospered or perished by virtue of your interaction (or lack of interaction) with them. Give it plenty of food and love, and it showed affection to you and got stronger. Ignore it and it might ignore you, or even worse, leave for the digital pet cemetary. This gave the user a reason to keep going back to the software. It was a gamified feedback loop that kept the user hooked. 

Unrelatedly, lately, I've been trying out different note taking and calendar apps to help me streamline my productivity. Evernote and Google Keep were already go-tos, along with team-based apps like Basecamp for larger projects with other humans, but I decided to check out what else was out there. I didn't find much else, so I thought it would be fun to gamify the to-do process by building my own simple app. The idea was simple: mix gamification with productivity to make being productive more fun. And so, the idea for Task Monster was born! Who could resist being productive when doing so helps take care of a cute virtual pet? (The answer, by the way, is no one.) Oh, and what if each monster is truly uniquely yours, cryptographically tied to your unique account on the Ethereum blockchain? Productivity problems solved! Plus, it would give me a chance to use some nifty new technologies, so I decided to take a couple of weeks to hack it out!

Technologies Used:
* React and Redux (for lightning fast (virtual) DOM manipulation in a slick responsive front-end)
* Rails (for a tried-and-true database API) 
* Ethereum (for smart contract logic and blockchain storage)

Thanks to FullstackReact, Gitter and StackOverflow for helping along the way.

**Part 1: From the front to the back: React meets Rails**

The React app needs data persistence. I used Rails as my back-end API for this, which gives me access to the great ecosystem of libraries/gems that could bootstrap things like authentication and validation (e.g. Devise) for me. Only, I wouldn't need any views at all in my Rails app - just models and controllers; the things to architect my database and organize and feed the information to my React front-end. Luckily, the Rails team has this covered. We can start a bloat-free API-only Rails project like so:

`$ rails new task_monster --api`

This creates our new project directory, with a Gemfile and all of the subdirectories we've come to expect, just without the front-end stuff. In our command prompt, if we `cd` into the project directory, we can make a sub directory that will hold all of our client-facing (React) parts of the app. Using `create-react-app`, linked [here](https://github.com/facebook/create-react-app), which is Facebook's in-house implementation of a bootstrapped fresh React project, I was good to go and had both parts of my application ready to work on. I named my fresh React app 'client.'

![](https://i.imgur.com/MJwsdU1.png?1)
> The project directory

Notice that there's a Gemfile in the root Rails install and a package.json in the React client folder install. The client and server therefore specify their own dependencies independent of one another. They work together, but following this technique, for all intents and purposes, our front-end and back-end can be considered their own discrete apps and git repos (talk about separation of concerns!). 

There are two caveats with this set-up:

**a) Managing multiple processes (front-end and back-end) simultaneously**

**b) CSRF token access for user authentication and form sending**

**a)** We need two separate processes running simultaneously for our configuration to work. For example, we can start the client's server to launch on, say, localhost:3000, and likewise, the back-end Rails server can be launched on a separate port on localhost:3001. The client needs to communicate with our back-end server, and vice-versa, so in this configuration, the user's browser would load localhost:3000, accessing all of the static client assets from our front-facing server, which in turn fetches and posts new data to and from our database at localhost:3001 as needed. 

![](https://i.imgur.com/i0DVIjm.png)

[Foreman](https://github.com/ddollar/foreman) is a handy gem to run multiple processes at once for this purpose. We can declare a Procfile in our root directory with `touch Procfile` which lists the processes that Foreman is to manage. In our app's case, we want to be able to start and manage the web server and the api server simultaneously on different ports. 

Declaring...

`web: cd client && npm start
api: bundle exec rails s -p 3001`

...in our Procfile takes care of this. We can now start both servers with the command `foreman start -p 3000`, specifying that our web server runs on port 3000.

**b)** Rails uses CSRF tokens for POST requests. If we want to be able to do things like create new users, tasks, and monsters in our database, we probably (definitely) need this functionality.  This is taken care of automatically with Rails' form builders and action helpers, but since we're building our front-end with React, we have no such luck. Luckily, there is an open source JavaScript solution called JSON Web Token (or [JWT](https://jwt.io/)) which is an open standard that stores hashed tokens in the browser to verify sessions. 

The [JWT Gem](https://github.com/jwt/ruby-jwt) is a nifty Ruby gem that takes care of the encoding and decoding of the token, in our case, allowing the back-end to know that the sender of data is in fact the one currently signed into the app. 

After all of the above, and tinkering with a few [CORS permissions](https://til.hashrocket.com/posts/4d7f12b213-rails-5-api-and-cors) to allow same-origin requests, we can see a simple front-end form successfully creating a new Task object in our back-end and asynchronously populating the view with the returned data.

![](https://i.imgur.com/hrq4DR4.gif)
> So... beautiful. 

The front-end of the app is made up of multiple components, as stateless as possible, using Redux to maintain a single point of truth for all child components. The form components (SignInForm, SignOutForm, etc.) are an exception, as they don't need to be made aware of global state, and are instead controlled components, that use the native fetch API to send and receive data from our Rails back-end, based on the user's input updating the form component's own internal state.

One component, 'TaskModule,' acts as the main 'gatekeeper' and parent component, keeping track of our Redux dispatch actions and updating its state, and passing this down, along with any methods needed, to all of its child components as props. The child components then invoke these passed-down callback methods which respond to various synthetic event triggers, updating the parent component's (or Redux store's) state, therefore re-rendering any and all affected components. 

React Router is used for RESTful component routing, which makes routing very manageable all in one place.

**Part 2: Database Domain Model**

My database (Rails) would need to persist Users, Monsters, Tasks and TaskLists.

Users should be able to make many TaskLists (a.k.a. to-do lists), and each would in turn have many Tasks (a.k.a. to-do items) within them. Users could have one or many Monsters. Each TaskList should be tied to one Monster (but a Monster could be tied to multiple TaskLists), which either grows stronger or weaker depending on whether the User accomplishes his/her goals on time (depending on whether they meet the TaskLists' deadline). Users should be able to create and delete new TaskLists, and also do the same for individual Tasks within each TaskList. 

The model relationships are summarized as follows:

```
class Monster 
	belongs_to :user
	has_many :task_lists 
end

class TaskList 
	has_many :tasks
	belongs_to :user
	belongs_to :monster
end

class Task 
	belongs_to :task_list
end

class User 
	has_many :monsters
	has_many :task_lists
	has_many :tasks, through: :task_lists
end
```

**Part 3: Defining Monsters**

With those hurdles solved, comes the fun part: monster brainstorming. Luckily, I had some help, as my girlfriend is awesome at design, and is a fellow digital monster lover. We had fun mixing and matching different little critter ideas, which she created sweet vector art (and eventually gifs) for. For this simple MVP, we finalized two monsters for some variety (Though in a finished version, we'd ideally have many more available to collect).

![](https://i.imgur.com/MavEmEu.jpg =623x466)
> Monster sketches

The idea is that as tasks are completed, your monster gets stronger and levels up. I originally didn't know how far to take this concept. Would your monster have stats that would increase too? Would they have attacks? Evolved forms? Oh, what if you could battle other monsters and prove your productivity superiority??? Then I thought "Wait -- wasn't the whole point of this app to help you do your work?" In the end, I figured that all of that fluff, though cool, would actually be counter productive to, well, productivity. You can see the stats still in effect in the .gif below, but I decided to get rid of them in the end product in favour of simplicity. Your monster would have a single attribute, its level, which determined how strong it was, and, therefore, how productive you were being. Nothing else is necessary for incentivization. In fact, too many other features would probably get in the way of actually doing work and be more distracting than anything else. 

![](https://imgur.com/llVfK0B.gif)
> Monster cat hates Mondays.

**Part 4: Bells and Whistles**

A piece of technology is only as good as its user experience. Everything can work fine, but if it's tedious to use and unpleasant to look at, it's not going to make anyone's lives easier and it's not going to be useful. So our app needs a paint job. Luckily, a React UI framework called [Material-UI](https://material-ui.com/) is here to save the day. Based off Google's flat and minimal Material Design principles, these styles bring a touch of modernity to the app. With some choice colors (using our monster cat's pink/purple fur as the starting guideline) and Google Font imports, we're good to go.  

![](https://imgur.com/tyOqMCs.gif)
> Implementing Material Design.

The Progress Bar calculation is done by looking at the length of a passed-in 'tasks' prop, which is an array containing all of the particular to-do list's to-do items. Each individual task has a 'done' attribute, which is a bool that, when truthy, determines that task item's completion.  We filter the 'done' task items from the tasks prop array, and divide that by the total length of the array, multiplied by 100.

```
    let progress = {
      width: parseInt((((this.props.tasks.filter(task => task.done === true).length) / this.props.tasks.length) * 100),10) + "%"
    }
```

The Task Monster logo is typed in [Amatic SC](https://fonts.google.com/specimen/Amatic+SC), which I chose because it reminded me a bit of the font used in Maurice Sendak's [Where The Wild Things Are](https://images-na.ssl-images-amazon.com/images/I/51fCyTfSXRL.jpg/).

**Part 5: New Monster on the Block**

I thought it would be cool to persist the monsters that users create on the Ethereum blockchain. Each new monster is generated (albeit primitively right now) uniquely as a result of the user's Ethereum wallet address and the name of their to-do list, and lives on the Task Monster smart contract on the blockchain. So, your monster is cryptographically stored as belonging to you, and levels up as a result of your productivity - the being that it's kind of like a permanent, untamperable record of how productive you are (in cute digital monster form!). 

I used the [Remix IDE](https://remix.ethereum.org) to code and debug my smart contract, and then used [Truffle](https://truffleframework.com) to deploy it to the Ethereum blockchain (Kovan Test Chain). Client-side, I used the compiled abi and bytecode and fed it into the web3.js JavaScript libraries to create a client-side connection to the smart contract, accessible to users via the [Metamask](https://metamask.io) Chrome extension (you need to have Metamask installed to use Task Monster).

On the blockchain, the monsters exist as a struct, having a name, a gender, and are owned by a particular Ethereum address, as denoted in the Solidity code below. We create an array of the unique monster ids (unsigned 256 bit integers), and create a mapping of the monster ids to the actual monster struct objects for easy retrieval.

```
pragma solidity ^0.4.23;

contract TaskMonster {
    address owner;

    struct Monster {
        string name;
        address owner;
        string gender;
    }
    
    mapping (uint256 => Monster) monsters;
    uint256[] public monsterIds; 
```

Monsters on the blockchain are identified by unique ids (unsigned 256-bit integers) which are created using the unique combination of the users' Ethereum address and their particular to-do list name. The method newMonster() is called when the 'Get New Monster' button is clicked on the front-end, which passes in the monster's randomly rolled gender, the user's Ethereum address, and the combined Ethereum address/to-do list name. 

The smart contract's getId() function hashes the passed in combined string using the keccak256 hashing algorithm, returning an unsigned 256-bit integer, used as the monster's unique id. If this app is developed further, hopefully this generation process will become more complex with countless unique monsters able to be generated, but for now, with only two sample monsters in existance for our MVP, it's as easy as an even integer resulting in one monster (Schrodinger) and an odd integer resulting in the other (Leaflet).

The newMonster() method uses the getId() method and generates the new monster struct with our passed in attributes, then adds it to our array of monster ids for future retrieval:

```
   function getId(string _combined) public returns (uint256) {
        return uint256(keccak256(_combined));
    }
		
    function newMonster(string _gender, string _combined) public returns (string, string, address) {
        uint256 id;
        id = getId(_combined);
        if (id % 2 != 0) {
            monsters[id].name = "Schrodinger"; 
        } else {
            monsters[id].name = "Leaflet";
        }
        monsters[id].owner = msg.sender;
        monsters[id].gender = _gender;
        
        monsterIds.push(id) - 1;
        
        return getMonster(id);
    }
```
		
Computation on the blockchain is expensive, so we want to do most of the heavy lifting on the client side. Namely, randomly generating the monster's gender to pass into our smart contract, and providing our smart contract method the arguments it needs to generate the new monster (the combined string, and the users' Ethereum address, which is readily available in our browser window's web3.js object).

```
// In our React component
// below, randomly rolls either 1 or 0, with 1 being female and 0 being male
// and combines the user's Ethereum address with their to-do list name

let newMonsterGender = Math.floor(Math.random() * 2) === 1 ? "♂" : "♀";
let combined = window.web3.eth.accounts[0].concat(this.state.name)

// we then pass this into the compiled smart contract upon 'Get New Monster' button click

handleClick = (...arguments) => {
    taskMonsterInstance.newMonster(arguments.newMonsterGender, arguments.combined, {from: window.web3.eth.accounts[0]})
}
```

![](https://imgur.com/XsnkDEO.gif =100x20)
> It's Alive!

**In the end:**

I'm happy with my little prodictivity tracker MVP, which, by the way, I used to track my progress as I coded the app itself! If interested, fork the [git repo](https://github.com/jelocodes/task-monster-react) and make it better, or watch the demo of it all coming together below!:

<iframe width="560" height="315" src="https://www.youtube.com/embed/XPvhgOBo8jA?ecver=2" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

Now get some work done!
