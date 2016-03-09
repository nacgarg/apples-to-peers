# Peering
Everyone listens on the _apples to peers_ port, which is going to be a number. When someone joins, they join by the user inputting the hostname of someone currently running it, and it'll connect to that hostname. Then it'll ask that hostname for all the other peers, and connect to all those.

So basically when you join, you connect to everyone already on the network, and then you start listening for connections from people who might join after you

When a connection is opened, the first data that is sent is the nickname and public key. This data is sent in both directions.
Then, the side that was listening for the connections sends hostnames of all peers it knows of to the side that instigated the connection.
# Random Dealing
1. Everyone generates a RSA private key.
3. Everyone submits their RSA public key
* Their ID is the hash of their public key.

Then, to facilitate random number generation that's agreed on by the group, everyone in the group generates a shared random number:
`groupRandomSeed = hash(everyone's ID, sorted from least to greatest)`
Then each person generates a random number that is dependant on the group random number but is different for each person:
`localRandomSeed = hash(groupRandomSeed + my ID)`


Then in order to prevent duplicates (the same card being in multiple hands), we split up the deck into numPlayers segments.

Here's how you do this:

1. Make a random number generator, seeded by the string `groupRandomSeed + "whiteCards"`
2. Use that random number generator to shuffle the white cards
	* Assume that each player has the same list of white cards in the same order to begin with
3. Split this deck into `numPlayers` equal segments
4. Assign each player a segment, going in order of player ID from least to greatest

Now every player has a list of cards that they could draw the card in their hand from 

**Then each player selects `n` cards from their segment to be their hand.**
First they select random numbers `myRandom[0...n]`. They keep these secret.
Here's how they calculate the cardIndex of card i of their hand: `cards[i]=mySubDeck[hash(localRandomSeed + myRandom[i] + i) % mySubDeckLength]` **this hash needs to use a different algorithm that is designed to be slow, like bcrypt, to prevent brute forcing to get a certain card**

Now that each player knows what cards are in their hand, they need to "encrypt" and declare their hashed hand.
They generate more random numbers `otherRandomNumbers[0...n]`. Then they calculate `hashedCard[i]=hash(cards[i] + otherRandomNumbers[i])`. Their encrypted hand is composed of `hashedCard[0...n]`. Everyone broadcasts their hashed hand, signed by their private key, to everyone.  (not sure about the signed by the private key part, might be unnecessary)


The point of all this is so that each player has a secret set of cards in their hand, **but at any point can prove that a given card is in their hand**.
In order to prove that I really have `cards[i]`, I have to provide `myRandom[i]` and `otherRandomNumbers[i]`. Then others can verify `mySubDeck[hash(localRandom + myRandom[i] + i) % mySubDeckLength]=cards[i]` and that `hashedCard[i]=hash(cards[i] + otherRandomNumbers[i])`. This proves that I randomly selected this card (and didn't specifically pick it), and that it was in the encrypted hand that I originally disseminated. 



# Gameplay

**Here's how it works to actually play the game.** What should happen is that everyone gives a card face down to the judge, the judge flips them over, picks one, then the person who submitted that card says "hey that was me".
	This is made easier by the fact that there are no duplicate cards.
	Here's how we do that in a p2p way:

1. **everyone gives a facedown card to the judge**
Everyone picks a card, and encrypts it with the judge's public key. (add random padding that judge will ignore to the card before encrypting to prevent fingerprinting attacks)
Then they send the encrypted cards to each other. Whenever someone who isn't the judge receives an encrypted card, they forward it to three other random people (which may include the judge).
	Why do this? Because now the judge is receiving encrypted cards from random people, not necesarily the same person as who picked the card. This ensures that the judge doesn't know who submitted what card.
		So now the judge has all the encrypted cards.
* **judge flips them over and picks one**
		Judge decrypts cards with its private key. Judge picks one (accept user input).
		Judge signs the decision with its private key, and sends to everyone.  
* **Judge shows all cards to everyone, along with its decision** Judge broadcasts all the cards and the padding, so that everyone can verify that these were the real cards that were sent around. Displays to all the users the list of submitted cards, which one was theirs, and which one won. Verify judge's signature for who won
* **person who submitted winning cards claims winnings**
		This is easy: this person generates the hash proof discussed earlier and gives it to everyone. 
		Everyone agrees that that person won by verifying the proof.




Also, the order of who is judge when is generated by taking the list of users, and randomizing them according to the key `groupRandomSeed + "judgeOrder"`. 

The order of the black cards (which is public, like the order of the judges) is generated by taking the list of black cards and randomizing them according to the key `groupRandomSeed + "blackCards"`




Now people need to draw another card... (to be continued)


