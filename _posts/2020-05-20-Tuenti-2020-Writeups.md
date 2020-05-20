# [Tuenti Contest](https://contest.tuenti.net/) 10 Write-ups

In this post I will be sharing my answers to the questions I successfully solved.
In addition you can find the codes to the questions from: [Link](https://github.com/sukrubezen/tuenti10)

## Challenge 1 - Rock, Paper, Scissors

Rock beats Scissors, Scissors beats Paper and Paper beats Rock. In the input R is rock, S is scissors and P is paper. We are expected to print the winner of each case and if it is a tie print "-". Fairly simple.

## Challenge 2 - The Lucky One

In this question we are provided multiple test cases. For each test case we are given matches between two players and the result of that match. If result is 1 then first player wins and if 0 second player wins. We are asked to find the strongest player in each test case. It is also given that no match results with a tie.

This problem can be represented as a directed graph. Assume each player in a match as a node and add a directed edge from the winner to the loser. If all matches in a test case are represented like that we would have a directed graph. Since question asks for the strongest player we can assume there is one and only one node that have no incoming edge and that is our result.

## Challenge 3 - Fortunata and Jacinta

In this question we are asked to analyze a book by the writer "Benito Perez Galdos" named "Fortunata y Jacinta". We simply need to rank the words in the book in terms of their frequency in descending order and ties are broken by ascending UNICODE order of the words.

Some preprocessing steps are also given.

* Words will be lowercased
* Only valid chars are: " a, b , c, d, e, f, g, h, i, j, k, l, m, n, ñ, o, p, q, r, s, t, u, v, w, x, y, z, á, é, í, ó, ú, ü" Others will be discarded with space.
* Unicode order of characters are in ascending order: "a, b, c, d, e, f, g, h, i, j, k, l, m, n, o, p, q, r, s, t, u, v, w, x, y, z, á, é, í, ñ, ó, ú, ü"
* Words having less than 3 characters will be discarded.


After preprocessing and fetching each each word we can simply count the occurence of each word and first sort ascendingly based on word and then later descendingly based on the count of that word.

## Challenge 4 - My Favorite Video Game

This question is related with network security. There is a production, preproduction and  development server. Production and preproduction share the same database. Preproduction and development servers are behind a VPN.

Production server URL: "steam-origin.contest.tuenti.net:9876"
Preproduction server URL: "pre.steam-origin.contest.tuenti.net:9876"

At first I thought this was some kind of SQL injection type of question. I spent a lot of time.

But later I found that there was no DNS registration for Preproduction server URL. That means they were using Virtual Hosts inside their webserver so I tried changing the "HOST" key in the header to preproduction while I was querying to the production to fool it as if I was actually querying preproduction and it worked.

```python
import requests

url = "steam-origin.contest.tuenti.net:9876/games/cat_fight/get_key"

payload = {}
headers = {
  'Host': 'pre.steam-origin.contest.tuenti.net'
}

response = requests.request("GET", url, headers=headers, data = payload)

print(response.text.encode('utf8'))
```

## Challenge 5 - Tuentistic Numbers

A Tuentistic number is a number such that when written in text form includes the word "twenty". Examples are 20, 29, 122, 100020

Question asks us to find the maximum number of tuentistic numbers when summed up gives the given number or say it is impossible to construct the given number by summing up any tuentistic numbers.

Since lowest Tuentistic number is 20, I checked how many 20s I should add to get as close as I can get to the provided number. If there is a remainder then I checked whether with the amount of 20s I used is it possible to cover this remainder. Remember 20, 21, 22, 23, 24, 25, 26, 27, 28, 29 are all Tuentistic numbers and I can swap a 20 I used with any of them if necessary. Code is below.

```python
N = input()

for t in xrange(N):
    now = input()
    k, rem = now//20, now%20

    if 0 <= rem <= 9*k:
        print "Case #%d: %d" % (t+1, k)
    else:
        print "Case #%d: IMPOSSIBLE" % (t+1)
```

## Challenge 6 - Knight Labyrinth

We connect to a server and when we do a 5x5 grid shows up, we represent a knight (and move like one in chess L shaped) and our current position is always in the middle cell. We need to save the princess by jumping to her cell.

At first I tried to generate the map of the labyrinth manually by moving on the game and writing the grid values in a paper. Later I realized that labyrinth was larger than what I expected it to be and I spent quite a time.

My second approach was to write a backtracking solution. Whenever I move somewhere I have not visited before and could not find the princess there, I was moving in the reverse direction thanks to recursion. By trying every option, code finally managed to find the princess. Code can be found in github.

## Challenge 7 - Encrypted lines

Question gives us an encrypted text and a audio file. We need to decrypt it somehow.

Recently I played a game called Cypher and there were various cryptography challenges and techniques there. One of them was "Monoalphabetic Substitution" and I wanted to give a chance to it. The idea of Monoalphabetic Substitution is a permutation of the alphabet is taken and every character at the same indexes are replaced in the encrypted text. To find the permutated alphabet one can check the frequency of words of that language and try to guess the characters. And that's what I did. For english, "the" and "and" are good starts so I started mapping them to text and trying other letters.

Later when half of the permutated alphabet was discovered I started to wonder what was the music for? Did I miss something? I tried using audio steganography techniques, analyzing frequencies etc but nothing. Later an idea came up my mind and I soundclouded the music.

It was "Symphony No:9 Op 95, In e Mi" by "Antonin Dvorak". When I saw the name Dvorak suddenly I checked whether the substitions I found so far matched with dvorak mapping and it did!

So I wrote a dvorak to english translator and voila!


## Challenge 8 - Headache

This queston only provided an empty image.

I downloaded the image and tried some steganography techniques but no result. Later I found that there was a brainfuck code embeded in the png file. It is always useful to run "strings FILE" to see if there is any strings that are embedded in the binary. When brainfuck code was executed flag was there.

## Challenge 9 - Just Another Day on Battlestar Galactica

This was another cryptography question. The code that encrypts the text is given and one sample of encrypted and decrypted message is also given and we are asked to decrypt another encrypted message.

The main logic of encryption was by xoring the key in reverse order with the message provided and when key is found by using the samples provided it was rather easy. Code can be found on github.

## Challenge 10 - Villa Feristela escape room

We connect to a remote machine and when we do we realize that bash only accepts emojis. First I tried to map bash commands to the provided emojis and later I started traversing directories.

There are vampires, zombies, robots, princesses, wizards, kings inside directories each one of them requests and provides us some item. After we provide everyone what they want we learn that there is a ladybug missing in the castle.

We go the toilet directory but we can not go into lower directories because of permissions. We only have execute rights not read rights. So I found the emoji related with ladybug and tried to execute it under that directory and it worked, voila!

## Challenge 11 - All the Possibilities

Question asks us how many ways are there to reach to a certain number using any positive number any amount of times. There are also prohibited numbers that can not be used in the summation.

This question is an example of dynamic programming question ["Coin Change"](https://www.geeksforgeeks.org/coin-change-dp-7/). Code can be found on github.

## Challenge 12 - Hackerman Origins

Could not solve

## Challenge 13 - The Great Toilet Paper Fortress

We can build a toilet paper tower such that central tower has a height and an area, and every level outside of the central tower needs to have a height two less than the previous one. This should continue until there are no outer levels that can be put.

Given the number of toilet papers we have now, question asks us to find the maximum height of the tower and maximum amount of used toilets papers to build that heighted tower.

My solution used binary search because ranges were really big. In addition I divided the problem into two parts. First I found the maximum height that can be reached and after that I tried to find the maximum amount of toilet papers that can be used with that height with the upper bound provided in the input.

Code can be found on github.


## Challenge 14 - Public Safety Bureau

Could not solve

## Challenge 15 - The internsheep

This was the hardest question IMHO.

There are some files given (Total size is 13 TB)
There are some queries which asks us to change 1 specific byte to some value and after every change we are asked to calculate the CRC32 of the file. Changes are persistent.

To start with I do not even 13GB of empty space in my HDD so extracting the compressed files  was no option for me.

I learned one could iterate through contents of a compressed file without extracting it byte by byte. So I implemented that.

Calculating CRC32 is a linear process so to calculate CRC32 of a file one needs to start from beginning till the end. Considering the size of files this could take forever.

Later I learned that it was possible to calculate CRC32 of a file if this file was concanatation of two other files and these two other files' CRC32 was known.

After this knowledge, I split the file based on the query index points and I had some amount of smaller files, I calculated their CRC32. For every query I inserted that byte and recalculated the CRC32 by using the CRC32 values I found earlier.

Even using that technique and using a cloud machine with 8 cores and 16gb of RAM it took 3 whole days for the program to complete. Using Python can be a reason as well.

Code can be found on github

## Challenge 16 - The new office

There is a technological company, there are teams like software engineers, designers etc. Every team has some amount of people in them and have access to specific floors in the building. If everyone in the company would go to toilets at the same time and every floor have the same amount of toilets,  what would be the minimum toilet count on each floor?

At first this seemed like a network flow problem but I could not implement that.

Later I used another algorithm. It was a greedy approach. For every floor I checked teams that has access to it and according to the (number of members in the group not visited a toilet yet) / (number of floors this team has access to) metric I found the group that needs to use this floor most and assigned one of their members to that floor and continued.

Code can be found on github.


This was the last question that I could solve unfortunately.


















