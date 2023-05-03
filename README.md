# HeartGame
In the Hearts game, the start player plays a card. This is called the lead card. Each player will take turn
to play another card. Here are the game rules. 

Rule 1: each player must play a card of the same suit as
the lead card, unless the player has no card of the same suit. In that case, he/she is allowed to play any
card. We call such a case a “discard” of a card in a different suit. 

Rule 2: the player playing the highest
card of the same suit as the lead card will “win” all 4 cards played in the round. A card of a different suit
than the lead card always loses, even if it is an A. For example, if the lead card is 2, the card A will
still lose to it. 

Rule 3: the winner of a round will be the start player in the next round.

The scoring rule in Hearts is unconventional. A player winning a card of  will be given one point. A
player “winning” the very special card of Q will be given 13 points. So after a hand is played, a total of
26 points would normally be given for all players. There is one single and very important exception. If a
particular player wins all 13  cards and also the Q, then he/she will be the big winner and all the
other three players will be given 26 points instead (so there is a total of 78 points given out). The player
with the least number of points will be the champion after the game is played the pre-determined
number of times.

In this program, the parent will act like the table for holding the cards, as well as the arbitrator to tell
each child what card(s) have been played. A child will play a card by sending its card via a pipe to the
parent, who will then relay the cards played in the round to the next child via a pipe for its consideration.
The parent actually serves as the overall controller of the whole program, prompting the child processes
for card and informing cards played to them.

Here is a simple arrangement: each child always tries to read from the pipe about request from parent.
Parent starts the game by “writing” tolead to the first child and then “reads” from that child. Child
“reads” from parent and “writes” its card to be played with play. Parent then “writes” played with all
cards played in this round to the next child. When the game is finished, parent will compute and print the
score for each child, since it knows all cards being played and the “winner” for each round. It then
“writes” done to all child processes and concludes the game. Note that the sequence of messages
printed by child processes at the beginning may occur in different orders and you do not need to try to
control their order.

<img width="478" alt="image" src="https://user-images.githubusercontent.com/113018465/235930523-44f78048-40db-4254-b516-6e996b064953.png">
<img width="479" alt="image" src="https://user-images.githubusercontent.com/113018465/235930666-2e7dfadb-9ce0-49a7-92ce-8bee7c7205b0.png">
<img width="476" alt="image" src="https://user-images.githubusercontent.com/113018465/235930831-21d5225f-bc01-4f92-9d5c-8b38f9447f94.png">
<img width="478" alt="image" src="https://user-images.githubusercontent.com/113018465/235930924-8fd5caf4-2b01-4c72-9af6-05132ca1cf23.png">
<img width="476" alt="image" src="https://user-images.githubusercontent.com/113018465/235931034-873671ce-5d21-442b-a508-984230872ae1.png">
