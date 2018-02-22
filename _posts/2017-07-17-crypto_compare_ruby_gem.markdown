---
layout: post
title:      "Crypto Compare: A Ruby Gem to Track Cryptocurrency Valuations"
date:       2017-07-17 12:12:43 -0400
permalink:  crypto_compare_ruby_gem
---

![](http://i.imgur.com/170Govo.jpg)
> Which is more real? The currency on or off screen?

The aggregate total of all coins and bank notes (i.e. cash) in the world according to [sources](https://www.youtube.com/watch?v=w2tKg3E53DM) in 2013 exceeded 5 trillion U.S. dollars. This is referred to by economists as 'M0'. However, this is less than 10% of all of the money in the world. The rest is created through banks, from the central banks down to commercial banks, via a system called [fractional reserve banking](https://en.wikipedia.org/wiki/Fractional-reserve_banking), which is how banks increase their money supply to lend out to borrowers. Greatly simplified, it means that for every deposit made to banks (referred to in-part as 'M2'), so long as the banks keep a fraction of that deposit in their savings, for lending purposes they can electronically update their records with up to 90% more than that initial deposit. This is referred to as 'M1'. Out of the three, only M0, or 10%, is physical cash.  This means that most of the money in the world economy, around 90%, is purely digital. What people commonly think of as money, paper bills and metal coins, is actually just a metaphor. Presently, most of the world's money supply is simply stored as records in databases in financial computer systems worldwide. 

In this modern reality of an increasingly cashless world, and on the heels of the 2008 U.S. recession, an unknown programmer (or maybe group of programmers) under the alias of Satoshi Nakamoto released an open-source software called Bitcoin, which was the next logical step in this trend: a bankless, *fully* digital currency, or, as it became known, a cryptocurrency. Unbeknownst to the world then, Bitcoin, and the underlying peer-to-peer cryptographic technology powering it, the Blockchain, would reach a multi billion dollar market cap and spawn dozens of so-called alternative coins, or "altcoins," which all leverage the same underlying technology. It's beyond the scope of the blog post to go into detail about what a Blockchain actually is, but it's a pretty big deal (basically facilitating global financial exchange without a middle man).

After Bitcoin, a multitude of altcoins arrived to try and solve different problems in finance and carve out niches for themselves. One such coin, Ether, powers a platform called [Ethereum](https://www.ethereum.org/), which has all of what the Bitcoin network offers, with an added general programming language built on top of their Blockchain. This means that complex logic can be programmed on top of the Ethereum network, potentially leveraging the network to not only facilitate simple transactions, but to host and enable *entire* decentralized software applications. Then there's [Grantcoin](http://www.grantcoin.org/), a cryptocurrency that's sidestepping governments and attempting to self fuel and distribute a global digital basic income fund.

The above should give you an idea as to why there's a lot of interest in this tech, it's disruptive nature and what it can potentially mean for the future of finance. I've even invested in some crypto myself. 

Finding information on the crypto I'm currently holding however has been pretty cumbersome. Luckily, aggregator websites have sprung up that hold multiple cryptocurrency price valuations all on a single webpage. One such website is Coinmarketcap.com, which has (from what I can tell) all cryptocurrencies indexed, ordered from highest to lowest market cap, with various information in USD. Great! However, I'd still need to open up a web browser, scroll down a lengthy list or do a manual search for the cryptocurrency I want out of hundreds of possible choices. What's needed is a way to organize the growing cryptocurrency data into a readable concise format. 

![](http://i.imgur.com/VxI4HdI.png?1)
> Coinmarketcap.com's homepage, listing all cryptocurrencies by market cap

What if we could access all of this information directly in our command line without ever opening a browser? Maybe with a bit of Ruby cleverness I could make it happen through a simple Command Line Interface (CLI), abstracting away all of the superfluity and instead just display the direct information that we want. Except not everybody lives in America. A useful feature could also be currency conversion, where information on specific cryptocurrency valuations could be expressed to the user in any major fiat currency of the user's choosing, not just USD.

The domain model for this could be looked at like a foreign currency exchange centre. The main object classes could be:
- A **Scraper** class to acquire the information from the target website and pass that onto the other objects for use
- A **CryptoCurrency** class to represent each instance of a cryptocurrency, with attributes representing relevant characteristics 
- A **Money** class to represent instances of fiat currency amounts and exchange rate logic
- A **CLI** class that acts as a teller, handling user input and creating an interface to manipulate and display the data to users

First task: scraping the information we need from the website. Luckily, a handy Ruby gem called 'Nokogiri' takes care of the scraping. Combined with Ruby's built-in 'open-uri,' we can get the page information. This task is facilitated by the aforementioned Scraper class. Because the page document may need to be used by other methods, let's put it in an instance variable. And as the app only scrapes from one website and each Scraper instance will use the same site data, we can automatically initialize each Scraper instance with the instance variable @page pointing to the Nokogiri doc containing the scraped site information. 

![](http://i.imgur.com/Z8ZDrUr.png?1)

An issue with cryptocurrency websites is the sheer volume of information packed on the page, which sometimes obfuscates the information the user wants. What if specific coins could be made searchable on the CLI, and show only the user's selected coin info? That sounds like a good idea, but it might also be nice to have a *limited* selection of coins presented to the user in a list-view, perhaps of the 10 most popular (by market cap) cryptocurrencies, and leave it up to the user to pick from this list or enter their own more niche coin search. Let's scrape the Nokogiri doc for the names of the top 10 cryptocurrencies on the website.

Dev tools tells us that the CSS selector for each cryptocurrency name on the page is the text inside the "a" tag within the "td" tag with a class of "currency-name." We can use this with Nokogiri to scrape the relevant information. However, if there's a space in the name, let's substitute it (using gsub) for a dash to match the page's name formatting for id tags (this is important for scraping purposes later on).

Let's create a method #get_list, that scrapes and pushes the top 10 cryptocurrency names into an array. 

![](http://i.imgur.com/aHhThcS.png?1)

Sandwich code can be avoided via a collect enumerator instead, while keeping track of the index numbers of the items collected to only collect the names of the first 10 cryptocurrencies as well as delete all of the nil indexes.

![](http://i.imgur.com/ez3xF8M.png?1)

Great, now we have a list of the top 10 crypto currencies by name. However, at this point, each index simply contains a string that knows nothing about itself. They're not yet *actual* CryptoCurrency objects. 

Let's make a hash of attributes that represents information that we want our CryptoCurrency object to know about itself, and ultimately, the relevant information that we want to display to the user. Speaking for myself, the main information relevant to me as a trader would be:
- **Name:** The cryptocurrency of interest's name 
- **Price:** The current price of the cryptocurrency of interest in whatever fiat currency is relevant to me (to see if I'm gaining or losing money on my investment)
- **Market cap:** What is the current total aggregate value of this currency? 
- **Circulating supply:** How much is currently circulating? 
- **Daily percent change:** In the last 24 hours, did the currency valuation go up or down, and by how much? 

Taken as a whole, this data gives a fairly top level, concise impression of how a crypto currency is presently doing. Let's call our hash 'data,' and the method to procure said data #get_attributes. Because we want to carry out procedures with this data, let's make the method's return value the hash. 

Using dev tools, we find that the attributes for specific cryptocurrencies is within the "id" tags of ids named 'id-that-cryptocurrency's-name,' the name of which we can represent as an argument 'crypto_currency' that we can pass into the method. We also get rid of or minimize odd whitespace with gsub.

![](http://i.imgur.com/asXeU8v.png?1)

So we can now scrape from our source page, get a list of the top 10 cryptocurrency names, and have a way to scrape the relevant attributes about specific cryptocurrencies if we pass in that crypto currency's name. 

We can now take this information to instantiate actual CryptoCurrency objects that know about themselves.

Within our CryptoCurrency class, let's create attr_accessors for all of the relevant data that we scraped with our Scraper's #get_attributes method. Recall that our #get_attributes method returns a hash with the attribute names and attribute information as key value pairs. Because the names of the hash's keys match the names of our CryptoCurrency class's attributes, we can use the entire hash as an argument in a method that mass assigns data to each new CryptoCurrency instance's attributes. 

While we're at it, let's also create a class variable called @@all to store each new CryptoCurrency object that we create in an array, as well as enable the pushing of new instances of cryptocurrencies to this array if it isn't already present in the collection (we don't want repeat cryptocurrencies).

![](http://i.imgur.com/s20AXI9.png?1)

Great! Now by passing in the return value of the Scraper class's #get_attributes method (the data hash) to our CryptoCurrency class's #initialize callback method, we can instantiate a new CryptoCurrency object with appropriate data for our chosen attributes. Seeing as we have a method to scrape the attributes of a specific cryptocurrency by passing in it's name, and a way to use those attributes to instantiate CryptoCurrency objects that know about themselves, we can now use these methods in tandem to create new CryptoCurrency objects from each name listed in our top 10 list by iterating through that list with the just discussed #initialize method.

![](http://i.imgur.com/6oz2DS6.png?1)

Now we can instantiate any cryptocurrency we want as an instance of the CryptoCurrency class, with its price set in USD. But what about the users that aren't Americans? Luckily, we don't have to reinvent the wheel, as a Gem called 'Ruby Money' can do the heavy lifting for us. 

Ruby Money is a library that has a Money class to represent information about a certain *amount* of money, including what type of currency it is, and what its value is. It also has a Currency class to encapsulate information about a particular *type* of fiat currency. Note that Ruby Money takes money amounts in cents, thus any amount in USD must be multiplied by 100 to be represented accurately.  

![](http://i.imgur.com/M9kxyYW.png?1)

Finally, converting money amounts from one currency to another is performed by a Bank object, which has logic comparing Money objects based on their Currency type. You can add different rates via a method called #add_rate so the Bank object knows what the exchange rate is between the different Currency objects. 

Google Currency, a library that extends Ruby Money, gives us access to the exchange rates for all major fiat currencies represented by their global ISO codes (e.g. USD, CAD, GBP), based on Google's online listings scraped from [Google Finance Converter](https://www.google.com/finance/converter). This saves us the effort of researching and creating the rates from scratch. Ruby Money's #exchange_to method allows us to convert currency valuations like so:

![](http://i.imgur.com/fSwtzJL.png?1)

Now that we have all of the objects, methods and gems needed for our application, we can program the app logic in a CLI object, which passes around data from all of our other domain objects in order to display the relevant information to the user. 

Greatly simplified and presented without its full context, the basic logic of the CLI class, using the price attribute as an example, is that we can present the user with a list of the top 10 cryptocurrencies by reading the CryptoCurrency class's @@all variable, and depending on the user's choice of crypto, we can convert the scraped in-USD crypto currency price to the user's choice of fiat by instantiating that amount as a new "USD" Money object, and carry out the conversion using Ruby Money. We then display this information back to the user. The logic is similar for the other attributes.

![](http://i.imgur.com/DHf85ac.png?1)
		
After much refactoring and testing, a beta version of the CLI application was packed into a Gem and published at RubyGems.org for community use, aptly named [Crypto Compare](https://rubygems.org/gems/crypto_compare). You can check out the source code, submit bugs, fork, and improve the Gem there. 

Time will tell if this tech pans out, but now when I want to check my crypto investments, I just open up the terminal and within a few keystrokes, I have my data. The time this saves is, well, priceless.

You can watch a walkthrough of the CLI in action below:

<div style="position:relative;height:0;padding-bottom:56.25%"><iframe src="https://www.youtube.com/embed/ccNV0bpre_Y?ecver=2" style="position:absolute;width:100%;height:100%;left:0" width="640" height="360" frameborder="0" allowfullscreen></iframe></div>

