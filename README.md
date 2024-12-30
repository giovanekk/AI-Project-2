# AI-Project-2
This is an extension of AI project 1. Most of the code is different because I worked with a different partner for this project.

REMINDER OF RULES: 


Explain how the space rat knowledge base should be updated, based on whether the bot receives a ping or does not receive a ping
We want to find the probability that the rat is in any open (meaning not blocked as a wall) square on the ship. Let N denote the number of open cells on the ship. The rat has an equal probability of being at any open cell to begin. So set each of these probabilities to
1/N.
The information we can use is the pinging of the rat detector and whether the bot hears the ping or not. So for all open cells (let a cell be denoted C) on the board we want to compute
P(rat at C | ping heard).
By Bayes’ Theorem, this is equivalent to
[P(ping heard | rat at C) * P(rat at C)] / P(ping heard).
The probability of hearing a ping is
e ^ (-α(dBR-1)),
where α is the detector sensitivity and dBR is the manhattan distance between the bot and rat. Now we only need the probability of hearing a ping assuming the rat is at the cell in question. This probability is the same as that of hearing the ping, except the distance is not between the bot and where the rat actually is, but rather assuming it’s at the cell in question. So the probability becomes
e ^ (-α(dBC-1)),
where dBC denotes the manhattan distance between the bot and the cell in question where we have assumed the rat to be. Now substitute:
[P(ping heard | rat at C) * P(rat at C)] / P(ping heard) = [ (e ^ (-α(dBC-1))) * 1/N] / e ^ (-α(dBR-1). This is how you would update the knowledge base the first time from just 1/N in every square to [ (e ^ (-α(dBC-1))) * 1/N] / e ^ (-α(dBR-1). Call this value the “probability_last_time”. The knowledge base can continued to be updated as
[ (e ^ (-α(dBC-1))) * probability_last_time] / e ^ (-α(dBR-1).
Similarly, if the ping is not heard the knowledge base can be updated as
[ (1 - (e ^ (-α(dBC-1)))) * probability_last_time] / (1 - e ^ (-α(dBR-1)),
where the differences come from the probability of not hearing a ping being 1 - P(ping heard) and 1 - P(ping heard | rat at C). This is because you either hear the ping or do not, and these probabilities must sum to 1. So it is appropriate to just take one and subtract the probabilities relating to hearing a ping in order to get those of not hearing a ping.


Question 2
Explain the design and algorithm for the bot that you design, being as specific as possible
as to what your bot is actually doing. How does your bot make use of the information available to make informed decisions about what to do next?
For phase 1, the bot works exactly the same as the baseline bot. It moves around until it finds itself. For the second phase, the first action the bot takes is pinging. Depending on whether it
  
 senses the ping or not, it updates the probabilities accordingly. The pinging value (sensed or not sensed) is then stored for later use. Now instead of finding the highest probability cell, we decided to calculate the cell with the highest probability to distance ratio. That is, we got the knowledge base of the rat locations, and then for a given cell we got the probability that the rat is in it and divided it by the distance from the bot to that cell. So each cell had a probability to distance ratio. We then iterated through the entire ship and got the cell with the highest ratio.
We then used BFS to calculate the shortest path from the bot to the highest ratio cell.
Then, we revisit the pinging value mentioned previously.
If the bot did sense the ping, we treated it as though there was a higher probability that the rat was very near. Therefore, it would only walk one step along the path before pinging again.
If the bot did not sense the ping, we treated it as though there was a higher probability that the rat was further away. Therefore, it would take more than one step along the path before pinging again. In fact, we made the bot walk up to a length of 1/20*ɑ of the path.
This cycle continued until the bot found the rat.


Question 3
Evaluate your bot vs the baseline bot, reporting a thorough comparison of performance. Plot your results as a function of α.
     
 Question 4
Previously, the space rat was assumed to be stationary. Now assume that in every timestep, after the bot takes an action, the space rat moves in a random open direction.
– Work out the space rat knowledge base updates based on this. What changes, what probabilities do you need to calculate? Again, there is a correct answer.
When the pinging action occurs, the knowledge base is updated in the exact same way as in question one. Every time the bot takes one of its three actions, one unit of time passes. Then the rat moves one step. Every time the rat moves, the knowledge base must be updated. This is where updating the knowledge base differs. You will do the following for every cell. Denote one of these cells C as an example. Make sure C is open and not blocked on the ship. If C is open, then look up, down, left, right and make sure these neighbors are open as well. The following is done for each of the four neighbors if they are open. Denote one neighbor D as an example. The number of open neighbors for C is denoted N. The rat is currently in D with probability P(D) and there is a 1/N chance that the rat goes to each of the neighbors in D. C is one of these neighbors, so the probability of the rat being at C increases by 1/N times P(D). Similarly, the probability of the rat being at D decreases by 1/N times P(D). Again, I want to emphasize that this is repeated for all neighbors D for a cell C, and this entire process is repeated for every cell C. See the rat_cardinal_update method for the code.
– Simulate and compare both bots in this new environment. How does the performance change? Again compare across α > 0.
The performance of both bots becomes much worse. Both bots take hundreds of more actions than when the rat was not moving. Neither bot is really better than the other because neither of them are suited to the new environment. There also is much less of a correlation with the number of actions the bot takes and alpha. The variance is suddenly very high. This is probably because the bot gets stuck chasing where the rat was, and it takes a long time for the bot and rat to happen to end up on the same square, regardless of how good the bots are. Note that the different graphs look quite similar, but if you look at the raw data in the code, the numbers are different. It is just hard to tell from the graphs because the number of actions is so high.
   
   – Try to improve on your bot design to be even more effective in this moving space rat situation. What did you need to change, if anything?
One thing that improved our bot and made it catch the rat much faster is trying to move the bot to where it anticipates the rat will be. The original improved bot that we made assumes a stationary rat, so it just updates the knowledge base for the rat as described in question one. In the first part of question four, we explained how we adapted this bot to update the knowledge base for the rat every time the rat moves. The thing is, this means that the bot is just going to the square where it thinks the rat is most likely to be. Because the rat moves and then the bot pings, the bot is left chasing where the rat was. We want the rat to move into the same cell as the bot and then ping so that way the rat is caught. In order to accomplish this, we modified the bot to move towards the cell that is most likely to have the rat one timestep in the future. The bot simulates the rat moving once, looks for the highest probability, but does not update the knowledge base with this information. Rather it just sets its path to where it thinks the rat will be when the rat moves. This allows the rat to move onto the bot in much less time. One other thing that decreased is the variance between trials with different alpha values. This is probably because the bot originally chased where the rat was, and so it was much more random for the bot and rat to be in the same cell. Now you are trying to ensure they are in the same place at the same time, increasing consistency. Again, the graphs look similar, but look at the raw data with the code to confirm it is different.

     
