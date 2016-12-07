Handshake Problem
=======================
Ten people are in a room, and they are given a task, namely, for each person to shake hands with exactly three different people.

How many ways could they complete this task?

Or more generally, how many ways could n people shake hands with k different people (hereafter 'n_k')?

(For the sake of clarity, no handshake can be repeated, and no person can shake hands with herself.
	To illustrate, 4_2 = 3. For people A,B,C,D, here are the 3 combinations:
		# AB AC BD CD
		# AB AD BC CD
		# AC AD BC BD)

The goal is to solve for 10_3.


---

BACKSTORY

This is the combinatorics problem that got me into coding.
While teaching GRE Quant, a student of mine came up with this question.
I became obsessed, and eventually realized I would need a computer to accurately solve it.
So I learned some python, wrote my first program (woo hoo!), and the rest was history.

My original (brute force) approach to 10_3 required testing over 344 billion combinations (45 choose 15), and took over 24 hours to run. Ouch!
Many iterations later, I came up with the dynamic solution below. handshake(10,3) now runs in less than one second.

Below I'll descripe how the main function works.


---


ABSTRACTION: represting people with primes

We start off with an array of n primes (representing people). So when n = 10, we start off like this:
    people = [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]

By representing people with primes, we can "encode" handshakes as the product of two primes.
For example, a possible solution would look like this:
	solution = [6, 10, 14, 15, 21, 35, 143, 209, 221, 253, 323, 377, 437, 493, 667]

The '6' in the solution encodes that 2 and 3 shook hands. And the '10' encodes that 2 and 5 shook hands, etc.
	(The prime factorization of 6 is 2x3 -- that's why we used primes to represent people).

* IMPORTANT: This gives us an easy way to test whether a series of handshakes is a solution.

When each of 10 people shakes hands with 3 people, there will be 15 handshakes total. 
So a solution will contain 15 distinct handshake-encoding integers.
And the product of those 15 integers will be equal to:
	target_product = (2x3x5x7x11x13x17x19x23x29) ^ 3

Or that's at least what the prime factorization of target_product would look like. Each person will be involved in 3 shakes. So each prime will need to appear in the prime factorization of target_product exactly 3 times.
	(Another way to look at this: the 15 shakes will involve 30 total hands. Each of the 10 hands will be involved in a shake exactly 3 times).

So we can take another look at a solution:
	solution = [6, 10, 14, 15, 21, 35, 143, 209, 221, 253, 323, 377, 437, 493, 667]

We we know this is a solution, because each of the numbers encodes a handshake (i.e. is the product of two primes on the original list), and because all the numbers taken together have a product of target_prime.


---

DYNAMIC APPROACH

I'll use 4_2 to demonstrate the main function. For this problem, we would have:
	primes = [2,3,5,7]
	possible handshakes = [6,10,14,15,21,35]
	target_product = (2x3x5x7)^2 = 44100

Note that a brute force approach here would take 15 iterations (6 possible handshakes, choose 4). But things increase dramatically for 10_3 (45 choose 15).

One way to proceed dynamically would be to start with an empty array [], and then iterate through the handshakes, adding handshakes whenever they are elgibile.
On this approach, we would start off like this:
	[]
It would be "legal" to either add or not add 6 to the empty array. So after one iteration, we would have this:
	[]
	[6]
On the next iteration, we could either add or not add 10 to each of the arrays:
	[]
	[6]
	[10]
	[6,10]
And then we could add 14 to just those cases where it would be legal, etc...

In genearal, this approach would be much faster than brute-force, since it would add only "eligible" or "legal" handshakes (and thus not test every single inelibible combination).
And we would eventually construct every possible solution.

But we can do better, since we are only required to count (rather than construct) the solutions.

So instead of starting off with an empty array, we can start off with an object like this:
	master = {44100: 1}

This means there is exactly 1 combination where the combination would require handshakes that multiply to 44100.
   (This combination could be represented with an empty array, but the array doesn't need to appear in our code)

Then we'll iterate through the possible handshakes. 44100 is divisible by 6, so we would add the 44100/6 = 7350 key to master:
	master = {44100: 1,
			   7350: 1}

The second key just means there is 1 combination (represented by [6]) where handshakes with a product of 7350 still need to be added.

After iterating through this list, the final solution would be value of master[1].

Note that this second dynamic approach would save A LOT of time when we increase the number of people. For 10_3, the first dynamic approach is about 60x slower than the second.


---

ONE FINAL SHORTCUT

The main function uses one final "shortcut" to simplify the problem. I'll illustrate again with 10_3.

For 10_3, we started off with a list of people:
	people = [2,3,5,7, ..., 29]

On the "shortcut", we can then calculate only those solutions where first person on the list (2) shakes hands with the next three (3,5,7).
In other words, we can begin by assuming that any solution would begin like this:
	solution = [6,10,14, ... ]
		
		Note: On the dynamic approach, this would entail dividing our initial target_product by (6*10*14).
		And our "handshake" array would also shrink (which means fewer iterations!), since we could remove all the other handshakes that involved person 2.
		Another note: In the main function, I actully do this shortcut with the largest primes (rather than smallest), just so the function can work with smaller numbers.

Then after getting our initial result, our final solution would just be:
		initial result * (9 choose 3)
	i.e.,
		initial result * 84

Why does this work?
	This works because it doesn't matter which three handshakes we assign to the first person.No matter which 3 handshakes we choose, there will be the same nubmer of solutions that could be completed from the inital assingment.
	And there are 84 initial assignmetns that we could give to the first person. (9 other people, choose 3)

