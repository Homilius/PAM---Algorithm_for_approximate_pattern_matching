The code is not completely done and has not been properly tested yet.

I have uploaded this project for educational purpuses as i think it is a quite intuitive way of solving (and understanding) the approximate pattern matching problem. I have not heard of or seen this algorithm before so i guess i can name it PAM (Pigeonhole approximative matching).  

The algorithm is build on following principles:

Allowing d mismatches we use the pigeonhole principle and devide our sustring into d+1 pieces. One of these pices can be found (if a match exists) using exact pattern matching (which is a much easier problem to solve).
These pieces are then search using binary search in a suffix array constructed from the reference string (which in this implementation is created by prefix doubling). 
When we get an exact match of one of these pieces we can go from there back to the position that corresponds to the beginning of our pattern (note this position is with respect to the substring not the reference string!).
So above steps will give us all candidate positions that could be interesting to investigate further. 
Now we do a simple local alignment. From the corresponding local alignment matrix we select the smallest value within 2*d entries away from last entry; last row and col). We do 2*d because one d goes to actual edits + one d since we need to generate a flexible interval to find all possible matches within d edits (since the start position is with respect to the pattern and not the reference string and we need to take possible indels into account).

Some of these steps have a worst case running time that is pretty slow. BUT the main idea in this algorithm is that it is very unlikely that we will have to do any extra work. If we devide a pattern of size 100 into four pieces of size 25 and we get a match it is very likely that this match actually represents a reel position of our pattern since a match of 25 is relativly unlikely to habben many times just by chance (in a genome at least). So while the worst case running time is not ideal er are kind of protected it as long as our pattern has a certain length. Also note that this algorithm has very littlen overheads. 
Im convinced that this algorithm has a potential to run pretty fast with the rigth implementation.
Feel free correcting or optimizing parts and shoot me an email; antonhomilius@hotmail.com.






